---
name: guard
version: 0.1.0
description: |
  フル安全モード：/careful + /freezeを統合。本番環境での最大安全性。
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../careful/bin/check-careful.sh"
          statusMessage: "Checking for destructive commands..."
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。


# /guard — フルセーフティモード

破壊的コマンドの警告とディレクトリスコープの編集制限の両方を有効にします。
`/careful` + `/freeze` を1つのコマンドにまとめたものです。

**依存関係に関する注意：** このスキルは、隣接する `/careful` および `/freeze` スキルディレクトリのフックスクリプトを参照しています。両方がインストールされている必要があります（gstackのセットアップスクリプトで一緒にインストールされます）。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"guard","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## セットアップ

ユーザーに編集を制限するディレクトリを確認します。AskUserQuestionを使用してください：

- 質問：「ガードモード：編集を制限するディレクトリはどこですか？破壊的コマンドの警告は常にオンです。選択したパス外のファイルは編集がブロックされます。」
- テキスト入力（選択肢ではない）— ユーザーがパスを入力します。

ユーザーがディレクトリパスを提供したら：

1. 絶対パスに解決します：
```bash
FREEZE_DIR=$(cd "<user-provided-path>" 2>/dev/null && pwd)
echo "$FREEZE_DIR"
```

2. 末尾スラッシュを確保し、freezeステートファイルに保存します：
```bash
FREEZE_DIR="${FREEZE_DIR%/}/"
STATE_DIR="${CLAUDE_PLUGIN_DATA:-$HOME/.gstack}"
mkdir -p "$STATE_DIR"
echo "$FREEZE_DIR" > "$STATE_DIR/freeze-dir.txt"
echo "Freeze boundary set: $FREEZE_DIR"
```

ユーザーに伝えてください：
- 「**ガードモードが有効になりました。** 2つの保護が動作しています：」
- 「1. **破壊的コマンドの警告** — rm -rf、DROP TABLE、force-pushなどは実行前に警告されます（オーバーライド可能）」
- 「2. **編集境界** — ファイル編集が `<path>/` に制限されます。このディレクトリ外の編集はブロックされます。」
- 「編集境界を解除するには `/unfreeze` を実行してください。すべてを無効にするにはセッションを終了してください。」

## 保護対象

破壊的コマンドパターンと安全な例外の完全なリストは `/careful` を参照してください。
編集境界の適用の仕組みは `/freeze` を参照してください。
