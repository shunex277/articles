---
title: "Cloudflare OneのTunnelsを使って，サーバーのポートを公開せずSSHする"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['cloudflare']
published: true
published_at: 2025-07-21 09:28
---

基本的には[公式ドキュメント](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/use-cases/ssh/ssh-infrastructure-access/) の通りにやるだけですが，ステップが多く分かりづらいところも多いのでUIの画像付きで，備忘録的に書きます（ただしUIは2025/05/05時点のものです．)

# 1. Cloudflare Tunnelの作成

https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/use-cases/ssh/ssh-infrastructure-access/#1-connect-the-server-to-cloudflare

## ダッシュボード設定ガイドに従ってサーバー用のCloudflare Tunnelを作成します。

https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1746414089-oynOK0MW8cm47e6h3DZUvuGs.png)

2025/05/05 時点ではWARP ConnectorがBetaなので，とりあえずCloudflaredにします

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1746414159-Kd5LImyNtFQMkC091UR3VoHq.png)

「Tunnel name」は適当にいれて，「Save tunnel」します

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1746414988-SNPdjypbOWJro8GBqzXUkLcu.png)

リモートサーバーでdockerのコマンドを走らせます．
おそらくホストのネットワークに接続できないといけないので `--net=host` が必要です

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1746415055-FRiV9Dq2fSIpr4uvmBhAX8jC.png)

アプリケーション接続ステップをスキップし、ネットワーク接続に進みます。

トンネルの「Private Networks」タブで、サーバーのIPまたはCIDRアドレスを入力します（通常はプライベートIPですが、パブリックIPも可能です）。

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1746416116-69h2fuWMQLaXpke3D7iRY0xv.png)

# 2. PCをCloudflareに接続する

https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/use-cases/ssh/ssh-infrastructure-access/#2-set-up-the-client

## a. デバイスに[WARPクライアントをデプロイ](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/)します（Gateway with WARPモード）。

Manual Deploymentにします．デプロイといっても手元のPCにダウンロードして，設定するだけです

https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/manual-deployment/

「Windows, macOS, and Linux」の「Enroll using the GUI」のステップ3はmacの場合は画面上部のメニューバーのことを指してます．（ただし，この作業は本記事の下に続く，「b. TCPのGatewayプロキシを有効化します。」と「c. ゼロトラスト組織に登録できるデバイスを決定するデバイス登録ルールを作成します。」を完了したあとに行う必要があります）

上記リンク先のドキュメントのステップ7の team nameは settings > Custom Pagesから見れます

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1746416608-iWIpEP7y1XBCNTzbcgHKQ24r.png)


## b. TCPのGatewayプロキシを有効化します

https://developers.cloudflare.com/cloudflare-one/policies/gateway/proxy/#turn-on-the-gateway-proxy

これは以下のページ

https://developers.cloudflare.com/cloudflare-one/policies/gateway/proxy/

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1752914258-iW3IMvbcXjZq0mGEJdLCptl8.png)
![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1752914291-xDYwSWT62U8oNuFytKGM9EX5.png)

## c. ゼロトラスト組織に登録できるデバイスを決定する[デバイス登録ルール](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/device-enrollment/)を作成します。

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1752914561-eAKogyR8YaMtp9THhvlxZLC7.png)

Device enrollment permissionsの「Manage」をクリック

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1752914527-Rt1CVLxYuU8XpoAKqs4JTFwg.png)

Policyがすでにある場合は「Select existing policies」から選択，ない場合は「Create Policy」か「Add」のようなボタンがあると思うのでクリックする．するとポリシー作成のページに移動するので[デバイス登録ルールページ](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/device-enrollment/)の例を参考にポリシーを作成し，Device enrollment permissionsのページに戻り，「Select existing policies」から作成したポリシーを選択します

ここまで来たら「a. デバイスにWARPクライアントをデプロイします（Gateway with WARPモード）。」の[「Enroll using the GUI」](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/manual-deployment/#enroll-using-the-gui)を再開します．

# 3. Split Tunnelsの設定

https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/use-cases/ssh/ssh-infrastructure-access/#3-route-server-ips-through-warp

デフォルトで、WARPはRFC 1918領域（プライベートネットワークで使用される172.16.0.0/12、192.168.0.0/16、10.0.0.0/8など）宛てのトラフィックを除外します．これにより、WARPクライアントはプライベートIPアドレスのSSHサーバーにトラフィックを送信しません。

なので，WARP使用時にプライベートIPでSSHすることができるようにするためにSplit Tunnelsを設定する必要があります

まず，Settings > WARP Clientに行きます

Device settingsでプロファイルを作成するか，Defaultプロファイルの設定を変更します．（右端の「⋮」をクリックして「Configure」を選択）

「Split Tunnels」の「Manage」ボタンをクリックして，リモートマシンのプライベートIPやリモートマシンのネットワーク範囲が除外されるようにします

# 4. ターゲットの作成

https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/use-cases/ssh/ssh-infrastructure-access/#4-add-a-target

1. https://one.dash.cloudflare.com/ で、「Networks」>「Targets」に移動します。
2. 「Add a target」を選択します。
3. 「Target hostname」に、ターゲットリソースのわかりやすい名前を入力します（例：production-server）。
4. 「IP addresses」に、リモートサーバーのIPv4またはIPv6アドレスを入力します。
5. ドロップダウンメニューから、リソースが存在するIPアドレスと仮想ネットワークを選択します。
6. 「Add target」を選択します。

# 5. Accessアプリケーションの作成

https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/use-cases/ssh/ssh-infrastructure-access/#5-add-an-infrastructure-application

1. https://one.dash.cloudflare.com/ で、「Access」>「Applications」に移動します。
2. 「Add an application」を選択します。
3. 「Infrastructure」を選択します。
4. アプリケーションの名前を入力します。
5. 「Target criteria」で、アプリケーションを表すターゲットホスト名を選択します。
6. サーバーへの接続に使用されるプロトコルとポートを入力します。
7. 「Next」を選択します。
8. ターゲットへのアクセスを制御するポリシーを設定します：
   - ポリシー名を入力します。
   - ターゲットにアクセスできるユーザーを定義するルールを作成します。
   - 「Connection context」で以下の設定を行います：
     - 「SSH user」：ユーザーがログインできるUNIXユーザー名を入力します（例：root、ec2-user）。
     - 「Allow users to log in as their email alias」：（オプション）選択すると、ポリシーに一致するユーザーは、小文字の電子メールアドレスのプレフィックスを使用してターゲットにアクセスできます。
9. 「Add application」を選択します。

# 6. SSHサーバーの設定

https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/use-cases/ssh/ssh-infrastructure-access/#7-configure-ssh-server

Cloudflare SSH CAを信頼するようにSSHサーバーを設定することで、Accessは従来のSSH鍵の代わりに短期間有効な証明書を使用して認証できるようになります。

## a. Cloudflare SSH CAを生成して公開鍵を取得する

まず，次の設定でAPIトークンを作成します

https://developers.cloudflare.com/fundamentals/api/get-started/create-token/

一旦Zero Trustのページから抜けて，https://dash.cloudflare.com/ に右上の人形のアイコンをクリックして「Profile」を選択し，https://dash.cloudflare.com/profile/api-tokens を開きます

![](/images/5a27c385327239-ssh-with-cloudflare_one-tunnels/n0eae51cfa4da_1752983811-tvdIbBJkWYFTK3rZ47fcGE9s.png)

そして，Cloudflare APIに対してリクエストを実行し、SSH CA公開鍵を取得します。

```bash
curl "https://api.cloudflare.com/client/v4/accounts/$ACCOUNT_ID/access/gateway_ca" \
  --request POST \
  -H "Authorization: Bearer $CLOUDFLARE_API_KEY"
```

返されたpublic_key値をコピーします。

## b. リモートのターゲットマシンに公開鍵を保存

1. SSHの設定ディレクトリに移動します：`cd /etc/ssh`
2. 公開鍵を保存するファイルを作成します： `vim ca.pub`
3. ca.pubファイルに、先程の公開鍵を貼り付けます： `ecdsa-sha2-nistp256 <redacted> open-ssh-ca@cloudflareaccess.org`
4. ファイルを保存します。

## c. sshd_configファイルの修正

1. `/etc/ssh`ディレクトリ内で、sshd_configファイルを開きます： `vim /etc/ssh/sshd_config`
2. PubkeyAuthenticationという行を探し、コメントを解除します：

   `# PubkeyAuthentication yes`
   ⇓
   `PubkeyAuthentication yes`

3. PubkeyAuthenticationの下に新しい行を追加します：

   `TrustedUserCAKeys /etc/ssh/ca.pub`

4. ファイルを保存して、エディタを終了します。

## d. SSHサービスの再起動

SSHDの設定を変更したら、リモートマシンでSSHサービスを再起動します：

- 古いDebian/Ubuntuの場合： `sudo service ssh restart`
- 新しいDebian/Ubuntuの場合： `sudo systemctl restart ssh`
- CentOS/RHEL 6以前の場合： `sudo service sshd restart`
- CentOS/RHEL 7以降の場合： `sudo systemctl restart sshd`

# 7. リモートマシンに接続

https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/use-cases/ssh/ssh-infrastructure-access/#8-connect-as-a-user

これで，手元のデバイスでWARPクライアントにログインしてる状態でSSHできるようになります
