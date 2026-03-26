---
name: unfreeze
version: 0.1.0
description: |
  /freezeの制限を解除。すべてのディレクトリへの編集を再度許可。
allowed-tools:
  - Bash
  - Read
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。


# /unfreeze — Freeze境界の解除

`/freeze` で設定された編集制限を解除し、すべてのディレクトリへの編集を許可します。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"unfreeze","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## 境界の解除

```bash
STATE_DIR="${CLAUDE_PLUGIN_DATA:-$HOME/.gstack}"
if [ -f "$STATE_DIR/freeze-dir.txt" ]; then
  PREV=$(cat "$STATE_DIR/freeze-dir.txt")
  rm -f "$STATE_DIR/freeze-dir.txt"
  echo "Freeze boundary cleared (was: $PREV). Edits are now allowed everywhere."
else
  echo "No freeze boundary was set."
fi
```

ユーザーに結果を伝えてください。なお、`/freeze` フックはセッション中引き続き登録されていますが、ステートファイルが存在しないためすべてを許可します。再度制限するには `/freeze` を再実行してください。
