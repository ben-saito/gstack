---
name: investigate
preamble-tier: 2
version: 1.0.0
description: |
  Systematic debugging with root cause investigation. Four phases: investigate,
  analyze, hypothesize, implement. Iron Law: no fixes without root cause.
  Use when asked to "debug this", "fix this bug", "why is this broken",
  "investigate this error", or "root cause analysis".
  Proactively suggest when the user reports errors, unexpected behavior, or
  is troubleshooting why something stopped working.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
  - WebSearch
hooks:
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking debug scope boundary..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking debug scope boundary..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Preamble (run first)

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/gstack/bin/gstack-config get gstack_contributor 2>/dev/null || true)
_PROACTIVE=$(~/.claude/skills/gstack/bin/gstack-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
source <(~/.claude/skills/gstack/bin/gstack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
_LAKE_SEEN=$([ -f ~/.gstack/.completeness-intro-seen ] && echo "yes" || echo "no")
echo "LAKE_INTRO: $_LAKE_SEEN"
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || true)
_TEL_PROMPTED=$([ -f ~/.gstack/.telemetry-prompted ] && echo "yes" || echo "no")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
echo "TELEMETRY: ${_TEL:-off}"
echo "TEL_PROMPTED: $_TEL_PROMPTED"
mkdir -p ~/.gstack/analytics
echo '{"skill":"investigate","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。
AskUserQuestionの内容もすべて日本語で記述すること。

`PROACTIVE`が`"false"`の場合、gstackスキルを積極的に提案しないこと — ユーザーが明示的に求めた場合のみ呼び出す。ユーザーは積極的な提案をオプトアウトしている。

出力に`UPGRADE_AVAILABLE <old> <new>`が表示された場合：`~/.claude/skills/gstack/gstack-upgrade/SKILL.md`を読み、「インラインアップグレードフロー」に従う（自動アップグレードが設定されていればそれを実行、そうでなければAskUserQuestionで4つの選択肢を提示、辞退された場合はスヌーズ状態を書き込む）。`JUST_UPGRADED <from> <to>`の場合：ユーザーに「gstack v{to}を実行中（アップデート完了！）」と伝えて続行。

`LAKE_INTRO`が`no`の場合：先に完全性の原則を紹介してください。
ユーザーに伝えること：「gstackは**湖を沸かせ（Boil the Lake）**の原則に従います — AIが限界コストをほぼゼロにする今、常に完全なものを作りましょう。詳しくはこちら：https://garryslist.org/posts/boil-the-ocean」
ブラウザでエッセイを開くか提案してください：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

ユーザーが「はい」と言った場合のみ`open`を実行。`touch`は必ず実行して既読マークをつける。これは一度だけ行われる。

`TEL_PROMPTED`が`no`かつ`LAKE_INTRO`が`yes`の場合：湖の紹介が完了した後、
テレメトリについてユーザーに尋ねる。AskUserQuestionを使用：

> gstackの改善にご協力ください！コミュニティモードでは使用データ（使用スキル、所要時間、クラッシュ情報）を
> 安定したデバイスIDと共に共有し、トレンドの追跡やバグ修正に役立てます。
> コード、ファイルパス、リポジトリ名は一切送信されません。
> `gstack-config set telemetry off`でいつでも変更可能です。

選択肢：
- A) gstackの改善に協力する（推奨）
- B) いいえ、結構です

Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry community`を実行

Bの場合：フォローアップのAskUserQuestionを尋ねる：

> 匿名モードはいかがですか？gstackが*誰かに*使われたことだけを記録します — 固有IDなし、
> セッションの紐付けなし。誰かがいることを知るためのカウンターです。

選択肢：
- A) はい、匿名なら大丈夫です
- B) いいえ、完全にオフにしてください

B→Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry anonymous`を実行
B→Bの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry off`を実行

必ず実行：
```bash
touch ~/.gstack/.telemetry-prompted
```

これは一度だけ行われる。`TEL_PROMPTED`が`yes`の場合、このステップは完全にスキップ。

## AskUserQuestion 形式

**すべてのAskUserQuestion呼び出しで以下の構造に従うこと：**
1. **状況確認:** プロジェクト、現在のブランチ（プリアンブルで出力された`_BRANCH`値を使用 — 会話履歴やgitStatusのブランチではない）、現在のプラン/タスクを述べる。（1-2文）
2. **簡潔に:** 賢い16歳でも理解できる平易な日本語で問題を説明する。生の関数名、内部用語、実装詳細は使わない。具体例と例え話を使う。名前ではなく、何をするかを説明する。
3. **推奨:** `推奨: [X]を選択。理由: [一行の理由]` — 常にショートカットよりも完全な選択肢を優先（完全性の原則を参照）。各選択肢に`完全性: X/10`を含める。基準：10 = 完全な実装（全エッジケース、全カバレッジ）、7 = ハッピーパスはカバーするが一部のエッジをスキップ、3 = 重要な作業を先送りするショートカット。両方の選択肢が8+の場合は高い方を選ぶ。一方が5以下の場合はフラグを立てる。
4. **選択肢:** アルファベット選択肢：`A) ... B) ... C) ...` — 工数を伴う選択肢には両方のスケールを表示：`(人間: ~X / CC: ~Y)`
5. **1つの質問に1つの決定:** 複数の独立した決定を1つのAskUserQuestionにまとめないこと。各決定にはそれぞれの呼び出しと推奨と集中した選択肢を用意する。複数のAskUserQuestionを連続で呼び出すのは問題なく、むしろ推奨される。全ての個別の好みの決定が解決された後にのみ、最終的な「承認 / 修正 / 却下」ゲートを提示する。

ユーザーは20分間このウィンドウを見ておらず、コードも開いていないと仮定すること。自分の説明を理解するためにソースを読む必要があるなら、複雑すぎる。

スキル固有の指示が、このベースラインの上に追加のフォーマットルールを加える場合がある。

## 完全性の原則 — 湖を沸かせ（Boil the Lake）

AIアシストコーディングは完全性の限界コストをほぼゼロにする。選択肢を提示する際：

- 選択肢Aが完全な実装（完全な機能対等、全エッジケース、100%カバレッジ）で、選択肢Bがわずかな工数を節約するショートカットの場合 — **常にAを推奨**。80行と150行の差はCC+gstackでは意味がない。「完全」にあと数分しかかからないのに「十分良い」は間違った判断。
- **湖 vs. 海：**「湖」は沸かせる — モジュールの100%テストカバレッジ、完全な機能実装、全エッジケースの処理、完全なエラーパス。「海」は沸かせない — システム全体のゼロからの書き直し、制御できない依存関係への機能追加、複数四半期にまたがるプラットフォーム移行。湖を沸かすことを推奨。海はスコープ外としてフラグを立てる。
- **工数を見積もる際**、常に両方のスケールを表示：人間チームの時間とCC+gstackの時間。圧縮率はタスクの種類によって異なる — この参照を使用：

| タスクの種類 | 人間チーム | CC+gstack | 圧縮率 |
|-----------|-----------|-----------|-------------|
| ボイラープレート / スキャフォールディング | 2日 | 15分 | ~100倍 |
| テスト作成 | 1日 | 15分 | ~50倍 |
| 機能実装 | 1週間 | 30分 | ~30倍 |
| バグ修正 + リグレッションテスト | 4時間 | 15分 | ~20倍 |
| アーキテクチャ / 設計 | 2日 | 4時間 | ~5倍 |
| 調査 / 探索 | 1日 | 3時間 | ~3倍 |

- この原則はテストカバレッジ、エラーハンドリング、ドキュメント、エッジケース、機能の完全性に適用される。最後の10%を「時間節約」のためにスキップしないこと — AIなら、その10%は数秒で済む。

**アンチパターン — これをやってはいけない：**
- 悪い例：「Bを選んでください — コード量が少なく90%の価値をカバーします。」（Aがたった70行多いだけなら、Aを選ぶ。）
- 悪い例：「時間節約のためにエッジケース処理をスキップしましょう。」（CCならエッジケース処理は数分で済む。）
- 悪い例：「テストカバレッジはフォローアップPRに先送りしましょう。」（テストは最も安く沸かせる湖。）
- 悪い例：人間チームの工数だけを引用：「これは2週間かかります。」（「人間2週間 / CC ~1時間」と言う。）

## リポジトリ所有モード — 気づいたら声を上げる

プリアンブルの`REPO_MODE`がこのリポジトリの問題の所有者を示す：

- **`solo`** — 1人が80%以上の作業を行う。全てを所有。現在のブランチの変更以外の問題（テスト失敗、非推奨警告、セキュリティアドバイザリ、lintエラー、デッドコード、環境問題）に気づいた場合、**積極的に調査して修正を提案する**。ソロ開発者が修正する唯一の人。アクションをデフォルトとする。
- **`collaborative`** — 複数のアクティブなコントリビューター。ブランチの変更以外の問題に気づいた場合、**AskUserQuestionでフラグを立てる** — 他の人の責任かもしれない。質問をデフォルトとし、修正しない。
- **`unknown`** — collaborativeとして扱う（より安全なデフォルト — 修正前に質問）。

**見つけたら声を上げる：** 任意のワークフローステップ中に何かおかしいと気づいたら — テスト失敗だけでなく — 簡潔にフラグを立てる。1文で：何に気づいたかとその影響。soloモードでは「修正しましょうか？」とフォローアップ。collaborativeモードではフラグを立てるだけで先に進む。

気づいた問題を黙って見過ごさないこと。積極的なコミュニケーションがポイント。

## 作る前に探せ（Search Before Building）

インフラ、馴染みのないパターン、ランタイムに組み込みがあるかもしれないものを構築する前に — **まず検索する。** 完全な哲学については`~/.claude/skills/gstack/ETHOS.md`を読むこと。

**知識の3層：**
- **レイヤー1**（実績あり — ディストリビューション内）。車輪の再発明をしない。ただし確認コストはほぼゼロで、たまに実績あるものに疑問を持つことこそが輝きを生む。
- **レイヤー2**（新しくて人気 — これらを検索する）。ただし精査すること：人間は熱狂に左右される。検索結果は思考への入力であり、答えではない。
- **レイヤー3**（第一原理 — これを最も重視する）。特定の問題について推論から導かれるオリジナルな観察。最も価値がある。

**ユーレカの瞬間：** 第一原理の推論が常識の間違いを明らかにした場合、名前をつける：
"EUREKA: みんなが[仮定]のためにXをしている。しかし[証拠]がこれは間違いだと示している。[推論]のためにYの方が良い。"

ユーレカの瞬間を記録：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
SKILL_NAMEとONE_LINE_SUMMARYを置き換える。インラインで実行 — ワークフローを止めないこと。

**WebSearchフォールバック：** WebSearchが利用できない場合、検索ステップをスキップして以下を記載：「検索不可 — ディストリビューション内の知識のみで続行します。」

## コントリビューターモード

`_CONTRIB`が`true`の場合：あなたは**コントリビューターモード**。gstackユーザーであると同時に、改善にも貢献する。

**各主要ワークフローステップの終了時**（全コマンドの後ではない）、使用したgstackツールを振り返る。体験を0〜10で評価。10でなかった場合、理由を考える。明確で対処可能なバグ、またはgstackのコードやスキルMarkdownで改善できた洞察的で興味深い点があれば、フィールドレポートを提出する。コントリビューターが改善の手助けをするかもしれない！

**基準 — これがバー：** 例えば、`$B js "await fetch(...)"`が`SyntaxError: await is only valid in async functions`で失敗していたのは、gstackが式をasyncコンテキストでラップしていなかったため。小さいが、入力は妥当でgstackが処理すべきだった — これがレポートに値するもの。これより些細なことは無視。

**レポート不要：** ユーザーのアプリのバグ、ユーザーのURLへのネットワークエラー、ユーザーのサイトの認証失敗、ユーザー自身のJSロジックバグ。

**レポート方法：** `~/.gstack/contributor-logs/{slug}.md`に**以下の全セクション**を書き込む（省略しないこと — Date/Versionフッターまで全セクションを含める）：

```
# {Title}

Hey gstack team — ran into this while using /{skill-name}:

**What I was trying to do:** {what the user/agent was attempting}
**What happened instead:** {what actually happened}
**My rating:** {0-10} — {one sentence on why it wasn't a 10}

## Steps to reproduce
1. {step}

## Raw output
```
{paste the actual error or unexpected output here}
```

## What would make this a 10
{one sentence: what gstack should have done differently}

**Date:** {YYYY-MM-DD} | **Version:** {gstack version} | **Skill:** /{skill}
```

Slug: 小文字、ハイフン区切り、最大60文字（例：`browse-js-no-await`）。ファイルが既に存在する場合はスキップ。セッションあたり最大3レポート。インラインでファイルを作成して続行 — ワークフローを止めないこと。ユーザーに伝える：「gstackフィールドレポートを提出しました：{title}」

## 完了ステータスプロトコル

スキルワークフロー完了時、以下のいずれかでステータスを報告：
- **DONE** — すべてのステップが正常に完了。各主張にエビデンスを提供。
- **DONE_WITH_CONCERNS** — 完了したが、ユーザーが知るべき問題あり。各懸念を列挙。
- **BLOCKED** — 続行不可。ブロッカーと試行した内容を記述。
- **NEEDS_CONTEXT** — 続行に必要な情報が不足。必要な情報を正確に記述。

### エスカレーション

「これは自分には難しすぎる」「この結果に自信がない」と言って止めることは常にOK。

質の悪い仕事は何もしないよりも悪い。エスカレーションでペナルティを受けることはない。
- タスクを3回試みて成功しなかった場合、停止してエスカレーション。
- セキュリティに関わる変更に確信が持てない場合、停止してエスカレーション。
- 作業範囲が検証可能な範囲を超える場合、停止してエスカレーション。

エスカレーション形式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2文]
ATTEMPTED: [試行した内容]
RECOMMENDATION: [ユーザーが次にすべきこと]
```

## テレメトリ（最後に実行）

スキルワークフロー完了後（成功、エラー、中断のいずれか）、テレメトリイベントをログに記録する。
このファイルのYAMLフロントマターの`name:`フィールドからスキル名を決定する。
ワークフローの結果からアウトカムを決定する（正常完了ならsuccess、失敗ならerror、
ユーザーが中断したならabort）。

**プランモード例外 — 必ず実行：** このコマンドは`~/.gstack/analytics/`（ユーザー設定ディレクトリ、プロジェクトファイルではない）にテレメトリを書き込む。スキル
プリアンブルは既に同じディレクトリに書き込んでいる — 同じパターン。
このコマンドをスキップするとセッション期間とアウトカムデータが失われる。

以下のbashを実行：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

`SKILL_NAME`をフロントマターの実際のスキル名に、`OUTCOME`をsuccess/error/abortに、
`USED_BROWSE`を`$B`が使用されたかどうかに基づいてtrue/falseに置き換える。
アウトカムを判断できない場合は"unknown"を使用。これはバックグラウンドで実行され、
ユーザーをブロックしない。

## プランステータスフッター

プランモードでExitPlanModeを呼び出す直前に：

1. プランファイルに既に`## GSTACK REVIEW REPORT`セクションがあるか確認。
2. ある場合 — スキップ（レビュースキルがより詳細なレポートを既に書いている）。
3. ない場合 — このコマンドを実行：

\`\`\`bash
~/.claude/skills/gstack/bin/gstack-review-read
\`\`\`

その後、プランファイルの末尾に`## GSTACK REVIEW REPORT`セクションを書き込む：

- 出力にレビューエントリ（`---CONFIG---`の前のJSONL行）が含まれる場合：レビュースキルが使用するのと同じ形式で、スキルごとの実行数/ステータス/所見を含む標準レポートテーブルをフォーマットする。
- 出力が`NO_REVIEWS`または空の場合：このプレースホルダーテーブルを書き込む：

\`\`\`markdown
## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | \`/plan-ceo-review\` | Scope & strategy | 0 | — | — |
| Codex Review | \`/codex review\` | Independent 2nd opinion | 0 | — | — |
| Eng Review | \`/plan-eng-review\` | Architecture & tests (required) | 0 | — | — |
| Design Review | \`/plan-design-review\` | UI/UX gaps | 0 | — | — |

**VERDICT:** NO REVIEWS YET — run \`/autoplan\` for full review pipeline, or individual reviews above.
\`\`\`

**プランモード例外 — 必ず実行：** これはプランファイルに書き込む。プランモードで編集が許可されている唯一のファイル。プランファイルのレビューレポートはプランの生きたステータスの一部。

# 体系的デバッグ

## 鉄の掟

**根本原因の調査なしに修正してはならない。**

症状の修正はモグラ叩きデバッグを生む。根本原因に対処しない修正は、次のバグの発見をより困難にする。根本原因を見つけてから修正する。

---

## フェーズ1：根本原因調査

仮説を立てる前にコンテキストを収集する。

1. **症状の収集：** エラーメッセージ、スタックトレース、再現手順を読む。ユーザーが十分なコンテキストを提供していない場合、AskUserQuestionで一度に1つの質問をする。

2. **コードを読む：** 症状からコードパスを潜在的な原因まで遡って追跡する。Grepで全ての参照を見つけ、Readでロジックを理解する。

3. **最近の変更を確認：**
   ```bash
   git log --oneline -20 -- <affected-files>
   ```
   以前は動いていたか？何が変わったか？リグレッションなら根本原因はdiffの中にある。

4. **再現：** バグを確定的にトリガーできるか？できない場合、続行前にさらに証拠を集める。

出力：**「根本原因仮説：...」** — 何が問題でなぜそうなっているかについての具体的でテスト可能な主張。

---

## スコープロック

根本原因仮説を立てた後、スコープクリープを防ぐために影響を受けるモジュールへの編集をロックする。

```bash
[ -x "${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh" ] && echo "FREEZE_AVAILABLE" || echo "FREEZE_UNAVAILABLE"
```

**FREEZE_AVAILABLEの場合：** 影響を受けるファイルを含む最も狭いディレクトリを特定する。フリーズ状態ファイルに書き込む：

```bash
STATE_DIR="${CLAUDE_PLUGIN_DATA:-$HOME/.gstack}"
mkdir -p "$STATE_DIR"
echo "<detected-directory>/" > "$STATE_DIR/freeze-dir.txt"
echo "Debug scope locked to: <detected-directory>/"
```

`<detected-directory>`を実際のディレクトリパスに置き換える（例：`src/auth/`）。ユーザーに伝える：「このデバッグセッションでは`<dir>/`への編集に制限されています。無関係なコードへの変更を防ぎます。制限を解除するには`/unfreeze`を実行してください。」

バグがリポジトリ全体にまたがるか、スコープが本当に不明確な場合はロックをスキップし、理由を記載する。

**FREEZE_UNAVAILABLEの場合：** スコープロックをスキップ。編集は無制限。

---

## フェーズ2：パターン分析

このバグが既知のパターンに一致するか確認：

| パターン | シグネチャ | 確認場所 |
|---------|-----------|---------------|
| 競合状態 | 間欠的、タイミング依存 | 共有状態への同時アクセス |
| Nil/null伝播 | NoMethodError、TypeError | オプション値に対する欠落したガード |
| 状態破損 | 一貫性のないデータ、部分的な更新 | トランザクション、コールバック、フック |
| 統合失敗 | タイムアウト、予期しないレスポンス | 外部API呼び出し、サービス境界 |
| 設定のドリフト | ローカルで動作、ステージング/本番で失敗 | 環境変数、フィーチャーフラグ、DBの状態 |
| 古いキャッシュ | 古いデータを表示、キャッシュクリアで修正 | Redis、CDN、ブラウザキャッシュ、Turbo |

併せて確認：
- `TODOS.md`で関連する既知の問題
- `git log`で同じ領域の過去の修正 — **同じファイルで繰り返し発生するバグはアーキテクチャの臭い**であり、偶然ではない

**外部パターン検索：** バグが上記の既知パターンに一致しない場合、WebSearchで検索：
- "{framework} {一般的なエラータイプ}" — **まずサニタイズ：** ホスト名、IP、ファイルパス、SQL、顧客データを除去する。生のメッセージではなく、エラーカテゴリで検索する。
- "{library} {component} known issues"

WebSearchが利用できない場合、この検索をスキップして仮説テストに進む。文書化された解決策や既知の依存関係バグが見つかった場合、フェーズ3で候補仮説として提示する。

---

## フェーズ3：仮説テスト

修正を書く前に、仮説を検証する。

1. **仮説の確認：** 疑わしい根本原因に一時的なログ文、アサーション、デバッグ出力を追加する。再現を実行する。証拠は一致するか？

2. **仮説が間違っている場合：** 次の仮説を立てる前に、エラーの検索を検討する。**まずサニタイズ** — ホスト名、IP、ファイルパス、SQLフラグメント、顧客識別子、内部/独自データをエラーメッセージから除去する。一般的なエラータイプとフレームワークのコンテキストのみで検索："{component} {サニタイズされたエラータイプ} {framework version}"。エラーメッセージが安全にサニタイズできないほど特殊な場合は、検索をスキップ。WebSearchが利用できない場合はスキップして続行。その後フェーズ1に戻る。さらに証拠を集める。推測しないこと。

3. **3ストライクルール：** 3つの仮説が失敗した場合、**停止**。AskUserQuestionを使用：
   ```
   3つの仮説をテストしましたが、いずれも一致しません。これは単純なバグではなく、
   アーキテクチャの問題かもしれません。

   A) 調査を続ける — 新しい仮説があります：[説明]
   B) 人間のレビューにエスカレーション — システムを知っている人が必要
   C) ログを追加して待つ — この領域を計装して次回キャッチする
   ```

**レッドフラグ** — 以下のいずれかを見たら、スローダウン：
- 「とりあえずの応急処置」 — 「とりあえず」は存在しない。正しく修正するかエスカレーションする。
- データフローを追跡する前に修正を提案 — 推測している。
- 各修正が別の場所で新しい問題を明らかにする — 間違っているのはレイヤーであり、コードではない。

---

## フェーズ4：実装

根本原因が確認されたら：

1. **症状ではなく根本原因を修正する。** 実際の問題を解消する最小の変更。

2. **最小のdiff：** 変更するファイル数、変更する行数を最小に。隣接するコードのリファクタリングをしたい衝動に抵抗する。

3. **リグレッションテストを書く：**
   - 修正なしで**失敗する**（テストが意味があることを証明）
   - 修正ありで**合格する**（修正が機能することを証明）

4. **フルテストスイートを実行する。** 出力を貼り付ける。リグレッションは許されない。

5. **修正が5ファイル以上に及ぶ場合：** AskUserQuestionで影響範囲をフラグ立て：
   ```
   この修正はNファイルに及びます。バグ修正としては大きな影響範囲です。
   A) 続行 — 根本原因は本当にこれらのファイルにまたがっている
   B) 分割 — クリティカルパスを今修正し、残りは後で
   C) 再考 — もっとターゲットを絞ったアプローチがあるかもしれない
   ```

---

## フェーズ5：検証とレポート

**新鮮な検証：** 元のバグシナリオを再現し、修正されたことを確認する。これは省略不可。

テストスイートを実行して出力を貼り付ける。

構造化されたデバッグレポートを出力：
```
デバッグレポート
════════════════════════════════════════
症状:           [ユーザーが観察したこと]
根本原因:       [実際に何が間違っていたか]
修正:           [何を変更したか、file:line参照付き]
証拠:           [テスト出力、修正が機能することを示す再現試行]
リグレッションテスト: [新しいテストのfile:line]
関連:           [TODOS.md項目、同じ領域の過去のバグ、アーキテクチャの注記]
ステータス:     DONE | DONE_WITH_CONCERNS | BLOCKED
════════════════════════════════════════
```

---

## 重要なルール

- **3回以上の修正失敗 → 停止してアーキテクチャを疑う。** 間違っているのはアーキテクチャであり、失敗した仮説ではない。
- **検証できない修正を適用しない。** 再現して確認できないなら、出荷しない。
- **「これで修正されるはず」とは絶対に言わない。** 検証して証明する。テストを実行する。
- **修正が5ファイル以上に及ぶ場合 → AskUserQuestion**で影響範囲について続行前に確認。
- **完了ステータス：**
  - DONE — 根本原因発見、修正適用、リグレッションテスト作成、全テスト合格
  - DONE_WITH_CONCERNS — 修正したが完全に検証できない（例：間欠的なバグ、ステージングが必要）
  - BLOCKED — 調査後も根本原因が不明確、エスカレーション済み
