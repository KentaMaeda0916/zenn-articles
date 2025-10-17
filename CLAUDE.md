# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

このリポジトリはZennの記事・本を管理するためのものです。Zennはエンジニアが技術知識やアイデアを共有するための日本のプラットフォームです。すべての記事は日本語で書かれています。

## ディレクトリ構造

- `articles/` - 記事のマークダウンファイル
- `books/` - 本のプロジェクト（現在は空）
- `images/` - 画像アセット（現在は空）

## 記事管理

### 記事のフォーマット

記事は`articles/`ディレクトリ内のマークダウンファイルで、Front Matterメタデータを含みます：

```yaml
---
title: "記事のタイトル"
emoji: "🔨"
type: "tech" | "idea"
topics:
  - "トピック1"
  - "トピック2"
published: true | false
published_at: "YYYY-MM-DD HH:MM"
---
```

**重要な要件:**
- ファイル名は12〜50文字（半角英数字（a-z0-9）、ハイフン（-）、アンダースコア（_））
- `type`は"tech"（技術記事）または"idea"（アイデア記事）
- `topics`は関連する技術タグ
- `published: true`で記事がZenn上で公開される

### コンテンツのトピック

このリポジトリは以下に焦点を当てています：
- AI支援開発（Cursor、ChatGPT、Perplexity）
- 個人開発の実践方法
- iOS/Swift開発ツール（Xcode、Alex Sidebar）
- 開発生産性とワークフロー

## よく使うコマンド

### 新しい記事を作成

```bash
npx zenn new:article
```

オプション付き：
```bash
npx zenn new:article --slug article-slug --title "記事タイトル" --type tech
```

### 新しい本を作成

```bash
npx zenn new:book
```

### ローカルでプレビュー

```bash
npx zenn preview
```

公開前に記事や本をプレビューするためのローカルサーバーが起動します。

## デプロイ

コンテンツは連携されたGitHubリポジトリにプッシュすると、自動的にzenn.devにデプロイされます。手動でのビルドやデプロイコマンドは不要です。

## 執筆ガイドライン

- すべてのコンテンツは**日本語**で書く
- 記事は実践的で実体験に基づいたものにする
- ツールやワークフローを説明する際は具体例とスクリーンショットを使用する
- Zennのマークダウン形式の規約に従う
- その他詳細なルールは以下のドキュメントを参照する
https://zenn.dev/zenn/articles/zenn-cli-guide
