---
title: "Claude Code起動時にserenaMCPのブラウザが自動で開くのを無効化する方法"
emoji: "✔️"
type: "tech"
topics:
  - "ClaudeCode"
  - "serena"
  - "ai"
  - "mcp"
published: false
---

# 問題

Claude Codeを起動するたびに、[SerenaMCP](https://github.com/oraios/serena)のWebダッシュボードがブラウザで自動的に開いてしまい、煩わしさを感じることがあります。

# 原因

SerenaMCPは設定ファイル内で`web_dashboard_open_on_launch`というオプションが`true`に設定されている場合、起動時に自動的にブラウザを開く仕様になっています。

# 解決方法

## 1. 設定ファイルの場所を確認

SerenaMCPの設定ファイルは以下の場所にあります：

```bash
~/.serena/serena_config.yml
```

## 2. 設定ファイルを編集

設定ファイルを開き、`web_dashboard_open_on_launch`の値を変更します：

```yaml
# 変更前
web_dashboard_open_on_launch: true

# 変更後
web_dashboard_open_on_launch: false
```
