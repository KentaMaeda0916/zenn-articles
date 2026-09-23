# CLAUDE.md

Zenn の記事を管理するリポジトリ。GitHub 連携済みで、`main` に push するとそのまま zenn.dev に反映される（ビルド・デプロイ作業は無い）。

## コマンド

```bash
npx zenn new:article --slug <slug> --title "タイトル" --type tech   # 記事の雛形を作成
npx zenn preview                                                    # ローカルプレビュー
```

## 記事ファイル

`articles/<slug>.md`。Front Matter:

```yaml
---
title: "記事のタイトル"
emoji: "🔨"
type: "tech"        # tech: 技術記事 / idea: アイデア記事
topics: ["swift", "ios"]
published: false
---
```

- `published: true` のまま `main` に push すると即公開される。下書きは `false` で作り、`true` への切り替えはユーザーの指示があったときだけ行う。
- `published_at` は通常書かない。未来日時を入れると予約投稿になる。
- ファイル名（slug）が記事 URL になる。公開済み記事のファイル名は変えない（URL が変わりリンクが切れる）。新規は内容を表す 12〜50 文字の英小文字・数字・`-`・`_`。16 進ランダム名の既存記事は Web エディタ由来なのでそのままでよい。
- `topics` は最大 5 個。
- 画像は Zenn のアップローダー（`https://storage.googleapis.com/zenn-user-upload/...`）の URL を貼る運用で、リポジトリに `images/` は無い。画像が必要な箇所はプレースホルダーを置き、ユーザーに差し替えを頼む。
- 補足ブロックは Zenn 記法の `:::message` / `:::details タイトル` を使う。記法の詳細: https://zenn.dev/zenn/articles/markdown-guide 、CLI の詳細: https://zenn.dev/zenn/articles/zenn-cli-guide

## 執筆方針

- 本文は日本語。実体験ベースで、手順やツールの説明には具体例（コマンド・設定・スクリーンショット）を添える。
