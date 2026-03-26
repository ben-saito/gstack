---
name: gstack-upgrade
version: 1.1.0
description: |
  gstackを最新版にアップグレード。グローバル vs ベンダーインストールを検出、アップグレード実行、変更点を表示。
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。


# /gstack-upgrade

gstackを最新バージョンにアップグレードし、新機能を表示します。

## インラインアップグレードフロー

このセクションは、すべてのスキルプリアンブルが `UPGRADE_AVAILABLE` を検出した際に参照されます。

### ステップ1：ユーザーに確認する（または自動アップグレード）

まず、自動アップグレードが有効かどうかを確認します：
```bash
_AUTO=""
[ "${GSTACK_AUTO_UPGRADE:-}" = "1" ] && _AUTO="true"
[ -z "$_AUTO" ] && _AUTO=$(~/.claude/skills/gstack/bin/gstack-config get auto_upgrade 2>/dev/null || true)
echo "AUTO_UPGRADE=$_AUTO"
```

**`AUTO_UPGRADE=true` または `AUTO_UPGRADE=1` の場合：** AskUserQuestionをスキップします。「gstack v{old} → v{new} を自動アップグレード中...」とログに記録し、ステップ2に直接進みます。自動アップグレード中に `./setup` が失敗した場合、バックアップ（`.bak` ディレクトリ）から復元し、ユーザーに警告します：「自動アップグレードに失敗しました — 以前のバージョンを復元しました。`/gstack-upgrade` を手動で実行して再試行してください。」

**それ以外の場合**、AskUserQuestionを使用します：
- 質問：「gstack **v{new}** が利用可能です（現在 v{old}）。今すぐアップグレードしますか？」
- 選択肢：["はい、今すぐアップグレード", "常に最新に保つ", "今はしない", "二度と聞かないで"]

**「はい、今すぐアップグレード」の場合：** ステップ2に進みます。

**「常に最新に保つ」の場合：**
```bash
~/.claude/skills/gstack/bin/gstack-config set auto_upgrade true
```
ユーザーに伝えます：「自動アップグレードが有効になりました。今後のアップデートは自動的にインストールされます。」その後ステップ2に進みます。

**「今はしない」の場合：** エスカレーティングバックオフでスヌーズ状態を書き込みます（初回スヌーズ = 24時間、2回目 = 48時間、3回目以降 = 1週間）。その後、現在のスキルを続行します。アップグレードについて再び言及しないでください。
```bash
_SNOOZE_FILE=~/.gstack/update-snoozed
_REMOTE_VER="{new}"
_CUR_LEVEL=0
if [ -f "$_SNOOZE_FILE" ]; then
  _SNOOZED_VER=$(awk '{print $1}' "$_SNOOZE_FILE")
  if [ "$_SNOOZED_VER" = "$_REMOTE_VER" ]; then
    _CUR_LEVEL=$(awk '{print $2}' "$_SNOOZE_FILE")
    case "$_CUR_LEVEL" in *[!0-9]*) _CUR_LEVEL=0 ;; esac
  fi
fi
_NEW_LEVEL=$((_CUR_LEVEL + 1))
[ "$_NEW_LEVEL" -gt 3 ] && _NEW_LEVEL=3
echo "$_REMOTE_VER $_NEW_LEVEL $(date +%s)" > "$_SNOOZE_FILE"
```
注意：`{new}` は `UPGRADE_AVAILABLE` 出力からのリモートバージョンです — アップデートチェック結果から代入してください。

ユーザーにスヌーズ期間を伝えます：「次のリマインダーは24時間後」（またはレベルに応じて48時間後または1週間後）。ヒント：「自動アップグレードには `~/.gstack/config.yaml` で `auto_upgrade: true` を設定してください。」

**「二度と聞かないで」の場合：**
```bash
~/.claude/skills/gstack/bin/gstack-config set update_check false
```
ユーザーに伝えます：「アップデートチェックが無効になりました。再度有効にするには `~/.claude/skills/gstack/bin/gstack-config set update_check true` を実行してください。」
現在のスキルを続行します。

### ステップ2：インストールタイプの検出

```bash
if [ -d "$HOME/.claude/skills/gstack/.git" ]; then
  INSTALL_TYPE="global-git"
  INSTALL_DIR="$HOME/.claude/skills/gstack"
elif [ -d "$HOME/.gstack/repos/gstack/.git" ]; then
  INSTALL_TYPE="global-git"
  INSTALL_DIR="$HOME/.gstack/repos/gstack"
elif [ -d ".claude/skills/gstack/.git" ]; then
  INSTALL_TYPE="local-git"
  INSTALL_DIR=".claude/skills/gstack"
elif [ -d ".agents/skills/gstack/.git" ]; then
  INSTALL_TYPE="local-git"
  INSTALL_DIR=".agents/skills/gstack"
elif [ -d ".claude/skills/gstack" ]; then
  INSTALL_TYPE="vendored"
  INSTALL_DIR=".claude/skills/gstack"
elif [ -d "$HOME/.claude/skills/gstack" ]; then
  INSTALL_TYPE="vendored-global"
  INSTALL_DIR="$HOME/.claude/skills/gstack"
else
  echo "ERROR: gstack not found"
  exit 1
fi
echo "Install type: $INSTALL_TYPE at $INSTALL_DIR"
```

上記で出力されたインストールタイプとディレクトリパスは、以降のすべてのステップで使用されます。

### ステップ3：旧バージョンの保存

ステップ2の出力のインストールディレクトリを使用します：

```bash
OLD_VERSION=$(cat "$INSTALL_DIR/VERSION" 2>/dev/null || echo "unknown")
```

### ステップ4：アップグレード

ステップ2で検出されたインストールタイプとディレクトリを使用します：

**gitインストールの場合**（global-git、local-git）：
```bash
cd "$INSTALL_DIR"
STASH_OUTPUT=$(git stash 2>&1)
git fetch origin
git reset --hard origin/main
./setup
```
`$STASH_OUTPUT` に "Saved working directory" が含まれる場合、ユーザーに警告します：「注意：ローカルの変更がstashされました。復元するにはスキルディレクトリで `git stash pop` を実行してください。」

**vendoredインストールの場合**（vendored、vendored-global）：
```bash
PARENT=$(dirname "$INSTALL_DIR")
TMP_DIR=$(mktemp -d)
git clone --depth 1 https://github.com/garrytan/gstack.git "$TMP_DIR/gstack"
mv "$INSTALL_DIR" "$INSTALL_DIR.bak"
mv "$TMP_DIR/gstack" "$INSTALL_DIR"
cd "$INSTALL_DIR" && ./setup
rm -rf "$INSTALL_DIR.bak" "$TMP_DIR"
```

### ステップ4.5：ローカルvendoredコピーの同期

ステップ2のインストールディレクトリを使用します。更新が必要なローカルvendoredコピーがあるかどうかを確認します：

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
LOCAL_GSTACK=""
if [ -n "$_ROOT" ] && [ -d "$_ROOT/.claude/skills/gstack" ]; then
  _RESOLVED_LOCAL=$(cd "$_ROOT/.claude/skills/gstack" && pwd -P)
  _RESOLVED_PRIMARY=$(cd "$INSTALL_DIR" && pwd -P)
  if [ "$_RESOLVED_LOCAL" != "$_RESOLVED_PRIMARY" ]; then
    LOCAL_GSTACK="$_ROOT/.claude/skills/gstack"
  fi
fi
echo "LOCAL_GSTACK=$LOCAL_GSTACK"
```

`LOCAL_GSTACK` が空でない場合、新しくアップグレードされたプライマリインストールからコピーして更新します（READMEのvendoredインストールと同じ方法）：
```bash
mv "$LOCAL_GSTACK" "$LOCAL_GSTACK.bak"
cp -Rf "$INSTALL_DIR" "$LOCAL_GSTACK"
rm -rf "$LOCAL_GSTACK/.git"
cd "$LOCAL_GSTACK" && ./setup
rm -rf "$LOCAL_GSTACK.bak"
```
ユーザーに伝えます：「`$LOCAL_GSTACK` のvendoredコピーも更新しました — 準備ができたら `.claude/skills/gstack/` をコミットしてください。」

`./setup` が失敗した場合、バックアップから復元してユーザーに警告します：
```bash
rm -rf "$LOCAL_GSTACK"
mv "$LOCAL_GSTACK.bak" "$LOCAL_GSTACK"
```
ユーザーに伝えます：「同期に失敗しました — `$LOCAL_GSTACK` の以前のバージョンを復元しました。`/gstack-upgrade` を手動で実行して再試行してください。」

### ステップ5：マーカーの書き込み + キャッシュのクリア

```bash
mkdir -p ~/.gstack
echo "$OLD_VERSION" > ~/.gstack/just-upgraded-from
rm -f ~/.gstack/last-update-check
rm -f ~/.gstack/update-snoozed
```

### ステップ6：新機能の表示

`$INSTALL_DIR/CHANGELOG.md` を読み取ります。旧バージョンと新バージョンの間のすべてのバージョンエントリを見つけます。テーマごとにグループ化した5〜7個の箇条書きで要約します。情報過多にならないように — ユーザー向けの変更に焦点を当ててください。重要でない限り内部リファクタリングはスキップします。

フォーマット：
```
gstack v{new} — v{old}からアップグレードしました！

新機能：
- [箇条書き1]
- [箇条書き2]
- ...

Happy shipping!
```

### ステップ7：続行

新機能の表示後、ユーザーが元々呼び出したスキルを続行します。アップグレードは完了しました — それ以上のアクションは不要です。

---

## スタンドアロン使用

`/gstack-upgrade` として直接呼び出された場合（プリアンブルからではなく）：

1. 強制的に新しいアップデートチェックを実行します（キャッシュをバイパス）：
```bash
~/.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || \
.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || true
```
出力を使用してアップグレードが利用可能かどうかを判断します。

2. `UPGRADE_AVAILABLE <old> <new>` の場合：上記のステップ2〜6に従います。

3. 出力がない場合（プライマリは最新）：古いローカルvendoredコピーがないか確認します。

上記のステップ2のbashブロックを実行してプライマリのインストールタイプとディレクトリ（`INSTALL_TYPE` と `INSTALL_DIR`）を検出します。次に上記のステップ4.5の検出bashブロックを実行してローカルvendoredコピー（`LOCAL_GSTACK`）を確認します。

**`LOCAL_GSTACK` が空の場合**（ローカルvendoredコピーなし）：ユーザーに「最新バージョン（v{version}）をお使いです。」と伝えます。

**`LOCAL_GSTACK` が空でない場合**、バージョンを比較します：
```bash
PRIMARY_VER=$(cat "$INSTALL_DIR/VERSION" 2>/dev/null || echo "unknown")
LOCAL_VER=$(cat "$LOCAL_GSTACK/VERSION" 2>/dev/null || echo "unknown")
echo "PRIMARY=$PRIMARY_VER LOCAL=$LOCAL_VER"
```

**バージョンが異なる場合：** 上記のステップ4.5の同期bashブロックに従ってプライマリからローカルコピーを更新します。ユーザーに伝えます：「グローバル v{PRIMARY_VER} は最新です。ローカルvendoredコピーを v{LOCAL_VER} → v{PRIMARY_VER} に更新しました。準備ができたら `.claude/skills/gstack/` をコミットしてください。」

**バージョンが一致する場合：** ユーザーに「最新バージョン（v{PRIMARY_VER}）をお使いです。グローバルとローカルvendoredコピーの両方が最新です。」と伝えます。
