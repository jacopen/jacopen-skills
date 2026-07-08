---
name: quants
description: 株式オートトレードに役立つクオンツ研究情報の収集・選別スキル。SSRN、arXiv q-fin、学術誌（JF/JFE/JPM）、ICAIF等のカンファレンス、AQR・Man Group等の運用会社リサーチから最新のディスカッションを収集し、有益と判断したもののみ Markdown ノートとして /home/jacopen/vault/07_Quants/ に保存する。トリガー例:「クオンツ情報を集めて」「最新の論文をチェックして」「quants sweep」「momentum の研究を探して」「ファクター投資の新しい議論は?」など、クオンツ・システマティックトレーディング関連の情報収集依頼。
---

# quants — クオンツ研究情報の収集・選別スキル

株式のオートトレード（システマティックトレーディング）に役立つクオンツ研究の最新情報を収集し、**内容を評価した上で、有益と判断したもののみ** Markdown として保存する。

**あなた（LLM）がキュレーターである。** 網羅的なクリッピングではなく、「トレーディングシステムの設計・改善に影響を与えうるか」という基準での選別が仕事の本体。玉石混交のワーキングペーパーから玉だけを拾う。

## 保存先レイアウト

ルート: `/home/jacopen/vault/07_Quants/`（以下 `QUANTS_HOME`）

```
07_Quants/
├── INDEX.md            # 保存済みノートの索引（テーマ別）
├── EVALUATED.md        # 評価済みリスト（保存/棄却の両方）— 重複評価防止のための台帳
└── notes/
    └── <YYYY>/
        └── <slug>.md   # 論文・記事ごとのノート
```

- 初回実行時にディレクトリと `INDEX.md` / `EVALUATED.md` がなければテンプレート（後述）から作成する。
- slug は英語 lowercase-kebab、60文字以内。例: `2026-ml-momentum-crash-prediction.md`
- 日付は必ず `date -u +%Y-%m-%dT%H:%M:%SZ` で取得する。推測や記憶で書かない。

## 情報ソース

| ソース | アクセス方法 | 備考 |
|---|---|---|
| **arXiv q-fin** | API: `https://export.arxiv.org/api/query?search_query=cat:q-fin.TR+OR+cat:q-fin.PM+OR+cat:q-fin.ST+OR+cat:q-fin.CP&sortBy=submittedDate&sortOrder=descending&max_results=30`（curl で取得、Atom XML） | 最も機械的に取れる。ML系アプローチが多い。トピック指定時は `all:"<query>"+AND+cat:q-fin.*` で絞る |
| **SSRN** | WebSearch で `site:papers.ssrn.com <query>` を検索。個別ページの取得がボット対策で失敗したら **zyte-fetch スキル** にフォールバック | 査読前の金融系ワーキングペーパーの集積地。"momentum"、"machine learning trading" 等で検索 |
| **学術誌** | WebSearch: Journal of Finance / Journal of Financial Economics の early view・最新号、Journal of Portfolio Management（実務寄り） | 多くは SSRN に WP 版があるので、本文が読めない場合は SSRN 版を探す |
| **カンファレンス** | WebSearch: ICAIF (ACM International Conference on AI in Finance) の accepted papers、NeurIPS / KDD の金融ML論文 | 開催時期の前後（ICAIF は例年秋）は重点的に |
| **運用会社リサーチ** | WebFetch: AQR `https://www.aqr.com/Insights/Research`、Man Group `https://www.man.com/insights`。取得に失敗したら zyte-fetch | 実務家による質の高い無料論文。AQR の "Fact, Fiction, and ..." シリーズなど |

補足:
- ペイウォールで本文が読めない場合は、アブストラクト + SSRN/arXiv のプレプリント版 + 著者ページを探す。それでも要旨しか得られなければ、要旨ベースで評価し、ノートに「本文未読」と明記する。
- 独立に調べられるソースは並列で取得してよい（複数の WebSearch / WebFetch を同時に投げる、または Explore/general-purpose エージェントにソースごとに割り振る）。

## 運用フロー

### 1. sweep — 最新情報の収集（デフォルト動作）

トリガー: 「クオンツ情報を集めて」「最新の論文チェックして」など、トピック指定のない依頼。

1. **`EVALUATED.md` を読む。** 直近の評価済みエントリ（URL / arXiv ID / SSRN ID / DOI）を把握し、再評価を避ける。前回 sweep の日付以降を対象期間とする（初回は直近3ヶ月）。
2. **各ソースから候補を収集する。** arXiv API → SSRN 検索 → 運用会社リサーチ → カンファレンス/学術誌の順に、対象期間の新着を集める。候補ごとに「タイトル / 著者 / ソース / URL / 日付 / アブストラクト」を控える。
3. **一次選別。** アブストラクトの時点で明らかに対象外のもの（株式オートトレードに無関係、暗号資産・保険数理のみ、既知の内容の焼き直し）を落とす。
4. **二次評価。** 残った候補は可能な範囲で本文（または詳細ページ）を読み、後述の評価基準でスコアリングする。
5. **保存。** 基準を満たしたもののみノートを作成し、`INDEX.md` を更新する。
6. **台帳更新。** 評価した**全候補**（保存・棄却とも）を `EVALUATED.md` に追記する。
7. **報告。** 保存したノートの一覧（1行サマリ付き）と、棄却数・棄却理由の傾向を報告する。

### 2. topic — トピック指定の調査

トリガー: 「momentum の研究を探して」「執行アルゴリズムの最新動向は?」など特定テーマの依頼。

sweep と同じ流れだが、全ソースに対して指定トピックの検索クエリで収集する。期間は直近1〜2年まで広げてよい。既に保存済みのノートがあれば先に提示し、差分（新しい研究）を探す。

### 3. review — 個別 URL / 論文の評価

トリガー: ユーザーが特定の論文・URL・PDF を渡してきた場合。

その1件を取得して評価基準にかけ、基準を満たせばノート化、満たさなければ理由を説明して `EVALUATED.md` にのみ記録する。ユーザーが「それでも保存して」と言ったら保存する（`status: user-requested` を付ける）。

## 評価基準 — 保存するかどうかの判断

各候補を以下の4軸で 0〜5 点で採点する:

| 軸 | 問い |
|---|---|
| **関連性** | 株式のシステマティックトレーディング（シグナル生成・ポートフォリオ構築・執行・リスク管理）に直接関係するか |
| **実装可能性** | 個人〜小規模チームが入手可能なデータ・計算資源で再現・応用できるか。手法やデータの記述は具体的か |
| **新規性** | 既に保存済みのノートや周知の結果に対して新しい知見があるか |
| **信頼性** | 著者・掲載先の実績。アウトオブサンプル検証・取引コスト考慮の有無。ルックアヘッドバイアスや過剰適合の兆候がないか |

**保存条件: 関連性 ≥ 3 かつ 合計 ≥ 12。** 迷ったら棄却する — このスキルの価値は選別の厳しさにある。ただし以下は例外的に保存してよい:

- 広く信じられている手法を**否定・反証**する研究（実運用のリスク回避に直結するため、実装可能性が低くても価値が高い）
- AQR の "Fact, Fiction, and ..." シリーズのような、分野の通説を整理するサーベイ・実務家論文

判断に使った採点はノートの frontmatter に残す（後から選別基準を振り返れるように）。

## ノートのフォーマット

`QUANTS_HOME/notes/<YYYY>/<slug>.md`:

```markdown
---
type: PaperNote
title: <原題>
authors: [<著者>, ...]
source: <SSRN | arXiv | JF | JFE | JPM | ICAIF | NeurIPS | KDD | AQR | Man Group | ...>
url: <URL>
published: <YYYY-MM-DD 論文・記事の日付>
ingested_at: <YYYY-MM-DDTHH:MM:SSZ>
tags: [momentum, machine-learning, execution, ...]
score: {relevance: n, implementability: n, novelty: n, credibility: n}
status: saved            # saved | user-requested
full_text_read: true     # 本文を読めたか。false ならアブストラクト評価
---

# <日本語の見出し（内容がひと目でわかるように）>

## TL;DR
<3行以内。何を主張し、なぜ重要か>

## 手法・データ
<使用データ、期間、モデル、検証方法。再現に必要な情報を優先>

## 主要な結果
<定量的な結果。シャープレシオ、超過リターン、統計的有意性など数字で>

## オートトレードへの示唆
<自分のトレーディングシステムにどう活かせるか / 何を避けるべきか。ここが最重要セクション>

## 限界・懸念
<取引コスト、キャパシティ、データスヌーピングの可能性、再現性への疑問など>

## 関連
<保存済みの関連ノートへのリンク: [title](../2025/slug.md)。矛盾する研究があれば明記>
```

## 台帳と索引

### EVALUATED.md — 評価台帳（追記のみ）

重複評価を防ぐための台帳。評価した全候補を1行ずつ追記する:

```markdown
# 評価台帳

> 追記のみ。形式: `- [YYYY-MM-DD] SAVED|REJECTED | <識別子(URL/arXiv ID/SSRN ID)> | <タイトル> | <一言理由>`

## [YYYY-MM-DD] sweep
- [2026-07-08] SAVED | arXiv:2506.12345 | Deep Momentum Networks Revisited | OOS検証が堅牢、実装可能
- [2026-07-08] REJECTED | ssrn.com/abstract=... | Crypto Sentiment Trading | 株式対象外
```

新しい sweep を始める前に必ずこのファイルを読み、識別子（URL、arXiv ID、SSRN abstract ID、DOI）で照合する。同じ論文の改訂版（v2 など）は、内容に実質的な更新がある場合のみ再評価する。

### INDEX.md — 保存ノートの索引

テーマ別セクション（例: モメンタム / ファクター投資 / 機械学習シグナル / 執行・マーケットマイクロストラクチャ / リスク管理 / ポートフォリオ構築 / 市場の通説検証）に、1ノート1行:

```markdown
# Quants Research Index

## モメンタム
- [Deep Momentum Networks Revisited](notes/2026/deep-momentum-networks-revisited.md) — LSTM ベースのモメンタムはコスト考慮後も有効と主張 (arXiv 2026-06)
```

ノートを保存したら必ず同じターンで INDEX.md を更新する。セクションは必要に応じて追加してよい。

## ルール

- **保存は選別の結果である。** 収集した候補を全部保存しない。基準未満は棄却し、台帳にだけ残す。
- **数字で書く。** 「有効だった」ではなく「シャープレシオ 1.2（コスト控除後 0.8）」のように。原文に数字がなければ「定量結果なし」と明記する。
- **出典のない主張を書かない。** ノート内の主張はすべて元論文に帰属させる。自分の推論・意見は「示唆」セクションに限定し、推論であることがわかるように書く。
- **矛盾は消さずに残す。** 保存済みノートと矛盾する研究を保存する場合、双方の「関連」セクションに相互リンクと矛盾の要旨を書く。
- **`EVALUATED.md` は追記のみ。** 過去の判断を書き換えない。判断を覆す場合は新しい行として追記する。
- **これは研究情報の整理であり、投資判断そのものではない。** ノートには研究の内容と限界を書く。特定銘柄の売買推奨は書かない。

## 初回ブートストラップ

`QUANTS_HOME` が存在しない場合:
1. `notes/` ディレクトリを作成する。
2. `INDEX.md` と `EVALUATED.md` を上記テンプレートの見出しだけで作成する。
3. ユーザーに関心トピック（例: モメンタム、ML シグナル、執行、日本株特有の話題など）を確認し、最初の sweep のクエリに反映する。ユーザーが不在・省略した場合は「momentum / factor investing / machine learning trading / portfolio optimization」を既定クエリとする。
