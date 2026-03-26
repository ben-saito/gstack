---
name: freeze
version: 0.1.0
description: |
  ファイル編集を特定ディレクトリに制限。デバッグ中のスコープ外変更を防止。「フリーズ」「編集制限」と聞かれた時に使用。
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
hooks:
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。


# /freeze — 編集をディレクトリに制限する

ファイル編集を特定のディレクトリにロックします。許可されたパス外のファイルに対するEditまたはWrite操作は（警告ではなく）**ブロック**されます。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"freeze","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## セットアップ

ユーザーに編集を制限するディレクトリを確認します。AskUserQuestionを使用してください：

- 質問：「編集を制限するディレクトリはどこですか？このパス外のファイルは編集がブロックされます。」
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

ユーザーに伝えてください：「編集が `<path>/` に制限されました。このディレクトリ外へのEditまたはWriteはブロックされます。境界を変更するには `/freeze` を再実行してください。解除するには `/unfreeze` を実行するかセッションを終了してください。」

## 仕組み

フックはEdit/Writeツール入力JSONから `file_path` を読み取り、パスがfreezeディレクトリで始まるかどうかを確認します。そうでない場合、操作をブロックするために `permissionDecision: "deny"` を返します。

freeze境界はステートファイルを通じてセッション中持続します。フックスクリプトはEdit/Writeの呼び出しごとにそれを読み取ります。

## 注意事項

- freezeディレクトリの末尾 `/` により、`/src` が `/src-old` にマッチすることを防ぎます
- FreezeはEditおよびWriteツールにのみ適用されます — Read、Bash、Glob、Grepには影響しません
- これは誤編集の防止であり、セキュリティ境界ではありません — `sed` などのBashコマンドは境界外のファイルを変更できます
- 無効にするには `/unfreeze` を実行するか会話を終了してください
