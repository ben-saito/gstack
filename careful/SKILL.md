---
name: careful
version: 0.1.0
description: |
  破壊的コマンドの安全ガードレール。rm -rf、DROP TABLE、force-pushなどの前に警告。ユーザーは各警告をオーバーライド可能。
allowed-tools:
  - Bash
  - Read
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-careful.sh"
          statusMessage: "Checking for destructive commands..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。


# /careful — 破壊的コマンドのガードレール

セーフティモードが**有効**になりました。すべてのbashコマンドが実行前に破壊的パターンのチェックを受けます。破壊的コマンドが検出された場合、警告が表示され、続行またはキャンセルを選択できます。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"careful","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## 保護対象

| パターン | 例 | リスク |
|---------|---------|------|
| `rm -rf` / `rm -r` / `rm --recursive` | `rm -rf /var/data` | 再帰的削除 |
| `DROP TABLE` / `DROP DATABASE` | `DROP TABLE users;` | データ消失 |
| `TRUNCATE` | `TRUNCATE orders;` | データ消失 |
| `git push --force` / `-f` | `git push -f origin main` | 履歴の書き換え |
| `git reset --hard` | `git reset --hard HEAD~3` | 未コミット作業の消失 |
| `git checkout .` / `git restore .` | `git checkout .` | 未コミット作業の消失 |
| `kubectl delete` | `kubectl delete pod` | 本番環境への影響 |
| `docker rm -f` / `docker system prune` | `docker system prune -a` | コンテナ/イメージの消失 |

## 安全な例外

以下のパターンは警告なしで許可されます：
- `rm -rf node_modules` / `.next` / `dist` / `__pycache__` / `.cache` / `build` / `.turbo` / `coverage`

## 仕組み

フックはツール入力JSONからコマンドを読み取り、上記のパターンと照合し、一致が見つかった場合は警告メッセージとともに `permissionDecision: "ask"` を返します。警告は常にオーバーライドして続行できます。

無効にするには、会話を終了するか新しい会話を開始してください。フックはセッションスコープです。
