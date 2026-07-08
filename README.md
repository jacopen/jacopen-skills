# jacopen-skills

jacopen が個人的に利用する [Claude Code](https://claude.com/claude-code) 向けの **Skill** を開発・管理するリポジトリです。

## Skill とは

Skill は Claude Code に専門的な振る舞いや手順を追加する仕組みです。各 Skill は 1 つのディレクトリにまとまっており、中心となる `SKILL.md` に、いつ起動すべきか（`description`）と、起動後に従うべき手順が記述されています。ユーザーが `/<skill-name>` を実行するか、`description` にマッチするタスクを依頼すると呼び出されます。

## 収録している Skill

| Skill | 概要 |
|---|---|
| [`llm-wiki`](./skills/llm-wiki) | Open Knowledge Format (OKF v0.1) に準拠した、LLM が維持・更新する永続ナレッジベース（LLM Wiki パターン）を構築・運用する。ソースの取り込み、KB への問い合わせ、lint / ヘルスチェック、テーマ追加に対応。 |
| [`quants`](./skills/quants) | 株式オートトレードに役立つクオンツ研究情報を SSRN・arXiv q-fin・学術誌（JF/JFE/JPM）・ICAIF 等のカンファレンス・AQR や Man Group のリサーチページから収集し、評価基準に基づいて選別した上で、有益と判断したもののみ Markdown ノートとして保存する。 |

## ディレクトリ構成

```
jacopen-skills/
├── README.md
└── skills/
    └── <skill-name>/
        └── SKILL.md      # Skill 本体（frontmatter の name / description + 手順）
```

各 Skill は `skills/<skill-name>/SKILL.md` を持ちます。この `skills/<name>/SKILL.md` レイアウトは [`npx skills`](https://github.com/vercel-labs/skills) が検出する標準構成です。`SKILL.md` の frontmatter は最低限 `name` と `description` を含みます。

## 新しい Skill を追加する

1. `skills/<skill-name>/` ディレクトリを作成する。
2. その中に `SKILL.md` を作成し、frontmatter に `name` と `description` を記述する。
   - `description` は「何をする Skill か」に加えて「いつ起動すべきか（トリガーとなる語句や状況）」を含めると、Claude が適切に呼び出しやすくなる。
3. 本文に、起動後に従うべき手順・ルールを記述する。
4. この README の「収録している Skill」表に 1 行追加する。

### SKILL.md frontmatter の例

```yaml
---
name: skill-name
description: この Skill が何をするか。加えて、起動すべきタイミングやトリガー語句を書く。
---
```

## インストール

[`npx skills`](https://github.com/vercel-labs/skills)（Vercel Labs の agent skills ツール）でこのリポジトリから Skill をインストールできます。プロジェクトのルートで実行してください。

```bash
# 収録スキルの一覧を表示
npx skills add jacopen/jacopen-skills --list

# すべてのスキルをインストール
npx skills add jacopen/jacopen-skills

# 特定のスキルだけをインストール
npx skills add jacopen/jacopen-skills --skill llm-wiki

# Claude Code 向けにグローバル・非対話でインストール（CI 等）
npx skills add jacopen/jacopen-skills --skill llm-wiki -g -a claude-code -y
```

インストール後、Claude Code は次回セッション起動時に `SKILL.md` を自動で読み込みます。手動で配置・有効化する方法（`~/.claude/skills/` への配置やプラグイン経由など）については、[Claude Code のドキュメント](https://code.claude.com/docs/en/skills)を参照してください。
