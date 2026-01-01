# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

このリポジトリは、Zenn（日本のテックブログプラットフォーム）の記事と本を管理するための専用リポジトリです。

## セットアップ

```bash
npm ci
```

Node.jsバージョン: 24.12.0 (.tool-versionsで管理)

## 主要コマンド

### 新規作成
```bash
# 新しい記事を作成
npx zenn new:article

# 新しい本を作成
npx zenn new:book
```

### プレビュー
```bash
# ローカルでプレビューサーバーを起動
npx zenn preview
```

## ディレクトリ構造

- `articles/` - Zenn記事のMarkdownファイル（`.md`）を格納
- `books/` - Zenn本のMarkdownファイルを格納

## コンテンツ作成時の注意点

- 記事と本は`zenn-cli`を使用して管理される
- コンテンツファイルはMarkdown形式で、Zennの特定のフロントマター形式に従う必要がある
- `npx zenn new:article`や`npx zenn new:book`コマンドを使用することで、正しいフォーマットのファイルが自動生成される