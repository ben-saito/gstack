# gstack — AIによる仮想エンジニアリングチーム

> 「12月以降、おそらく1行もコードを手で書いていない。これは極めて大きな変化だ。」 — [Andrej Karpathy](https://fortune.com/2026/03/21/andrej-karpathy-openai-cofounder-ai-agents-coding-state-of-psychosis-openclaw/)、No Priorsポッドキャスト、2026年3月

Karpathyのこの発言を聞いたとき、その方法を知りたくなりました。一人の人間がどうやって20人のチームのようにシップできるのか？Peter Steinbergerは[OpenClaw](https://github.com/openclaw/openclaw)（GitHub 247Kスター）をAIエージェントを使ってほぼ一人で構築しました。革命はもう始まっています。適切なツールを持つ一人のビルダーが、従来のチームよりも速く動けるのです。

私は[Garry Tan](https://x.com/garrytan)、[Y Combinator](https://www.ycombinator.com/)の社長兼CEOです。Coinbase、Instacart、Ripplingなど、ガレージで1-2人だった頃の数千のスタートアップと仕事をしてきました。YCの前は、Palantirの最初のエンジニア/PM/デザイナーの一人であり、Posterous（Twitterに売却）を共同設立し、YCの内部SNSであるBookfaceを構築しました。

**gstackが私の答えです。** 20年間プロダクトを作ってきましたが、今はこれまでで最もコードをシップしています。過去60日間で：**60万行以上の本番コード**（35%がテスト）、**1日あたり10,000〜20,000行**、パートタイムで、YCのフルタイム業務と並行して。3つのプロジェクトにまたがる直近の`/retro`の結果：**140,751行の追加、362コミット、約115k行の純増** — たった1週間で。

**2026年 — 1,237コントリビューションとカウント中：**

![GitHub contributions 2026 — 1,237 contributions, massive acceleration in Jan-Mar](docs/images/github-2026.png)

**2013年 — YCでBookfaceを構築していた頃（772コントリビューション）：**

![GitHub contributions 2013 — 772 contributions building Bookface at YC](docs/images/github-2013.png)

同じ人間。違う時代。違いはツールです。

**gstackは私のやり方です。** Claude Codeを仮想エンジニアリングチームに変えます — プロダクトを再考するCEO、アーキテクチャを固めるエンジニアリングマネージャー、AIスロップを見抜くデザイナー、本番バグを見つけるレビュアー、実際のブラウザを開くQAリード、OWASP + STRIDEの監査を実行するセキュリティオフィサー、そしてPRをシップするリリースエンジニア。20人の専門家と8つのパワーツール、すべてスラッシュコマンド、すべてMarkdown、すべて無料、MITライセンス。

これは私のオープンソースソフトウェアファクトリーです。毎日使っています。これらのツールは誰もが使えるべきだと考え、公開しました。

フォークして、改善して、あなたのものにしてください。無料のオープンソースソフトウェアを批判したいなら — どうぞ。でもまずは使ってみてほしい。

**こんな人のために：**
- **創業者やCEO** — 特にまだシップしたい技術志向の人
- **初めてのClaude Codeユーザー** — 白紙のプロンプトではなく構造化された役割
- **テックリードやスタッフエンジニア** — すべてのPRに厳密なレビュー、QA、リリース自動化

## クイックスタート

1. gstackをインストール（30秒 — 下記参照）
2. `/office-hours`を実行 — 何を作りたいか説明する
3. `/plan-ceo-review`を任意の機能アイデアで実行
4. `/review`を変更のあるブランチで実行
5. `/qa`をステージングURLで実行
6. そこでストップ。自分に合うかどうかわかります。

## インストール — 30秒

**必要なもの：** [Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[Git](https://git-scm.com/)、[Bun](https://bun.sh/) v1.0+、[Node.js](https://nodejs.org/)（Windowsのみ）

### ステップ1：マシンにインストール

Claude Codeを開いて以下を貼り付けてください。Claudeが残りを行います。

> gstackをインストール：**`git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`** を実行し、CLAUDE.mdに「gstack」セクションを追加して、すべてのWebブラウジングにgstackの/browseスキルを使用し、mcp\_\_claude-in-chrome\_\_\*ツールは使用せず、利用可能なスキル一覧を記載：/office-hours、/plan-ceo-review、/plan-eng-review、/plan-design-review、/design-consultation、/review、/ship、/land-and-deploy、/canary、/benchmark、/browse、/qa、/qa-only、/design-review、/setup-browser-cookies、/setup-deploy、/retro、/investigate、/document-release、/codex、/cso、/autoplan、/careful、/freeze、/guard、/unfreeze、/gstack-upgrade。チームメイトにも追加するか確認してください。

### ステップ2：リポジトリに追加してチームメイトも使えるようにする（オプション）

> gstackをこのプロジェクトに追加：**`cp -Rf ~/.claude/skills/gstack .claude/skills/gstack && rm -rf .claude/skills/gstack/.git && cd .claude/skills/gstack && ./setup`** を実行し、プロジェクトのCLAUDE.mdにgstackセクションを追加。

実際のファイルがリポジトリにコミットされます（サブモジュールではありません）。`git clone`するだけで動きます。すべてが`.claude/`内に収まります。PATHを汚さず、バックグラウンドプロセスも走りません。

### Codex、Gemini CLI、またはCursor

gstackは[SKILL.md標準](https://github.com/anthropics/claude-code)をサポートする任意のエージェントで動作します。スキルは`.agents/skills/`に格納され、自動的に検出されます。

1つのリポジトリにインストール：

```bash
git clone https://github.com/garrytan/gstack.git .agents/skills/gstack
cd .agents/skills/gstack && ./setup --host codex
```

setupが`.agents/skills/gstack`から実行されると、生成されたCodexスキルを同じリポジトリ内に配置し、`~/.codex/skills`には書き込みません。

ユーザーアカウントに一度だけインストール：

```bash
git clone https://github.com/garrytan/gstack.git ~/gstack
cd ~/gstack && ./setup --host codex
```

`setup --host codex`はランタイムルートを`~/.codex/skills/gstack`に作成し、生成されたCodexスキルをトップレベルにリンクします。

または、setupにインストール済みのエージェントを自動検出させる：

```bash
git clone https://github.com/garrytan/gstack.git ~/gstack
cd ~/gstack && ./setup --host auto
```

Codex互換ホストの場合、setupはリポジトリローカルインストール（`.agents/skills/gstack`）とユーザーグローバルインストール（`~/.codex/skills/gstack`）の両方をサポートします。全28スキルがすべてのサポートエージェントで動作します。フックベースの安全スキル（careful、freeze、guard）は非Claudeホストではインライン安全アドバイザリー文を使用します。

## 動作例

```
あなた: カレンダーの日次ブリーフィングアプリを作りたい。
あなた: /office-hours
Claude: [痛みについて聞く — 仮説ではなく具体的な例]

あなた: 複数のGoogleカレンダー、古い情報のイベント、間違った場所。
        準備に時間がかかりすぎて、結果も十分じゃない...

Claude: フレーミングに異議を唱えます。あなたは「日次ブリーフィング
        アプリ」と言いました。でも実際に説明したのは、パーソナル
        チーフオブスタッフAIです。
        [あなたが気づいていなかった5つの能力を抽出]
        [4つの前提に挑戦 — 同意、反対、または調整]
        [工数見積もり付きの3つの実装アプローチを生成]
        推奨: 最小限のウェッジを明日シップし、実際の使用から学ぶ。
        フルビジョンは3ヶ月プロジェクト — まず実際に動く
        日次ブリーフィングから始めましょう。
        [設計書を作成 → 下流のスキルに自動的にフィード]

あなた: /plan-ceo-review
        [設計書を読み、スコープに挑戦、10セクションレビューを実行]

あなた: /plan-eng-review
        [データフロー、ステートマシン、エラーパスのASCII図]
        [テストマトリクス、失敗モード、セキュリティ懸念]

あなた: プランを承認。プランモードを終了。
        [11ファイルにまたがる2,400行を記述。約8分。]

あなた: /review
        [自動修正] 2件。[確認] 競合状態 → 修正を承認。

あなた: /qa https://staging.myapp.com
        [実際のブラウザを開き、フローをクリックして、バグを見つけて修正]

あなた: /ship
        テスト: 42 → 51 (+9件新規)。PR: github.com/you/app/pull/42
```

あなたは「日次ブリーフィングアプリ」と言いました。エージェントは「チーフオブスタッフAIを作っている」と言いました — あなたの機能要求ではなく、あなたの痛みに耳を傾けたからです。8つのコマンド、エンドツーエンド。これはコパイロットではありません。これはチームです。

## スプリント

gstackはツールの寄せ集めではなく、プロセスです。スキルはスプリントと同じ順序で実行されます：

**考える → 計画する → 作る → レビューする → テストする → シップする → 振り返る**

各スキルは次のスキルにフィードします。`/office-hours`が書いた設計書を`/plan-ceo-review`が読みます。`/plan-eng-review`が書いたテスト計画を`/qa`が拾います。`/review`が見つけたバグを`/ship`が修正を確認します。すべてのステップが前のステップを知っているので、何も漏れません。

| スキル | 専門家 | 役割 |
|-------|--------|------|
| `/office-hours` | **YCオフィスアワー** | ここから始めましょう。コードを書く前にプロダクトを再構築する6つの質問。フレーミングに異議を唱え、前提に挑戦し、実装の代替案を生成します。設計書はすべての下流スキルにフィードされます。 |
| `/plan-ceo-review` | **CEO / 創業者** | 問題を再考する。リクエストの中に隠れた10つ星プロダクトを見つける。4つのモード：拡大、選択的拡大、スコープ維持、縮小。 |
| `/plan-eng-review` | **エンジニアリングマネージャー** | アーキテクチャ、データフロー、ダイアグラム、エッジケース、テストを固める。隠れた前提を表に出す。 |
| `/plan-design-review` | **シニアデザイナー** | 各デザイン要素を0-10で評価し、10点が何かを説明し、プランを編集して改善する。AIスロップ検出。インタラクティブ — デザインの選択ごとにAskUserQuestion。 |
| `/design-consultation` | **デザインパートナー** | ゼロから完全なデザインシステムを構築。ランドスケープを調査し、クリエイティブなリスクを提案し、リアルなプロダクトモックアップを生成。 |
| `/review` | **スタッフエンジニア** | CIは通るがプロダクションで爆発するバグを見つける。明らかなものは自動修正。完全性のギャップを指摘。 |
| `/investigate` | **デバッガー** | 体系的な根本原因のデバッグ。鉄則：調査なしの修正は行わない。データフローを追跡し、仮説をテストし、3回失敗したら停止。 |
| `/design-review` | **コードも書けるデザイナー** | /plan-design-reviewと同じ監査を行い、見つけた問題を修正。アトミックコミット、ビフォーアフターのスクリーンショット。 |
| `/qa` | **QAリード** | アプリをテストし、バグを見つけ、アトミックコミットで修正し、再検証。すべての修正に対して回帰テストを自動生成。 |
| `/qa-only` | **QAレポーター** | /qaと同じ手法だがレポートのみ。コード変更なしの純粋なバグレポート。 |
| `/cso` | **最高セキュリティ責任者** | OWASP Top 10 + STRIDE脅威モデル。ノイズゼロ：17の偽陽性除外、8/10以上の確信度ゲート、独立した検出検証。各検出にはコンクリートなエクスプロイトシナリオを含む。 |
| `/ship` | **リリースエンジニア** | mainを同期、テスト実行、カバレッジ監査、プッシュ、PR作成。テストフレームワークがなければブートストラップ。 |
| `/land-and-deploy` | **リリースエンジニア** | PRをマージし、CIとデプロイを待ち、プロダクションの健全性を確認。「承認済み」から「プロダクションで確認済み」まで1コマンド。 |
| `/canary` | **SRE** | デプロイ後の監視ループ。コンソールエラー、パフォーマンスの低下、ページの失敗を監視。 |
| `/benchmark` | **パフォーマンスエンジニア** | ページロード時間、Core Web Vitals、リソースサイズのベースライン。すべてのPRでビフォーアフター比較。 |
| `/document-release` | **テクニカルライター** | シップした内容に合わせてすべてのプロジェクトドキュメントを更新。古いREADMEを自動検出。 |
| `/retro` | **エンジニアリングマネージャー** | チーム対応の週次振り返り。メンバー別内訳、シッピングストリーク、テスト健全性トレンド、成長機会。`/retro global`はすべてのプロジェクトとAIツール（Claude Code、Codex、Gemini）横断で実行。 |
| `/browse` | **QAエンジニア** | 実際のChromiumブラウザ、実際のクリック、実際のスクリーンショット。コマンドあたり約100ms。 |
| `/setup-browser-cookies` | **セッションマネージャー** | 実際のブラウザ（Chrome、Arc、Brave、Edge）からヘッドレスセッションにCookieをインポート。認証済みページをテスト。 |
| `/autoplan` | **レビューパイプライン** | 1コマンドで完全にレビューされたプラン。CEO → デザイン → エンジニアリングレビューを自動実行。テイスト判断のみを承認のために提示。 |

### パワーツール

| スキル | 機能 |
|-------|------|
| `/codex` | **セカンドオピニオン** — OpenAI Codex CLIからの独立コードレビュー。3つのモード：レビュー（合格/不合格ゲート）、敵対的挑戦、オープン相談。`/review`と`/codex`の両方が実行された場合のクロスモデル分析。 |
| `/careful` | **安全ガードレール** — 破壊的コマンド（rm -rf、DROP TABLE、force-push）の前に警告。「be careful」で有効化。任意の警告をオーバーライド可能。 |
| `/freeze` | **編集ロック** — ファイル編集を1つのディレクトリに制限。デバッグ中のスコープ外の偶発的変更を防止。 |
| `/guard` | **フル安全** — `/careful` + `/freeze`を1コマンドで。プロダクション作業での最大安全性。 |
| `/unfreeze` | **ロック解除** — `/freeze`の制限を解除。 |
| `/setup-deploy` | **デプロイ設定** — `/land-and-deploy`のワンタイムセットアップ。プラットフォーム、本番URL、デプロイコマンドを検出。 |
| `/gstack-upgrade` | **自己更新** — gstackを最新にアップグレード。グローバル vs ベンダーインストールを検出、両方を同期、変更点を表示。 |

**[全スキルの詳細解説と哲学 →](docs/skills.md)**

## 並列スプリント

gstackは1つのスプリントでもうまく機能します。10個同時に走らせると面白くなります。

[Conductor](https://conductor.build)は複数のClaude Codeセッションを並列実行します — それぞれが独立したワークスペースで。1つのセッションが`/office-hours`、別のセッションが`/review`、3つ目が機能の実装、4つ目が`/qa`を実行。すべて同時に。スプリント構造があるからこそ並列処理が機能します — プロセスがなければ10個のエージェントは10個のカオスの源です。プロセスがあれば、各エージェントが何をすべきか、いつ止めるべきかを正確に知っています。

---

無料、MITライセンス、オープンソース。プレミアムティアなし、ウェイトリストなし。

私はソフトウェアの作り方をオープンソースにしました。フォークして、あなた自身のものにしてください。

> **採用中です。** 1日10K+LOCをシップし、gstackの強化を手伝いませんか？
> YCで働こう — [ycombinator.com/software](https://ycombinator.com/software)
> 極めて競争力のある給与とエクイティ。サンフランシスコ、Dogpatch地区。

## ドキュメント

| ドキュメント | 内容 |
|-------------|------|
| [スキル詳細ガイド](docs/skills.md) | 全スキルの哲学、例、ワークフロー（Greptile統合を含む） |
| [ビルダーの哲学](ETHOS.md) | ビルダー哲学：Boil the Lake、Search Before Building、3層の知識 |
| [アーキテクチャ](ARCHITECTURE.md) | 設計判断とシステム内部構造 |
| [ブラウザリファレンス](BROWSER.md) | `/browse`の完全なコマンドリファレンス |
| [コントリビューティング](CONTRIBUTING.md) | 開発セットアップ、テスト、コントリビューターモード、開発モード |
| [チェンジログ](CHANGELOG.md) | すべてのバージョンの変更点 |

## プライバシーとテレメトリ

gstackには**オプトイン**の使用テレメトリが含まれており、プロジェクトの改善に役立てています。正確に何が起こるかを説明します：

- **デフォルトはオフです。** 明示的に同意しない限り、何も送信されません。
- **初回実行時に**、匿名の使用データを共有するかどうか尋ねます。拒否できます。
- **送信されるもの（オプトインした場合）：** スキル名、所要時間、成功/失敗、gstackバージョン、OS。以上です。
- **決して送信されないもの：** コード、ファイルパス、リポジトリ名、ブランチ名、プロンプト、その他ユーザー生成コンテンツ。
- **いつでも変更可能：** `gstack-config set telemetry off`で即座にすべて無効化できます。

データは[Supabase](https://supabase.com)（オープンソースのFirebase代替）に保存されます。スキーマは[`supabase/migrations/`](supabase/migrations/)にあります — 何が収集されているか正確に確認できます。リポジトリ内のSupabase公開キーはパブリックキー（FirebaseのAPIキーと同様）です — 行レベルのセキュリティポリシーがすべての直接アクセスを拒否します。テレメトリはスキーマチェック、イベントタイプ許可リスト、フィールド長制限を実施するバリデーション済みエッジファンクションを通じて流れます。

**ローカル分析は常に利用可能です。** `gstack-analytics`を実行すると、ローカルのJSONLファイルからパーソナルな使用状況ダッシュボードが表示されます — リモートデータは不要です。

## トラブルシューティング

**スキルが表示されない？** `cd ~/.claude/skills/gstack && ./setup`

**`/browse`が失敗する？** `cd ~/.claude/skills/gstack && bun install && bun run build`

**古いインストール？** `/gstack-upgrade`を実行 — または`~/.gstack/config.yaml`で`auto_upgrade: true`を設定

**Codexが「Skipped loading skill(s) due to invalid SKILL.md」と表示する？** Codexスキルの説明が古いです。修正：`cd ~/.codex/skills/gstack && git pull && ./setup --host codex` — またはリポジトリローカルインストールの場合：`cd "$(readlink -f .agents/skills/gstack)" && git pull && ./setup --host codex`

**Windowsユーザー：** gstackはGit BashまたはWSL経由のWindows 11で動作します。Bunに加えてNode.jsが必要です — BunにはPlaywrightのパイプトランスポートに関する既知のバグがあります（[bun#4253](https://github.com/oven-sh/bun/issues/4253)）。browseサーバーは自動的にNode.jsにフォールバックします。`bun`と`node`の両方がPATHにあることを確認してください。

**Claudeがスキルを認識できない？** プロジェクトの`CLAUDE.md`にgstackセクションがあることを確認してください。以下を追加してください：

```
## gstack
すべてのWebブラウジングにはgstackの/browseを使用。mcp__claude-in-chrome__*ツールは使用しない。
利用可能なスキル: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review,
/design-consultation, /review, /ship, /land-and-deploy, /canary, /benchmark, /browse,
/qa, /qa-only, /design-review, /setup-browser-cookies, /setup-deploy, /retro,
/investigate, /document-release, /codex, /cso, /autoplan, /careful, /freeze, /guard,
/unfreeze, /gstack-upgrade.
```

## ライセンス

MIT。永久無料。何か作ろう。
