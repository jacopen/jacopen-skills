---
name: quants
description: 株式オートトレードに役立つクオンツ研究情報の収集・選別・蓄積スキル。SSRN、arXiv q-fin、学術誌（JF/JFE/JPM）、ICAIF等のカンファレンス、AQR・Man Group等の運用会社リサーチから最新のディスカッションを収集し、有益と判断したもののみ OKF準拠の LLM Wiki バンドル /home/jacopen/vault/06_Wiki/03_Quants/ に蓄積する。トリガー例:「クオンツ情報を集めて」「最新の論文をチェックして」「quants sweep」「momentum の研究を探して」「ファクター投資の新しい議論は?」「クオンツwikiで調べて」など、クオンツ・システマティックトレーディング関連の情報収集・調査依頼。
---

# quants — クオンツ研究の収集・選別・蓄積スキル

株式のオートトレード（システマティックトレーディング）に役立つクオンツ研究の最新情報を収集し、**内容を評価した上で、有益と判断したもののみ** OKF準拠の LLM Wiki バンドルに蓄積する。

**あなた（LLM）がキュレーターである。** 網羅的なクリッピングではなく、「トレーディングシステムの設計・改善に影響を与えうるか」という基準での選別が仕事の本体。玉石混交のワーキングペーパーから玉だけを拾う。拾った玉は、後から検索・横断参照できる知識ベースとして構造化して残す。

このスキルは [[llm-wiki]] スキルと同じ **Open Knowledge Format (OKF v0.1)** に準拠したバンドルを、クオンツ専用テーマ（`03_Quants`）として運用する。OKF の一般規約は llm-wiki スキルに従い、本スキルはそこにクオンツ特有の「厳格な選別」と「評価台帳」を上乗せする。

## 保存先レイアウト — OKF バンドル

ルート: `/home/jacopen/vault/06_Wiki/03_Quants/`（以下 `QUANTS_HOME`）。他テーマ（`01_PlatformEngineering` 等）と同じ構造。

```
03_Quants/                    # OKF バンドルルート
├── AGENTS.md                 # テーマスキーマ (type: Schema)
├── index.md                  # バンドル目録（予約ファイル・frontmatterなし）
├── log.md                    # 活動ログ（予約ファイル・frontmatterなし）
├── EVALUATED.md              # 評価台帳（本スキル固有・予約扱い・frontmatterなし）
├── raw/                      # 論文の本文（不変・編集禁止）
│   ├── index.md
│   └── <slug>.md             # type: RawSource — 本文フルテキスト
├── summaries/                # 論文ごとのキュレーションノート（旧 PaperNote）
│   ├── index.md
│   └── <slug>.md             # type: PaperNote
├── entities/                 # 著者・運用会社・データセット・指数・ファクター等
│   ├── index.md
│   └── <slug>.md             # type: Person | Fund | Dataset | Factor | Benchmark ...
├── concepts/                 # 手法・現象・バイアス（momentum, HAR, lookahead-bias ...）
│   ├── index.md
│   └── <slug>.md             # type: Concept | Method | Anomaly | Bias | Metric
├── syntheses/                # 複数論文を横断する主張・論点
│   ├── index.md
│   └── <slug>.md             # type: Synthesis
└── queries/                  # 回答済みの調査質問
    ├── index.md
    └── <slug>.md             # type: Query
```

- **予約ファイル**（`index.md` / `log.md` / `EVALUATED.md`）には frontmatter を付けない。それ以外の `.md` は必ず frontmatter を持ち、非空の `type` を含む（OKF規約）。
- slug は英語 lowercase-kebab、60文字以内。同一論文の raw と summary は**同じ slug** を使う（例: `lookahead-freedom-temporal-non-interference.md`）。
- 日付・時刻は必ず `date -u +%Y-%m-%dT%H:%M:%SZ` で取得する。推測や記憶で書かない。
- 意味的リンクは標準Markdownリンク（バンドルルートからの絶対パス推奨: `[HAR](/concepts/har-model.md)`）。Obsidian wikilink は補助として併用可だが、各関係は同一ページ内に最低1回は標準Markdownリンクで出す（llm-wiki のハイブリッドリンク方針に従う）。

## 情報ソース

| ソース | アクセス方法 | 備考 |
|---|---|---|
| **arXiv q-fin** | API: `https://export.arxiv.org/api/query?search_query=cat:q-fin.TR+OR+cat:q-fin.PM+OR+cat:q-fin.ST+OR+cat:q-fin.CP&sortBy=submittedDate&sortOrder=descending&max_results=40`（curl で Atom XML 取得）。本文は `https://arxiv.org/html/<id>` を WebFetch、無ければ PDF | 最も機械的に取れる。ML系が多い。トピック指定時は `all:"<query>"+AND+cat:q-fin.*` で絞る |
| **SSRN** | WebSearch で `site:papers.ssrn.com <query>`。個別ページはボット対策で失敗しやすい → **zyte-fetch スキル**で browser rendering 取得。abstract 抽出には zyte の article 抽出も可 | 査読前の金融系WPの集積地。"momentum", "machine learning trading" 等 |
| **学術誌** | WebSearch: JF / JFE の early view・最新号、Journal of Portfolio Management（実務寄り） | 多くは SSRN に WP 版あり。本文が読めなければ SSRN 版を探す |
| **カンファレンス** | WebSearch: ICAIF（ACM AI in Finance）accepted papers、NeurIPS / KDD の金融ML論文 | ICAIF は例年秋。開催前後は重点的に |
| **運用会社リサーチ** | WebFetch: AQR `https://www.aqr.com/Insights/Research`、Man Group `https://www.man.com/insights`。取得失敗時は **zyte-fetch** | 実務家の質の高い無料論文。AQR "Fact, Fiction, and ..." 等 |

**クロールの原則:**
- まず curl / WebFetch を試す。**ボット対策・Cloudflare・JS レンダリング依存で空・不完全なコンテンツが返ったら、迷わず [[zyte-fetch]] スキルにフォールバック**する（browser rendering で client-side JS を実行、article/SERP の構造化抽出も可）。SSRN 個別ページと運用会社サイトは特に zyte が有効。
- 独立に調べられるソースは並列取得してよい（複数の WebSearch / WebFetch を同時に、または Explore/general-purpose エージェントにソースごと割り振る）。

## 論文の読み方 — アブストラクトで選別し、本文を raw に残す

**二段階で読む。** これが本スキルの中核動作:

1. **アブストラクトで重要度をトリアージする（安価な一次選別）。** タイトル＋アブストラクトだけで、株式オートトレードへの関連性（後述の評価軸のうち「関連性」）を判定する。ここで明らかに対象外（無関係・暗号資産のみ・既知の焼き直し・純粋なデリバティブ価格付け理論など）を落とす。**関連性 < 3 と見込まれるものは本文を取りに行かない。**
2. **トリアージを通過した候補のみ、本文を取得して `raw/<slug>.md` に保存する。**
   - arXiv は `https://arxiv.org/html/<id>` を WebFetch で本文取得（無ければ PDF から抽出）。SSRN・運用会社 PDF は WebFetch → 失敗時 zyte-fetch。
   - raw には**本文フルテキスト**を入れる（要旨だけでなく、手法・データ・結果・限界の記述を含む本文）。raw は**不変・編集禁止**の一次ソース。
   - frontmatter は `type: RawSource`（テンプレは後述）。
3. **raw の本文を精読して、4軸で採点する。** アブストラクトだけで採点しない。本文で手法・OOS検証・取引コスト・過剰適合の兆候を確認する。
4. **採点結果に基づき保存判定。** 基準を満たせば `summaries/<slug>.md`（PaperNote）を作成し、entities / concepts / syntheses を更新する。基準未満なら **`raw/<slug>.md` を削除**（知識として残さない）し、`EVALUATED.md` に棄却として記録する。

> ペイウォール等で本文がどうしても取得できない場合のみ、アブストラクト＋プレプリント＋著者ページで評価し、raw にはアブストラクト＋取得できた範囲を保存、summary の frontmatter を `full_text_read: false` とし、本文未読を明記する。原則は本文取得。

## 運用フロー

### 1. sweep — 最新情報の収集（デフォルト動作）

トリガー: 「クオンツ情報を集めて」「最新の論文チェックして」など、トピック指定のない依頼。

1. **`EVALUATED.md` を読む。** 直近の評価済みエントリ（URL / arXiv ID / SSRN ID / DOI）を把握し、再評価を避ける。前回 sweep の日付以降を対象期間とする（初回は直近3ヶ月）。
2. **`AGENTS.md` と `index.md` を読む。** 既存の分類・synthesis・カバー済みトピックを把握する。
3. **各ソースから候補を収集する。** arXiv API → SSRN 検索 → 運用会社リサーチ → カンファレンス/学術誌の順に、対象期間の新着を集める。候補ごとに「タイトル / 著者 / ソース / URL / 日付 / アブストラクト」を控える。
4. **アブストラクトで一次トリアージ**（「論文の読み方」ステップ1）。対象外を落とす。
5. **通過候補の本文を raw に保存して精読**（ステップ2–3）。4軸で採点する。
6. **保存。** 基準を満たしたもののみ summary を作成し、entities / concepts / syntheses と各 `index.md` を更新する（「知識ベースの構築」参照）。
7. **台帳更新。** 評価した**全候補**（保存・棄却とも）を `EVALUATED.md` に追記する。
8. **`log.md` に追記**し、報告する。保存ノートの一覧（1行サマリ付き）と、棄却数・棄却理由の傾向。

### 2. topic — トピック指定の調査

トリガー: 「momentum の研究を探して」「執行アルゴリズムの最新動向は?」など特定テーマの依頼。

sweep と同じ流れだが、全ソースに対して指定トピックの検索クエリで収集する。期間は直近1〜2年まで広げてよい。**既に該当する summary / synthesis / concept があれば先に提示し**、差分（新しい研究）を探す。回答を残す価値があれば `queries/<slug>.md`（type: Query）に保存してよい。

### 3. review — 個別 URL / 論文の評価

トリガー: ユーザーが特定の論文・URL・PDF を渡してきた場合。

アブストラクトでトリアージ → 通過すれば本文を raw に取得して精読 → 4軸採点。基準を満たせば summary 化し関連ページを更新、満たさなければ理由を説明して raw を削除し `EVALUATED.md` にのみ記録する。ユーザーが「それでも保存して」と言ったら保存する（summary の `status: user-requested`）。

### 4. query — 蓄積した wiki への問い合わせ

トリガー: 「クオンツwikiで調べて」「これまでの研究で X はどうだった?」「momentum と mean-reversion を横断して比較して」。

llm-wiki の query 手順に従う: `index.md` を読む → 候補ページを読み、Markdownリンクを1–2ホップ辿る → **すべての非自明な主張に出典ページを Markdownリンクで付けて**回答する。答えられない場合は正直にギャップを述べ、埋めるべきソースを提案する。保存する価値があれば `queries/<slug>.md` に残す。

### 5. lint — 健全性チェック

トリガー: 「クオンツwikiを整えて」「lint」。llm-wiki の lint 手順（OKF準拠チェック＋コンテンツ健全性）に加え、本スキル固有:
- summary と対応する raw が揃っているか（saved なのに raw が無い／`full_text_read: true` なのに raw が無い等）。
- `EVALUATED.md` と summary の整合（SAVED なのに summary が無い、その逆）。
- 矛盾する研究（例: あるファクターの有効性を肯定/否定する2論文）が相互リンクされ synthesis に反映されているか。

## 評価基準 — 保存するかどうかの判断

各候補を以下の4軸で 0〜5 点で採点する（**採点は本文精読後**に確定する）:

| 軸 | 問い |
|---|---|
| **関連性** | 株式のシステマティックトレーディング（シグナル生成・ポートフォリオ構築・執行・リスク管理）に直接関係するか |
| **実装可能性** | 個人〜小規模チームが入手可能なデータ・計算資源で再現・応用できるか。手法やデータの記述は具体的か |
| **新規性** | 既存の summary / synthesis や周知の結果に対して新しい知見があるか |
| **信頼性** | 著者・掲載先の実績。OOS検証・取引コスト考慮の有無。ルックアヘッドバイアスや過剰適合の兆候がないか |

**保存条件: 関連性 ≥ 3 かつ 合計 ≥ 12。** 迷ったら棄却する — このスキルの価値は選別の厳しさにある。ただし以下は例外的に保存してよい:

- 広く信じられている手法を**否定・反証**する研究（実運用のリスク回避に直結するため、実装可能性が低くても価値が高い）
- AQR の "Fact, Fiction, and ..." シリーズのような、分野の通説を整理するサーベイ・実務家論文

判断に使った採点は summary の frontmatter（`score`）に残す。

## 知識ベースの構築 — 何を summary/entity/concept/synthesis に落とすか

保存すると決めた論文について、同じターンで以下を更新する:

- **summaries/<slug>.md（type: PaperNote）** — その論文のキュレーションノート本体。後述フォーマット。`raw/<slug>.md` へ Markdownリンクを張る。
- **entities/** — 論文に登場し繰り返し出うる固有物: 著者（`type: Person`）、運用会社（`type: Fund`、例 AQR / Man Group）、データセット（`type: Dataset`、例 CRSP / VOLARE）、ベンチマーク・指数（`type: Benchmark`）。既存なら統合し `# Sources` に summary リンクを追記、無ければ作成。**2ソース以上で登場 or ユーザー要望**のときに作る（単発言及はページ化しない）。
- **concepts/** — 手法・現象・バイアス・指標: momentum, HAR, look-ahead bias, absorption ratio, Almgren–Chriss, foundation models 等（`type: Concept | Method | Anomaly | Bias | Metric`）。同じ基準で作成/統合。
- **syntheses/<slug>.md（type: Synthesis）** — 複数論文を横断する主張。新しい論文が既存 synthesis を補強/反証/複雑化するなら本文を改訂し `sources_count` / `last_reviewed` を更新。矛盾は消さず `# Open questions` に両論併記。
- **各 `index.md`** と `log.md` を同ターンで更新（バンドルルート＋触れた各サブディレクトリ）。

**矛盾は残す。** 保存済みノートと矛盾する研究を保存する場合、双方に相互リンクと矛盾の要旨を書き、該当 synthesis に反映する。消して上書きしない。

## テンプレート

### raw/<slug>.md — type: RawSource（本文・不変）

```markdown
---
type: RawSource
title: <原題>
description: <一文>
resource: <URL>
authors: [<著者>, ...]
source_date: <YYYY-MM-DD 論文の日付>
ingested_at: <YYYY-MM-DDTHH:MM:SSZ>
tags: [...]
---

<取得した本文フルテキスト。編集しない。>
```

### summaries/<slug>.md — type: PaperNote（キュレーションノート）

```markdown
---
type: PaperNote
title: <原題>
description: <一文TL;DR>
resource: <URL>
authors: [<著者>, ...]
source: <SSRN | arXiv | JF | JFE | JPM | ICAIF | NeurIPS | KDD | AQR | Man Group | ...>
source_date: <YYYY-MM-DD>
ingested_at: <YYYY-MM-DDTHH:MM:SSZ>
timestamp: <YYYY-MM-DDTHH:MM:SSZ>
tags: [momentum, machine-learning, execution, ...]
score: {relevance: n, implementability: n, novelty: n, credibility: n}
status: saved            # saved | user-requested
full_text_read: true     # 本文を精読したか。取得不能時のみ false
---

# <日本語の見出し（内容がひと目でわかるように）>

## TL;DR
<3行以内。何を主張し、なぜ重要か>

## 手法・データ
<使用データ、期間、モデル、検証方法。再現に必要な情報を優先>

## 主要な結果
<定量的な結果。シャープレシオ、超過リターン、統計的有意性など数字で>

## オートトレードへの示唆
<自分のトレーディングシステムにどう活かせるか / 何を避けるべきか。ここが最重要>

## 限界・懸念
<取引コスト、キャパシティ、データスヌーピング、再現性への疑問など>

## Entities & concepts
<登場する entity/concept への Markdownリンク: [HAR](/concepts/har-model.md) 等>

## 関連
<関連 summary/synthesis への Markdownリンク。矛盾する研究があれば明記>

## Source
[raw source](/raw/<slug>.md) — <URL>
```

### entities / concepts / syntheses / queries

llm-wiki スキルの各テンプレート（`type` と推奨 frontmatter、本文セクション構成）に従う。`AGENTS.md` にクオンツ向けの `type` 一覧と本文構成を記載してある — 作成/統合前に読む。

## 台帳と索引

### EVALUATED.md — 評価台帳（追記のみ・予約ファイル）

重複評価を防ぐための台帳。frontmatter は付けない。評価した全候補を1行ずつ追記する:

```markdown
# 評価台帳

> 追記のみ。形式: `- [YYYY-MM-DD] SAVED|REJECTED | <識別子(URL/arXiv ID/SSRN ID)> | <タイトル> | <一言理由>`

## [YYYY-MM-DD] sweep — 対象期間: ... / ソース: ...
- [2026-07-08] SAVED | arXiv:2506.12345 | Deep Momentum Networks Revisited | OOS検証堅牢、実装可能 (16点)
- [2026-07-08] REJECTED | ssrn.com/abstract=... | Crypto Sentiment Trading | 株式対象外
```

新しい sweep を始める前に必ず読み、識別子（URL、arXiv ID、SSRN abstract ID、DOI）で照合する。同じ論文の改訂版（v2 など）は、内容に実質的な更新がある場合のみ再評価する。**追記のみ。過去の判断を書き換えない**（覆す場合は新しい行として追記）。

### index.md / log.md

OKF 予約ファイル（frontmatterなし）。llm-wiki の規約に従う。ノートを保存したら**必ず同じターンで** バンドルルートと触れた各サブディレクトリの `index.md`、および `log.md` を更新する。`index.md` は1エントリ1行（`- [Title](/path.md) — hook`）。テーマ別の見出し（モメンタム / ファクター投資 / 機械学習シグナル / 執行・マーケットマイクロストラクチャ / リスク管理 / ポートフォリオ構築 / 市場の通説検証 / バックテスト・検証インフラ）で整理する。

## ルール

- **保存は選別の結果である。** 収集した候補を全部保存しない。基準未満は棄却し、raw を削除して台帳にだけ残す。
- **アブストラクトで選別、本文で採点。** トリアージはアブストラクト、採点は raw の本文精読後。安価に落として、残ったものだけ深く読む。
- **raw は不変。** raw/ のファイルは一次ソース。編集しない（棄却時の削除のみ可）。
- **数字で書く。** 「有効だった」ではなく「シャープレシオ 1.2（コスト控除後 0.8）」のように。原文に数字がなければ「定量結果なし」と明記する。
- **出典のない主張を書かない。** ノート内の主張はすべて元論文に帰属させる。自分の推論・意見は「示唆」セクションに限定し、推論とわかるように書く。query 回答も全主張に出典リンク。
- **矛盾は消さずに残す。** 相互リンク＋synthesis 反映。
- **これは研究情報の整理であり、投資判断そのものではない。** ノートには研究の内容と限界を書く。特定銘柄の売買推奨は書かない。

## 初回ブートストラップ

`QUANTS_HOME`（`06_Wiki/03_Quants/`）に OKF 構造が無い場合:
1. `raw/ summaries/ entities/ concepts/ syntheses/ queries/` を作成し、各に `index.md`（予約・frontmatterなし）を置く。
2. バンドルルートに `AGENTS.md`（type: Schema、下記テーマ定義）、`index.md`、`log.md`、`EVALUATED.md` を作成する。
3. `log.md` に `## [DATE] bootstrap | 03_Quants initialized` を記録する。
4. ユーザーに関心トピック（例: モメンタム、ML シグナル、執行、日本株特有の話題など）を確認し、最初の sweep のクエリに反映する。省略時は「momentum / factor investing / machine learning trading / portfolio optimization」を既定クエリとする。
