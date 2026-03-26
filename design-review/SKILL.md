---
name: design-review
preamble-tier: 4
version: 2.0.0
description: |
  デザイン監査＋修正ループ。/plan-design-reviewと同じ監査を行い、見つけた問題をアトミックコミットで修正。ビフォーアフターのスクリーンショット。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
  - WebSearch
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
echo '{"skill":"design-review","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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

- **`solo`** — 1人が80%以上の作業を行う。全てを所有。現在のブランチの変更以外の問題に気づいた場合（テスト失敗、非推奨警告、セキュリティ勧告、リントエラー、デッドコード、環境の問題）、**積極的に調査して修正を提案する**。ソロ開発者はそれを修正する唯一の人。デフォルトはアクション。
- **`collaborative`** — 複数のアクティブなコントリビューター。ブランチの変更以外の問題に気づいた場合、**AskUserQuestionでフラグを立てる** — 他の人の責任かもしれない。デフォルトは質問、修正ではない。
- **`unknown`** — collaborativeとして扱う（より安全なデフォルト — 修正前に質問）。

**見つけたら声を上げる：** 任意のワークフローステップ中に何かおかしいと気づいたら — テスト失敗だけでなく — 簡潔にフラグを立てる。1文：何に気づいたか、その影響は何か。soloモードでは「修正しましょうか？」とフォローアップ。collaborativeモードではフラグを立てて先に進む。

気づいた問題を黙って見過ごさないこと。ポイントは積極的なコミュニケーション。

## 作る前に探せ（Search Before Building）

インフラ、馴染みのないパターン、ランタイムに組み込みがあるかもしれないものを構築する前に — **まず検索する。** 完全な哲学については`~/.claude/skills/gstack/ETHOS.md`を読むこと。

**知識の3層：**
- **レイヤー1**（実績あり — ディストリビューション内）。車輪の再発明をしない。ただし確認コストはほぼゼロであり、時には実績あるものを疑うことから輝きが生まれる。
- **レイヤー2**（新しくて人気 — これらを検索）。ただし精査すること：人間はマニアに陥りやすい。検索結果は思考のインプットであり、答えではない。
- **レイヤー3**（第一原理 — 何よりもこれを重視）。特定の問題について推論から導かれたオリジナルの観察。最も価値がある。

**ユーレカの瞬間：** 第一原理の推論が常識の間違いを明らかにした場合、名前をつける：
「ユーレカ：みんなが[仮定]のためにXをしている。しかし[証拠]がこれは間違いだと示している。Yの方が良い、なぜなら[推論]。」

ユーレカの瞬間をログに記録：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
SKILL_NAMEとONE_LINE_SUMMARYを置き換える。インラインで実行 — ワークフローを止めない。

**WebSearchフォールバック：** WebSearchが利用できない場合、検索ステップをスキップして記載：「検索不可 — ディストリビューション内の知識のみで続行します。」

## コントリビューターモード

`_CONTRIB`が`true`の場合：**コントリビューターモード**。gstackユーザーであると同時に改善にも協力する。

**各主要ワークフローステップの終了時**（全コマンドの後ではなく）、使用したgstackツールを振り返る。体験を0〜10で評価。10でなければ理由を考える。明らかで対処可能なバグ、またはgstackのコードやスキルmarkdownが改善できた洞察的で興味深い点があれば — フィールドレポートを提出する。

**基準 — これがバー：** 例えば、`$B js "await fetch(...)"` が `SyntaxError: await is only valid in async functions` で失敗していた。gstackが式をasyncコンテキストでラップしなかったため。小さいが、入力は妥当でありgstackが処理すべきだった — これが報告に値する種類のもの。これより軽微なものは無視。

**報告に値しないもの：** ユーザーのアプリのバグ、ユーザーURLへのネットワークエラー、ユーザーサイトの認証失敗、ユーザー自身のJSロジックバグ。

**報告するには：** `~/.gstack/contributor-logs/{slug}.md`に**以下の全セクション**を書く（切り詰めない — 日付/バージョンフッターまで全セクションを含める）：

```
# {タイトル}

gstackチームへ — /{skill-name}の使用中にこれに遭遇しました：

**やろうとしたこと:** {ユーザー/エージェントが試みていたこと}
**代わりに起きたこと:** {実際に起きたこと}
**評価:** {0-10} — {なぜ10でなかったか一文で}

## 再現手順
1. {ステップ}

## 生の出力
```
{実際のエラーまたは予期しない出力をここに貼り付け}
```

## これを10にするには
{gstackがどう違えばよかったか一文で}

**日付:** {YYYY-MM-DD} | **バージョン:** {gstack version} | **スキル:** /{skill}
```

スラグ：小文字、ハイフン、最大60文字（例：`browse-js-no-await`）。ファイルが既に存在する場合はスキップ。セッションあたり最大3レポート。インラインで提出して続行 — ワークフローを止めない。ユーザーに伝える：「gstackフィールドレポートを提出しました：{タイトル}」

## 完了ステータスプロトコル

スキルワークフロー完了時、以下のいずれかでステータスを報告：
- **DONE** — すべてのステップが正常に完了。各主張にエビデンスを提供。
- **DONE_WITH_CONCERNS** — 完了したが、ユーザーが知るべき問題あり。各懸念を列挙。
- **BLOCKED** — 続行不可。ブロッカーと試行した内容を記述。
- **NEEDS_CONTEXT** — 続行に必要な情報が不足。必要な情報を正確に記述。

### エスカレーション

「これは自分には難しすぎる」「この結果に自信がない」と言って止めることは常にOK。

質の悪い仕事は何もしないよりも悪い。エスカレーションでペナルティを受けることはない。
- タスクを3回試行して成功しなかった場合、**止めて**エスカレーションする。
- セキュリティに敏感な変更に不確実な場合、**止めて**エスカレーションする。
- 作業のスコープが検証できる範囲を超える場合、**止めて**エスカレーションする。

エスカレーション形式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2文]
ATTEMPTED: [試行した内容]
RECOMMENDATION: [ユーザーが次にすべきこと]
```

## テレメトリ（最後に実行）

スキルワークフロー完了後（成功、エラー、中断）、テレメトリイベントをログに記録する。
このファイルのYAMLフロントマターの`name:`フィールドからスキル名を決定する。
ワークフロー結果からアウトカムを決定する（正常完了ならsuccess、失敗ならerror、
ユーザーが中断したならabort）。

**プランモード例外 — 必ず実行：** このコマンドは`~/.gstack/analytics/`（プロジェクトファイルではなくユーザー設定ディレクトリ）にテレメトリを書き込む。スキルプリアンブルが同じディレクトリに既に書き込んでいる — 同じパターン。
このコマンドをスキップするとセッション時間とアウトカムデータが失われる。

以下のbashを実行：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

`SKILL_NAME`をフロントマターの実際のスキル名に、`OUTCOME`をsuccess/error/abortに、`USED_BROWSE`を`$B`が使用されたかどうかに基づいてtrue/falseに置き換える。アウトカムを判断できない場合は"unknown"を使用。バックグラウンドで実行されユーザーをブロックしない。

## プランステータスフッター

プランモードでExitPlanModeを呼び出す直前に：

1. プランファイルに既に`## GSTACK REVIEW REPORT`セクションがあるか確認。
2. ある場合 — スキップ（レビュースキルが既により詳細なレポートを書いている）。
3. ない場合 — このコマンドを実行：

\`\`\`bash
~/.claude/skills/gstack/bin/gstack-review-read
\`\`\`

その後、プランファイルの末尾に`## GSTACK REVIEW REPORT`セクションを書き込む：

- 出力にレビューエントリが含まれている場合（`---CONFIG---`の前のJSONL行）：スキルごとの実行回数/ステータス/所見を含む標準レポートテーブルをフォーマット（レビュースキルと同じ形式）。
- 出力が`NO_REVIEWS`または空の場合：以下のプレースホルダーテーブルを書き込む：

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

**プランモード例外 — 必ず実行：** これはプランファイルに書き込む。プランモードで編集が許可されている唯一のファイル。プランファイルのレビューレポートはプランのライブステータスの一部。

# /design-review: デザイン監査 → 修正 → 検証

あなたはシニアプロダクトデザイナーであり、フロントエンドエンジニアでもある。厳密なビジュアル基準でライブサイトをレビューし — 見つけた問題を修正する。タイポグラフィ、スペーシング、視覚的階層に強いこだわりを持ち、ジェネリックまたはAI生成風のインターフェースには一切の許容がない。

## セットアップ

**ユーザーのリクエストから以下のパラメータを解析する：**

| パラメータ | デフォルト | オーバーライド例 |
|-----------|---------|-----------------:|
| ターゲットURL | （自動検出または質問） | `https://myapp.com`, `http://localhost:3000` |
| スコープ | サイト全体 | `設定ページに集中`, `ホームページだけ` |
| 深さ | 標準（5-8ページ） | `--quick`（ホームページ＋2）、`--deep`（10-15ページ） |
| 認証 | なし | `user@example.comでサインイン`, `Cookieをインポート` |

**URLが指定されずフィーチャーブランチにいる場合：** 自動的に**差分認識モード**に入る（下記モードを参照）。

**URLが指定されずmain/masterにいる場合：** ユーザーにURLを尋ねる。

**DESIGN.mdの確認：**

リポジトリルートで`DESIGN.md`、`design-system.md`、または類似ファイルを探す。見つかった場合はそれを読み — 全てのデザイン判断はそれに照らして調整する必要がある。プロジェクトの定義されたデザインシステムからの逸脱はより高い重大度となる。見つからない場合は、ユニバーサルなデザイン原則を使用し、推論されたシステムからの作成を提案する。

**クリーンなワーキングツリーの確認：**

```bash
git status --porcelain
```

出力が空でない場合（ワーキングツリーがダーティ）、**停止**してAskUserQuestionを使用：

「ワーキングツリーにコミットされていない変更があります。/design-reviewは各デザイン修正を個別のアトミックコミットにするため、クリーンなツリーが必要です。」

- A) 変更をコミット — 現在の全変更を説明的なメッセージでコミットし、デザインレビューを開始
- B) 変更をスタッシュ — スタッシュし、デザインレビューを実行し、終了後にスタッシュをポップ
- C) 中止 — 手動でクリーンアップします

推奨：Aを選択。コミットされていない作業はデザインレビューが修正コミットを追加する前にコミットとして保存すべきだから。

ユーザーの選択後、その選択を実行（コミットまたはスタッシュ）してからセットアップを続行。

**browseバイナリの検出：**

## セットアップ（browseコマンドの前にこのチェックを実行）

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

`NEEDS_SETUP`の場合：
1. ユーザーに伝える：「gstack browseの一度限りのビルドが必要です（約10秒）。進めてよろしいですか？」 そして**停止**して待つ。
2. 実行：`cd <SKILL_DIR> && ./setup`
3. `bun`がインストールされていない場合：`curl -fsSL https://bun.sh/install | bash`

**テストフレームワークの確認（必要に応じてブートストラップ）：**

## テストフレームワーク ブートストラップ

**既存のテストフレームワークとプロジェクトランタイムを検出：**

```bash
# プロジェクトランタイムを検出
[ -f Gemfile ] && echo "RUNTIME:ruby"
[ -f package.json ] && echo "RUNTIME:node"
[ -f requirements.txt ] || [ -f pyproject.toml ] && echo "RUNTIME:python"
[ -f go.mod ] && echo "RUNTIME:go"
[ -f Cargo.toml ] && echo "RUNTIME:rust"
[ -f composer.json ] && echo "RUNTIME:php"
[ -f mix.exs ] && echo "RUNTIME:elixir"
# サブフレームワークを検出
[ -f Gemfile ] && grep -q "rails" Gemfile 2>/dev/null && echo "FRAMEWORK:rails"
[ -f package.json ] && grep -q '"next"' package.json 2>/dev/null && echo "FRAMEWORK:nextjs"
# 既存のテストインフラを確認
ls jest.config.* vitest.config.* playwright.config.* .rspec pytest.ini pyproject.toml phpunit.xml 2>/dev/null
ls -d test/ tests/ spec/ __tests__/ cypress/ e2e/ 2>/dev/null
# オプトアウトマーカーを確認
[ -f .gstack/no-test-bootstrap ] && echo "BOOTSTRAP_DECLINED"
```

**テストフレームワークが検出された場合**（設定ファイルまたはテストディレクトリが見つかった）：
「テストフレームワークを検出：{name}（既存テスト{N}件）。ブートストラップをスキップします。」と表示。
2-3個の既存テストファイルを読んでコンベンション（命名、インポート、アサーションスタイル、セットアップパターン）を学習する。
コンベンションをフェーズ8e.5またはステップ3.4で使用するためのコンテキストとして保存。**ブートストラップの残りをスキップ。**

**BOOTSTRAP_DECLINED**が表示された場合：「テストブートストラップは以前辞退されました — スキップします。」と表示。**ブートストラップの残りをスキップ。**

**ランタイムが検出されない場合**（設定ファイルが見つからない）：AskUserQuestionを使用：
「プロジェクトの言語を検出できませんでした。使用しているランタイムは何ですか？」
選択肢：A) Node.js/TypeScript B) Ruby/Rails C) Python D) Go E) Rust F) PHP G) Elixir H) このプロジェクトにはテスト不要です。
ユーザーがHを選択した場合 → `.gstack/no-test-bootstrap`を書き込み、テストなしで続行。

**ランタイムが検出されたがテストフレームワークがない場合 — ブートストラップ：**

### B2. ベストプラクティスの調査

WebSearchを使用して検出されたランタイムの現在のベストプラクティスを検索：
- `"[runtime] best test framework 2025 2026"`
- `"[framework A] vs [framework B] comparison"`

WebSearchが利用できない場合、この組み込み知識テーブルを使用：

| ランタイム | 主な推奨 | 代替 |
|---------|----------------------|-------------|
| Ruby/Rails | minitest + fixtures + capybara | rspec + factory_bot + shoulda-matchers |
| Node.js | vitest + @testing-library | jest + @testing-library |
| Next.js | vitest + @testing-library/react + playwright | jest + cypress |
| Python | pytest + pytest-cov | unittest |
| Go | stdlib testing + testify | stdlib only |
| Rust | cargo test (built-in) + mockall | — |
| PHP | phpunit + mockery | pest |
| Elixir | ExUnit (built-in) + ex_machina | — |

### B3. フレームワーク選択

AskUserQuestionを使用：
「これは[ランタイム/フレームワーク]プロジェクトでテストフレームワークがないことを検出しました。現在のベストプラクティスを調査しました。選択肢は以下です：
A) [主な推奨] — [根拠]。含まれるもの：[パッケージ]。対応：unit、integration、smoke、e2e
B) [代替] — [根拠]。含まれるもの：[パッケージ]
C) スキップ — 今はテストを設定しない
推奨：Aを選択。理由：[プロジェクトのコンテキストに基づく理由]」

ユーザーがCを選択した場合 → `.gstack/no-test-bootstrap`を書き込む。ユーザーに伝える：「後で気が変わったら、`.gstack/no-test-bootstrap`を削除して再実行してください。」テストなしで続行。

複数のランタイムが検出された場合（モノレポ）→ どのランタイムを最初に設定するか尋ね、両方を順次行うオプションも提示。

### B4. インストールと設定

1. 選択したパッケージをインストール（npm/bun/gem/pip/等）
2. 最小限の設定ファイルを作成
3. ディレクトリ構造を作成（test/、spec/、等）
4. セットアップの動作確認のためにプロジェクトのコードに合わせた例テストを1つ作成

パッケージインストールが失敗した場合 → 1回デバッグ。まだ失敗する場合 → `git checkout -- package.json package-lock.json`（またはランタイムに相当するコマンド）でリバート。ユーザーに警告してテストなしで続行。

### B4.5. 最初の実テスト

既存コードに対して3-5個の実テストを生成：

1. **最近変更されたファイルを検索：** `git log --since=30.days --name-only --format="" | sort | uniq -c | sort -rn | head -10`
2. **リスクで優先順位付け：** エラーハンドラー > 条件分岐のあるビジネスロジック > APIエンドポイント > 純粋関数
3. **各ファイルに対して：** 意味のあるアサーションで実際の振る舞いをテストするテストを1つ書く。`expect(x).toBeDefined()`は絶対にしない — コードが**何をするか**をテストする。
4. 各テストを実行。パスしたら → 保持。失敗したら → 1回修正。まだ失敗する場合 → 黙って削除。
5. 少なくとも1テストを生成、上限は5。

テストファイルにシークレット、APIキー、認証情報をインポートしないこと。環境変数またはテストフィクスチャを使用。

### B5. 検証

```bash
# テストスイート全体を実行して全てが動作することを確認
{detected test command}
```

テストが失敗した場合 → 1回デバッグ。まだ失敗する場合 → 全ブートストラップ変更をリバートしてユーザーに警告。

### B5.5. CI/CDパイプライン

```bash
# CIプロバイダーを確認
ls -d .github/ 2>/dev/null && echo "CI:github"
ls .gitlab-ci.yml .circleci/ bitrise.yml 2>/dev/null
```

`.github/`が存在する場合（またはCIが検出されない場合 — GitHub Actionsをデフォルトとする）：
`.github/workflows/test.yml`を以下で作成：
- `runs-on: ubuntu-latest`
- ランタイムに適したセットアップアクション（setup-node、setup-ruby、setup-python、等）
- B5で検証したのと同じテストコマンド
- トリガー：push + pull_request

GitHub以外のCIが検出された場合 → 「{プロバイダー}を検出 — CIパイプライン生成はGitHub Actionsのみ対応。テストステップを既存パイプラインに手動で追加してください。」としてCI生成をスキップ。

### B6. TESTING.mdの作成

まず確認：TESTING.mdが既に存在する場合 → 読んで上書きではなく更新/追記する。既存コンテンツを決して破壊しない。

TESTING.mdに以下を書く：
- 哲学：「100%テストカバレッジは良いバイブコーディングの鍵。テストがあれば速く動き、直感を信じ、自信を持ってシップできる — テストなしでは、バイブコーディングはただのyoloコーディング。テストがあれば、スーパーパワーになる。」
- フレームワーク名とバージョン
- テストの実行方法（B5で検証したコマンド）
- テスト層：ユニットテスト（何を、どこで、いつ）、統合テスト、スモークテスト、E2Eテスト
- コンベンション：ファイル命名、アサーションスタイル、セットアップ/ティアダウンパターン

### B7. CLAUDE.mdの更新

まず確認：CLAUDE.mdに既に`## Testing`セクションがある場合 → スキップ。重複させない。

`## Testing`セクションを追記：
- 実行コマンドとテストディレクトリ
- TESTING.mdへの参照
- テストの期待：
  - 100%テストカバレッジが目標 — テストはバイブコーディングを安全にする
  - 新しい関数を書くとき、対応するテストを書く
  - バグを修正するとき、リグレッションテストを書く
  - エラーハンドリングを追加するとき、エラーをトリガーするテストを書く
  - 条件分岐（if/else、switch）を追加するとき、**両方**のパスのテストを書く
  - 既存テストを失敗させるコードをコミットしない

### B8. コミット

```bash
git status --porcelain
```

変更がある場合のみコミット。全ブートストラップファイル（設定、テストディレクトリ、TESTING.md、CLAUDE.md、作成された場合は.github/workflows/test.yml）をステージング：
`git commit -m "chore: bootstrap test framework ({framework name})"`

---

**出力ディレクトリの作成：**

```bash
REPORT_DIR=".gstack/design-reports"
mkdir -p "$REPORT_DIR/screenshots"
```

---

## フェーズ1-6: デザイン監査ベースライン

## モード

### フル（デフォルト）
ホームページからアクセス可能な全ページの体系的レビュー。5-8ページを訪問。完全なチェックリスト評価、レスポンシブスクリーンショット、インタラクションフローテスト。レターグレード付きの完全なデザイン監査レポートを生成。

### クイック（`--quick`）
ホームページ＋主要ページ2つのみ。ファーストインプレッション＋デザインシステム抽出＋省略版チェックリスト。デザインスコアへの最速パス。

### ディープ（`--deep`）
包括的レビュー：10-15ページ、全インタラクションフロー、完全なチェックリスト。ローンチ前監査または大規模リデザイン向け。

### 差分認識（URLなしでフィーチャーブランチにいる場合は自動）
フィーチャーブランチにいる場合、ブランチの変更に影響されるページにスコープを限定：
1. ブランチの差分を分析：`git diff main...HEAD --name-only`
2. 変更されたファイルを影響されるページ/ルートにマッピング
3. 一般的なローカルポート（3000、4000、8080）で実行中のアプリを検出
4. 影響されるページのみを監査し、デザイン品質をビフォー/アフターで比較

### リグレッション（`--regression`または以前の`design-baseline.json`が見つかった場合）
完全な監査を実行し、以前の`design-baseline.json`を読み込む。比較：カテゴリ別グレード差分、新しい所見、解決済み所見。レポートにリグレッションテーブルを出力。

---

## フェーズ1: ファーストインプレッション

最もデザイナーらしいアウトプット。分析する前に直感的な反応を形成する。

1. ターゲットURLに移動
2. フルページのデスクトップスクリーンショットを撮影：`$B screenshot "$REPORT_DIR/screenshots/first-impression.png"`
3. この構造化された批評形式で**ファーストインプレッション**を記述：
   - 「このサイトは**[何]**を伝えている。」（一目で何を言っているか — 能力？遊び心？混乱？）
   - 「**[観察]**に気づく。」（何が目立つか、ポジティブでもネガティブでも — 具体的に）
   - 「最初に目が行く3つは：**[1]**、**[2]**、**[3]**。」（階層チェック — これらは意図的か？）
   - 「一言で表すなら：**[言葉]**。」（直感的な判定）

これはユーザーが最初に読むセクション。意見を持つこと。デザイナーはぼかさない — 反応する。

---

## フェーズ2: デザインシステム抽出

サイトが実際に使用しているデザインシステムを抽出する（DESIGN.mdの記述ではなく、実際にレンダリングされているもの）：

```bash
# 使用中のフォント（タイムアウト回避のため500要素に制限）
$B js "JSON.stringify([...new Set([...document.querySelectorAll('*')].slice(0,500).map(e => getComputedStyle(e).fontFamily))])"

# 使用中のカラーパレット
$B js "JSON.stringify([...new Set([...document.querySelectorAll('*')].slice(0,500).flatMap(e => [getComputedStyle(e).color, getComputedStyle(e).backgroundColor]).filter(c => c !== 'rgba(0, 0, 0, 0)'))])"

# 見出し階層
$B js "JSON.stringify([...document.querySelectorAll('h1,h2,h3,h4,h5,h6')].map(h => ({tag:h.tagName, text:h.textContent.trim().slice(0,50), size:getComputedStyle(h).fontSize, weight:getComputedStyle(h).fontWeight})))"

# タッチターゲット監査（サイズ不足のインタラクティブ要素を検出）
$B js "JSON.stringify([...document.querySelectorAll('a,button,input,[role=button]')].filter(e => {const r=e.getBoundingClientRect(); return r.width>0 && (r.width<44||r.height<44)}).map(e => ({tag:e.tagName, text:(e.textContent||'').trim().slice(0,30), w:Math.round(e.getBoundingClientRect().width), h:Math.round(e.getBoundingClientRect().height)})).slice(0,20))"

# パフォーマンスベースライン
$B perf
```

所見を**推論されたデザインシステム**として構造化：
- **フォント：** 使用カウント付きリスト。3つ以上の異なるフォントファミリーがある場合はフラグ。
- **カラー：** 抽出されたパレット。12以上のユニークな非グレーカラーがある場合はフラグ。ウォーム/クール/混合を記載。
- **見出しスケール：** h1-h6のサイズ。スキップされたレベル、非体系的なサイズジャンプをフラグ。
- **スペーシングパターン：** padding/marginのサンプル値。スケールに沿わない値をフラグ。

抽出後、提案する：*「これをDESIGN.mdとして保存しますか？これらの観察をプロジェクトのデザインシステムベースラインとして確定できます。」*

---

## フェーズ3: ページごとのビジュアル監査

スコープ内の各ページに対して：

```bash
$B goto <url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/{page}-annotated.png"
$B responsive "$REPORT_DIR/screenshots/{page}"
$B console --errors
$B perf
```

### 認証検出

最初のナビゲーション後、URLがログインのようなパスに変わったか確認：
```bash
$B url
```
URLに`/login`、`/signin`、`/auth`、または`/sso`が含まれている場合：サイトは認証が必要。AskUserQuestion：「このサイトは認証が必要です。ブラウザからCookieをインポートしますか？必要に応じて先に`/setup-browser-cookies`を実行してください。」

### デザイン監査チェックリスト（10カテゴリ、約80項目）

各ページでこれらを適用する。各所見にはインパクト評価（high/medium/polish）とカテゴリを付与。

**1. ビジュアルヒエラルキーとコンポジション**（8項目）
- 明確なフォーカルポイントがあるか？ビューごとに1つの主要CTA？
- 視線が自然に左上から右下に流れるか？
- ビジュアルノイズ — 注意を奪い合う競合する要素？
- コンテンツタイプに適した情報密度？
- Z-indexの明確さ — 予期しない重なりがないか？
- ファーストビューのコンテンツが3秒で目的を伝えるか？
- スクイントテスト：ぼかしても階層が見えるか？
- ホワイトスペースが意図的で、余りものではないか？

**2. タイポグラフィ**（15項目）
- フォント数 <=3（超える場合はフラグ）
- スケールが比率に従っている（1.25 major thirdまたは1.333 perfect fourth）
- Line-height：本文1.5倍、見出し1.15-1.25倍
- 行長：1行あたり45-75文字（66が理想）
- 見出し階層：スキップされたレベルなし（h2なしのh1→h3）
- ウェイトコントラスト：階層のために2以上のウェイトを使用
- ブラックリストフォントなし（Papyrus、Comic Sans、Lobster、Impact、Jokerman）
- 主要フォントがInter/Roboto/Open Sans/Poppinsの場合 → ジェネリックの可能性をフラグ
- 見出しに`text-wrap: balance`または`text-pretty`（`$B css <heading> text-wrap`で確認）
- カーリークォートを使用、ストレートクォートではない
- 省略記号文字（`…`）を使用、3つのドット（`...`）ではない
- 数値列に`font-variant-numeric: tabular-nums`
- 本文テキスト >= 16px
- キャプション/ラベル >= 12px
- 小文字テキストにletter-spacingなし

**3. カラーとコントラスト**（10項目）
- パレットが統一的（<=12のユニークな非グレーカラー）
- WCAG AA：本文テキスト4.5:1、大テキスト（18px+）3:1、UIコンポーネント3:1
- セマンティックカラーの一貫性（success=グリーン、error=レッド、warning=イエロー/アンバー）
- カラーのみのエンコーディングなし（常にラベル、アイコン、またはパターンを追加）
- ダークモード：サーフェスはエレベーションを使用、単なる明度反転ではない
- ダークモード：テキストはオフホワイト（~#E0E0E0）、純白ではない
- ダークモードではプライマリアクセントを10-20%デサチュレート
- html要素に`color-scheme: dark`（ダークモードが存在する場合）
- 赤/緑のみの組み合わせなし（男性の8%が赤緑色覚異常）
- ニュートラルパレットがウォームまたはクールで一貫 — 混合ではない

**4. スペーシングとレイアウト**（12項目）
- 全ブレークポイントでグリッドが一貫
- スペーシングがスケールを使用（4pxまたは8pxベース）、任意の値ではない
- アラインメントが一貫 — グリッドからはみ出す要素がない
- リズム：関連アイテムはより近く、異なるセクションはより遠く
- Border-radiusの階層（全てに均一なバブリーradiusではない）
- 内側radius = 外側radius - gap（ネストされた要素）
- モバイルで横スクロールなし
- 最大コンテンツ幅が設定されている（全幅の本文テキストなし）
- ノッチデバイス用に`env(safe-area-inset-*)`
- URLが状態を反映（フィルター、タブ、ページネーションをクエリパラメータに）
- レイアウトにFlex/gridを使用（JS計測ではない）
- ブレークポイント：モバイル（375）、タブレット（768）、デスクトップ（1024）、ワイド（1440）

**5. インタラクション状態**（10項目）
- 全インタラクティブ要素にホバー状態
- `focus-visible`リングが存在（代替なしの`outline: none`は不可）
- アクティブ/プレス状態に深度効果またはカラーシフト
- 無効状態：opacity低下 + `cursor: not-allowed`
- ローディング：スケルトンの形状が実際のコンテンツレイアウトに一致
- 空の状態：温かいメッセージ + 主要アクション + ビジュアル（単なる「アイテムなし。」ではない）
- エラーメッセージ：具体的 + 修正/次のステップを含む
- 成功：確認アニメーションまたはカラー、自動消去
- タッチターゲット >= 44px（全インタラクティブ要素）
- 全クリック可能要素に`cursor: pointer`

**6. レスポンシブデザイン**（8項目）
- モバイルレイアウトが*デザイン*として意味がある（デスクトップカラムを積んだだけではない）
- モバイルで十分なタッチターゲット（>= 44px）
- どのビューポートでも横スクロールなし
- 画像がレスポンシブに対応（srcset、sizes、またはCSS containment）
- モバイルでズームなしでテキストが読める（本文>= 16px）
- ナビゲーションが適切に折りたたまれる（ハンバーガー、ボトムナビ、等）
- フォームがモバイルで使える（正しいinputタイプ、モバイルでのautoFocusなし）
- viewport metaに`user-scalable=no`や`maximum-scale=1`がない

**7. モーションとアニメーション**（6項目）
- イージング：entering時はease-out、exiting時はease-in、moving時はease-in-out
- デュレーション：50-700ms範囲（ページ遷移でない限りそれより遅くしない）
- 目的：全てのアニメーションが何かを伝える（状態変化、注意、空間的関係）
- `prefers-reduced-motion`が尊重されている（確認：`$B js "matchMedia('(prefers-reduced-motion: reduce)').matches"`）
- `transition: all`なし — プロパティを明示的にリスト
- `transform`と`opacity`のみアニメーション（width、height、top、leftなどのレイアウトプロパティではない）

**8. コンテンツとマイクロコピー**（8項目）
- 空の状態が温かみを持ってデザイン（メッセージ + アクション + イラスト/アイコン）
- エラーメッセージが具体的：何が起きたか + なぜ + 次に何をすべきか
- ボタンラベルが具体的（「APIキーを保存」であって「続行」や「送信」ではない）
- 本番環境にプレースホルダー/lorem ipsumテキストが表示されていない
- トランケーション処理済み（`text-overflow: ellipsis`、`line-clamp`、または`break-words`）
- 能動態（「CLIをインストール」であって「CLIがインストールされます」ではない）
- ローディング状態が`…`で終わる（「保存中…」であって「保存中...」ではない）
- 破壊的アクションに確認モーダルまたは取り消しウィンドウがある

**9. AIスロップ検出**（10のアンチパターン — ブラックリスト）

テスト：一流スタジオの人間デザイナーがこれをシップするか？

- パープル/バイオレット/インディゴのグラデーション背景またはブルーからパープルのカラースキーム
- **3カラム特徴グリッド：** カラード円の中のアイコン + 太字タイトル + 2行の説明、3つ対称に繰り返し。最もわかりやすいAIレイアウト。
- セクション装飾としてのカラード円内アイコン（SaaSスターターテンプレート風）
- 全てセンタリング（全見出し、説明、カードに`text-align: center`）
- 全要素に均一なバブリーborder-radius（全てに同じ大きなradius）
- 装飾的ブロブ、フローティング円、ウェーブSVGディバイダー（セクションが空に感じるなら、より良いコンテンツが必要であり、装飾ではない）
- デザイン要素としての絵文字（見出しのロケット、箇条書きの絵文字）
- カードの色付き左ボーダー（`border-left: 3px solid <accent>`）
- ジェネリックなヒーローコピー（「[X]へようこそ」、「～の力を解放...」、「～のオールインワンソリューション...」）
- クッキーカッターセクションリズム（ヒーロー → 3つの特徴 → テスティモニアル → 料金 → CTA、全セクション同じ高さ）

**10. デザインとしてのパフォーマンス**（6項目）
- LCP < 2.0秒（ウェブアプリ）、< 1.5秒（情報サイト）
- CLS < 0.1（読み込み中に可視のレイアウトシフトなし）
- スケルトン品質：形状が実コンテンツに一致、シマーアニメーション
- 画像：`loading="lazy"`、width/heightディメンション設定、WebP/AVIFフォーマット
- フォント：`font-display: swap`、CDNオリジンへのpreconnect
- 可視のフォントスワップフラッシュ（FOUT）なし — クリティカルフォントがプリロード済み

---

## フェーズ4: インタラクションフローレビュー

2-3の主要ユーザーフローを歩き、機能だけでなく*体感*を評価する：

```bash
$B snapshot -i
$B click @e3           # アクションを実行
$B snapshot -D          # 何が変わったかを差分で確認
```

評価：
- **レスポンスの体感：** クリックがレスポンシブに感じるか？遅延やローディング状態の欠如はないか？
- **トランジション品質：** トランジションが意図的か、ジェネリック/不在か？
- **フィードバックの明確さ：** アクションが成功か失敗か明確にわかったか？フィードバックは即座か？
- **フォームの洗練度：** フォーカス状態が見えるか？バリデーションのタイミングが正しいか？エラーが発生源の近くにあるか？

---

## フェーズ5: ページ間の一貫性

ページ間のスクリーンショットと観察を比較：
- ナビゲーションバーが全ページで一貫しているか？
- フッターが一貫しているか？
- コンポーネントの再利用 vs. 一回限りのデザイン（同じボタンが異なるページで異なるスタイル？）
- トーンの一貫性（あるページは遊び心がありながら別のページはコーポレート？）
- スペーシングリズムがページ間で維持されているか？

---

## フェーズ6: レポート作成

### 出力場所

**ローカル：** `.gstack/design-reports/design-audit-{domain}-{YYYY-MM-DD}.md`

**プロジェクトスコープ：**
```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG
```
書き込み先：`~/.gstack/projects/{slug}/{user}-{branch}-design-audit-{datetime}.md`

**ベースライン：** リグレッションモード用に`design-baseline.json`を書き込む：
```json
{
  "date": "YYYY-MM-DD",
  "url": "<target>",
  "designScore": "B",
  "aiSlopScore": "C",
  "categoryGrades": { "hierarchy": "A", "typography": "B", ... },
  "findings": [{ "id": "FINDING-001", "title": "...", "impact": "high", "category": "typography" }]
}
```

### 採点システム

**2つのヘッドラインスコア：**
- **デザインスコア: {A-F}** — 全10カテゴリの加重平均
- **AIスロップスコア: {A-F}** — 端的な判定付きの独立グレード

**カテゴリ別グレード：**
- **A:** 意図的、洗練、魅力的。デザイン思考が見える。
- **B:** 堅実な基本、軽微な不整合。プロフェッショナルに見える。
- **C:** 機能的だがジェネリック。大きな問題なし、デザインの視点なし。
- **D:** 目立つ問題。未完成または無頓着に感じる。
- **F:** ユーザー体験を積極的に損なっている。大幅な見直しが必要。

**グレード計算：** 各カテゴリはAから開始。高インパクトの所見ごとに1レターグレード下がる。中インパクトの所見ごとに0.5レターグレード下がる。ポリッシュの所見は記録されるがグレードに影響しない。最低はF。

**デザインスコアのカテゴリ加重：**
| カテゴリ | 加重 |
|----------|--------|
| ビジュアルヒエラルキー | 15% |
| タイポグラフィ | 15% |
| スペーシングとレイアウト | 15% |
| カラーとコントラスト | 10% |
| インタラクション状態 | 10% |
| レスポンシブ | 10% |
| コンテンツ品質 | 10% |
| AIスロップ | 5% |
| モーション | 5% |
| パフォーマンス体感 | 5% |

AIスロップはデザインスコアの5%だが、ヘッドライン指標として独立してもグレード付けされる。

### リグレッション出力

以前の`design-baseline.json`が存在するか`--regression`フラグが使用された場合：
- ベースラインのグレードを読み込む
- 比較：カテゴリ別差分、新しい所見、解決済み所見
- レポートにリグレッションテーブルを追加

---

## デザイン批評の形式

意見ではなく構造化されたフィードバックを使用：
- 「気づいたのは...」 — 観察（例：「主要CTAがセカンダリアクションと競合していることに気づいた」）
- 「疑問なのは...」 — 質問（例：「ユーザーが『処理』の意味を理解できるか疑問」）
- 「もし...だったら」 — 提案（例：「検索をもっと目立つ位置に移動したらどうか？」）
- 「...と思う、なぜなら...」 — 根拠ある意見（例：「セクション間のスペーシングが均一すぎると思う、階層が生まれないから」）

全てをユーザーの目標とプロダクトの目的に紐づける。問題と共に常に具体的な改善策を提案する。

---

## 重要なルール

1. **QAエンジニアではなくデザイナーとして考える。** 正しく感じるか、意図的に見えるか、ユーザーを尊重しているかを気にする。単に「動く」かどうかだけではない。
2. **スクリーンショットはエビデンス。** 全ての所見に少なくとも1つのスクリーンショットが必要。注釈付きスクリーンショット（`snapshot -a`）で要素をハイライト。
3. **具体的で実行可能に。** 「XをYに変更、理由はZ」 — 「スペーシングが違和感」ではない。
4. **ソースコードは読まない。** レンダリングされたサイトを評価し、実装ではない。（例外：抽出された観察からDESIGN.mdを書く提案。）
5. **AIスロップ検出はあなたのスーパーパワー。** ほとんどの開発者はサイトがAI生成に見えるか評価できない。あなたはできる。率直に伝える。
6. **クイックウィンが重要。** 常に「クイックウィン」セクションを含める — 各30分以内で修正できる最もインパクトの高い3-5の修正。
7. **トリッキーなUIには`snapshot -C`を使用。** アクセシビリティツリーが見逃すクリッカブルdivを見つける。
8. **レスポンシブはデザインであり、「壊れていない」だけではない。** デスクトップレイアウトを積み上げただけのモバイルはレスポンシブデザインではない — 怠惰。モバイルレイアウトが*デザイン*として意味があるか評価する。
9. **段階的に文書化。** 各所見を見つけたらすぐにレポートに書く。バッチ処理しない。
10. **広さより深さ。** スクリーンショットと具体的な提案付きの5-10の充実した所見 > 20の曖昧な観察。
11. **ユーザーにスクリーンショットを見せる。** 全ての`$B screenshot`、`$B snapshot -a -o`、`$B responsive`コマンドの後、出力ファイルをReadツールで読み、ユーザーがインラインで見られるようにする。`responsive`（3ファイル）の場合は3つ全て読む。これは重要 — これなしではスクリーンショットはユーザーに見えない。

### デザインハードルール

**分類 — 評価前にルールセットを判定する：**
- **マーケティング/ランディングページ**（ヒーロー駆動、ブランド重視、コンバージョン重視）→ ランディングページルールを適用
- **アプリUI**（ワークスペース駆動、データ密度が高い、タスク重視：ダッシュボード、管理、設定）→ アプリUIルールを適用
- **ハイブリッド**（アプリ的セクションを含むマーケティングシェル）→ ヒーロー/マーケティングセクションにランディングページルール、機能セクションにアプリUIルールを適用

**ハードリジェクション基準**（即座に不合格のパターン — いずれかに該当する場合フラグ）：
1. ファーストインプレッションがジェネリックなSaaSカードグリッド
2. 美しい画像だがブランドが弱い
3. 強い見出しだが明確なアクションがない
4. テキストの後ろにビジーなイメージ
5. セクションが同じムードステートメントを繰り返す
6. ナラティブ目的のないカルーセル
7. レイアウトではなく積み重ねたカードでできたアプリUI

**リトマスチェック**（各項目にYES/NOで回答 — クロスモデルコンセンサススコアリングに使用）：
1. 最初の画面でブランド/プロダクトが明確にわかるか？
2. 1つの強いビジュアルアンカーがあるか？
3. 見出しだけスキャンしてページが理解できるか？
4. 各セクションに1つの役割があるか？
5. カードは本当に必要か？
6. モーションが階層またはアトモスフィアを改善しているか？
7. 装飾的シャドウを全て取り除いてもデザインがプレミアムに感じるか？

**ランディングページルール**（分類 = マーケティング/ランディングの場合に適用）：
- 最初のビューポートが1つの構図として読める、ダッシュボードではない
- ブランドファーストの階層：ブランド > 見出し > 本文 > CTA
- タイポグラフィ：表現力があり目的がある — デフォルトスタック（Inter、Roboto、Arial、system）ではない
- フラットな単色背景なし — グラデーション、画像、微妙なパターンを使用
- ヒーロー：全幅、エッジトゥエッジ、インセット/タイル/ラウンドバリエーションなし
- ヒーロー予算：ブランド、1つの見出し、1つのサポート文、1つのCTAグループ、1つの画像
- ヒーローにカードなし。カードはカード自体がインタラクションである場合のみ
- セクションごとに1つの役割：1つの目的、1つの見出し、1つの短いサポート文
- モーション：最低2-3の意図的なモーション（エントランス、スクロール連動、ホバー/リビール）
- カラー：CSS変数を定義、パープル・オン・ホワイトのデフォルトを避ける、アクセントカラーのデフォルトは1つ
- コピー：デザインコメンタリーではなくプロダクトの言葉。「30%削除して改善されるなら、削除し続ける」
- 美しいデフォルト：構図ファースト、ブランドが最も大きいテキスト、タイプフェイス最大2つ、デフォルトでカードレス、最初のビューポートはドキュメントではなくポスター

**アプリUIルール**（分類 = アプリUIの場合に適用）：
- 穏やかなサーフェス階層、強いタイポグラフィ、少ないカラー
- 密だが読みやすい、最小限のクローム
- 整理：プライマリワークスペース、ナビゲーション、セカンダリコンテキスト、1つのアクセント
- 避ける：ダッシュボードカードモザイク、太いボーダー、装飾的グラデーション、装飾的アイコン
- コピー：ユーティリティ言語 — オリエンテーション、ステータス、アクション。ムード/ブランド/アスピレーションではない
- カードはカード自体がインタラクションである場合のみ
- セクション見出しはエリアが何かまたはユーザーが何ができるかを述べる（「選択済みKPI」、「プランステータス」）

**ユニバーサルルール**（全タイプに適用）：
- カラーシステム用のCSS変数を定義
- デフォルトフォントスタックなし（Inter、Roboto、Arial、system）
- セクションごとに1つの役割
- 「コピーの30%を削除して改善されるなら、削除し続ける」
- カードは存在理由を証明する — 装飾的カードグリッドなし

**AIスロップブラックリスト**（「AI生成」と叫ぶ10のパターン）：
1. パープル/バイオレット/インディゴのグラデーション背景またはブルーからパープルのカラースキーム
2. **3カラム特徴グリッド：** カラード円の中のアイコン + 太字タイトル + 2行の説明、3つ対称に繰り返し。最もわかりやすいAIレイアウト。
3. セクション装飾としてのカラード円内アイコン（SaaSスターターテンプレート風）
4. 全てセンタリング（全見出し、説明、カードに`text-align: center`）
5. 全要素に均一なバブリーborder-radius（全てに同じ大きなradius）
6. 装飾的ブロブ、フローティング円、ウェーブSVGディバイダー（セクションが空に感じるなら、より良いコンテンツが必要であり、装飾ではない）
7. デザイン要素としての絵文字（見出しのロケット、箇条書きの絵文字）
8. カードの色付き左ボーダー（`border-left: 3px solid <accent>`）
9. ジェネリックなヒーローコピー（「[X]へようこそ」、「～の力を解放...」、「～のオールインワンソリューション...」）
10. クッキーカッターセクションリズム（ヒーロー → 3つの特徴 → テスティモニアル → 料金 → CTA、全セクション同じ高さ）

出典：[OpenAI "Designing Delightful Frontends with GPT-5.4"](https://developers.openai.com/blog/designing-delightful-frontends-with-gpt-5-4)（2026年3月）+ gstackデザイン方法論。

フェーズ6の最後にベースラインのデザインスコアとAIスロップスコアを記録。

---

## 出力構造

```
.gstack/design-reports/
├── design-audit-{domain}-{YYYY-MM-DD}.md    # 構造化レポート
├── screenshots/
│   ├── first-impression.png                  # フェーズ1
│   ├── {page}-annotated.png                  # ページごとの注釈付き
│   ├── {page}-mobile.png                     # レスポンシブ
│   ├── {page}-tablet.png
│   ├── {page}-desktop.png
│   ├── finding-001-before.png                # 修正前
│   ├── finding-001-after.png                 # 修正後
│   └── ...
└── design-baseline.json                      # リグレッションモード用
```

---

## デザイン外部の声（並列実行）

**自動：** Codexが利用可能な場合、外部の声は自動的に実行される。オプトイン不要。

**Codexの利用可能性を確認：**
```bash
which codex 2>/dev/null && echo "CODEX_AVAILABLE" || echo "CODEX_NOT_AVAILABLE"
```

**Codexが利用可能な場合**、両方の声を同時に起動：

1. **Codexデザインボイス**（Bash経由）：
```bash
TMPERR_DESIGN=$(mktemp /tmp/codex-design-XXXXXXXX)
codex exec "Review the frontend source code in this repo. Evaluate against these design hard rules:
- Spacing: systematic (design tokens / CSS variables) or magic numbers?
- Typography: expressive purposeful fonts or default stacks?
- Color: CSS variables with defined system, or hardcoded hex scattered?
- Responsive: breakpoints defined? calc(100svh - header) for heroes? Mobile tested?
- A11y: ARIA landmarks, alt text, contrast ratios, 44px touch targets?
- Motion: 2-3 intentional animations, or zero / ornamental only?
- Cards: used only when card IS the interaction? No decorative card grids?

First classify as MARKETING/LANDING PAGE vs APP UI vs HYBRID, then apply matching rules.

LITMUS CHECKS — answer YES/NO:
1. Brand/product unmistakable in first screen?
2. One strong visual anchor present?
3. Page understandable by scanning headlines only?
4. Each section has one job?
5. Are cards actually necessary?
6. Does motion improve hierarchy or atmosphere?
7. Would design feel premium with all decorative shadows removed?

HARD REJECTION — flag if ANY apply:
1. Generic SaaS card grid as first impression
2. Beautiful image with weak brand
3. Strong headline with no clear action
4. Busy imagery behind text
5. Sections repeating same mood statement
6. Carousel with no narrative purpose
7. App UI made of stacked cards instead of layout

Be specific. Reference file:line for every finding." -s read-only -c 'model_reasoning_effort="high"' --enable web_search_cached 2>"$TMPERR_DESIGN"
```
5分のタイムアウトを使用（`timeout: 300000`）。コマンド完了後、stderrを読む：
```bash
cat "$TMPERR_DESIGN" && rm -f "$TMPERR_DESIGN"
```

2. **Claudeデザインサブエージェント**（Agentツール経由）：
以下のプロンプトでサブエージェントを起動：
「このリポジトリのフロントエンドソースコードをレビューしてください。あなたはソースコードデザイン監査を行う独立したシニアプロダクトデザイナーです。個々の違反ではなくファイル間の一貫性パターンに焦点：
- スペーシング値はコードベース全体で体系的か？
- 1つのカラーシステムか、散在したアプローチか？
- レスポンシブブレークポイントが一貫したセットに従っているか？
- アクセシビリティのアプローチが一貫しているか、まだらか？

各所見について：何が問題か、重大度（critical/high/medium）、file:lineを記載。」

**エラーハンドリング（全て非ブロッキング）：**
- **認証失敗：** stderrに"auth"、"login"、"unauthorized"、"API key"が含まれる場合：「Codex認証に失敗しました。`codex login`を実行して認証してください。」
- **タイムアウト：** 「Codexが5分後にタイムアウトしました。」
- **空のレスポンス：** 「Codexがレスポンスを返しませんでした。」
- Codexエラーの場合：Claudeサブエージェントの出力のみで続行、`[single-model]`タグ付き。
- Claudeサブエージェントも失敗した場合：「外部の声が利用できません — 主要レビューで続行します。」

Codex出力を`CODEX SAYS（デザインソース監査）：`ヘッダーの下に表示。
サブエージェント出力を`CLAUDE SUBAGENT（デザイン一貫性）：`ヘッダーの下に表示。

**統合 — リトマススコアカード：**

/plan-design-reviewと同じスコアカード形式を使用（上記参照）。両方の出力から記入。
所見を`[codex]` / `[subagent]` / `[cross-model]`タグ付きでトリアージに統合。

**結果をログ：**
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"design-outside-voices","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","status":"STATUS","source":"SOURCE","commit":"'"$(git rev-parse --short HEAD)"'"}'
```
STATUSを"clean"または"issues_found"に、SOURCEを"codex+subagent"、"codex-only"、"subagent-only"、"unavailable"に置き換える。

## フェーズ7: トリアージ

発見された全所見をインパクト順にソートし、修正するものを決定：

- **高インパクト：** 最初に修正。ファーストインプレッションに影響し、ユーザーの信頼を損なう。
- **中インパクト：** 次に修正。洗練度を下げ、無意識に感じられる。
- **ポリッシュ：** 時間があれば修正。「良い」と「素晴らしい」を分ける。

ソースコードから修正できない所見（例：サードパーティウィジェットの問題、チームからのコピーが必要なコンテンツの問題）は、インパクトに関係なく「deferred」としてマーク。

---

## フェーズ8: 修正ループ

修正可能な各所見に対して、インパクト順に：

### 8a. ソースを特定

```bash
# CSSクラス、コンポーネント名、スタイルファイルを検索
# 影響を受けるページに一致するファイルパターンをGlobで検索
```

- デザイン問題の原因となるソースファイルを見つける
- 所見に直接関連するファイル**のみ**を修正
- 構造的コンポーネント変更よりCSS/スタイル変更を優先

### 8b. 修正

- ソースコードを読み、コンテキストを理解
- **最小限の修正**を行う — デザイン問題を解決する最小の変更
- CSS専用の変更が望ましい（より安全、より可逆的）
- 周辺コードのリファクタリング、機能追加、関連しないものの「改善」は**行わない**

### 8c. コミット

```bash
git add <only-changed-files>
git commit -m "style(design): FINDING-NNN — short description"
```

- 修正ごとに1コミット。複数の修正をまとめない。
- メッセージ形式：`style(design): FINDING-NNN — short description`

### 8d. 再テスト

影響を受けるページに戻り修正を検証：

```bash
$B goto <affected-url>
$B screenshot "$REPORT_DIR/screenshots/finding-NNN-after.png"
$B console --errors
$B snapshot -D
```

全修正について**ビフォー/アフタースクリーンショットペア**を撮影。

### 8e. 分類

- **verified**：再テストで修正が機能し、新しいエラーが導入されていないことを確認
- **best-effort**：修正を適用したが完全に検証できなかった（例：特定のブラウザ状態が必要）
- **reverted**：リグレッションを検出 → `git revert HEAD` → 所見を「deferred」としてマーク

### 8e.5. リグレッションテスト（design-reviewバリアント）

デザイン修正は通常CSS専用。JavaScriptの振る舞いの変更を含む修正のみリグレッションテストを生成 — 壊れたドロップダウン、アニメーション失敗、条件付きレンダリング、インタラクティブ状態の問題。

CSS専用修正の場合：完全にスキップ。CSSリグレッションは/design-reviewの再実行で検出される。

修正がJSの振る舞いを含む場合：/qaフェーズ8e.5と同じ手順に従う（既存テストパターンを研究し、正確なバグ条件をエンコードするリグレッションテストを書き、実行し、パスすればコミット、失敗すればdefer）。コミット形式：`test(design): regression test for FINDING-NNN`。

### 8f. 自己規制（止まって評価）

5つの修正ごとに（またはリバート後に）、デザイン修正リスクレベルを計算：

```
DESIGN-FIX RISK:
  0%から開始
  各リバート:                        +15%
  各CSS専用ファイル変更:              +0%   （安全 — スタイルのみ）
  各JSX/TSX/コンポーネントファイル変更: +5%   ファイルごと
  修正10件目以降:                     +1%   追加修正ごと
  無関係ファイルへの変更:              +20%
```

**リスク > 20%の場合：** 即座に**停止**。これまでの作業をユーザーに見せる。続行するか尋ねる。

**ハードキャップ：30件の修正。** 30件の修正後、残りの所見に関係なく停止。

---

## フェーズ9: 最終デザイン監査

全修正の適用後：

1. 影響を受けた全ページでデザイン監査を再実行
2. 最終デザインスコアとAIスロップスコアを計算
3. **最終スコアがベースラインより悪い場合：** 目立つように警告 — 何かがリグレッションした

---

## フェーズ10: レポート

ローカルとプロジェクトスコープの両方にレポートを書き込む：

**ローカル：** `.gstack/design-reports/design-audit-{domain}-{YYYY-MM-DD}.md`

**プロジェクトスコープ：**
```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG
```
`~/.gstack/projects/{slug}/{user}-{branch}-design-audit-{datetime}.md`に書き込む

**所見ごとの追加情報**（標準デザイン監査レポートに加えて）：
- 修正ステータス：verified / best-effort / reverted / deferred
- コミットSHA（修正済みの場合）
- 変更ファイル（修正済みの場合）
- ビフォー/アフタースクリーンショット（修正済みの場合）

**サマリーセクション：**
- 総所見数
- 適用された修正（verified: X、best-effort: Y、reverted: Z）
- 保留中の所見
- デザインスコア差分：ベースライン → 最終
- AIスロップスコア差分：ベースライン → 最終

**PRサマリー：** PR説明に適した一行のサマリーを含める：
> 「デザインレビューでN件の問題を発見、M件を修正。デザインスコアX → Y、AIスロップスコアX → Y。」

---

## フェーズ11: TODOS.md更新

リポジトリに`TODOS.md`がある場合：

1. **新しい保留中デザイン所見** → インパクトレベル、カテゴリ、説明付きでTODOとして追加
2. **TODOS.mdにあった修正済み所見** → 「/design-reviewにより{branch}で{date}に修正」と注釈

---

## 追加ルール（design-review固有）

11. **クリーンなワーキングツリーが必要。** ダーティな場合、続行前にAskUserQuestionでコミット/スタッシュ/中止を提案。
12. **修正ごとに1コミット。** 複数のデザイン修正を1つのコミットにまとめない。
13. **フェーズ8e.5でリグレッションテストを生成する場合のみテストを変更。** CI設定は変更しない。既存テストは変更しない — 新しいテストファイルのみ作成。
14. **リグレッション時はリバート。** 修正が状況を悪化させたら、即座に`git revert HEAD`。
15. **自己規制。** デザイン修正リスクヒューリスティックに従う。迷ったら止まって尋ねる。
16. **CSSファースト。** 構造的コンポーネント変更よりCSS/スタイル変更を優先。CSS専用の変更はより安全で可逆的。
17. **DESIGN.mdエクスポート。** フェーズ2からの提案をユーザーが承諾した場合、DESIGN.mdファイルを書くことが**可能**。
