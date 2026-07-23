# CLAUDE.md

このリポジトリは jacopen 個人用の [Claude Code](https://claude.com/claude-code) 向け **Skill** を開発・管理するもの。実行コードは持たず、成果物は `SKILL.md`（Markdown）のみ。

## リポジトリ構成

```
jacopen-skills/
├── README.md
├── CLAUDE.md
└── skills/
    └── <skill-name>/
        └── SKILL.md      # Skill 本体
```

`skills/<name>/SKILL.md` レイアウトは [`npx skills`](https://github.com/vercel-labs/skills) が検出する標準構成。この配置を崩さないこと。

収録スキル: `llm-wiki`（OKF v0.1 準拠の LLM Wiki 基盤）、`quants`（クオンツ研究の収集・選別。`llm-wiki` の OKF 規約を継承する）。

## Skill を書くときのルール

- frontmatter は最低限 `name` と `description`。`name` はディレクトリ名と一致させる。
- `description` には「何をするか」だけでなく **いつ起動すべきか（トリガー語句・状況）** を含める。日本語トリガーで起動させたいスキルは日本語のトリガー例も列挙する。
- 本文は「LLM が起動後に従う手順・判断基準」を書く。ユーザー向けの解説ではなく、実行者向けの指示として書く。
- 記述言語は既存スキルに合わせる（`llm-wiki` は英語、`quants` は日本語）。既存ファイルを編集するときは、そのファイルの言語・見出しスタイル・語調を踏襲する。
- スキル間で規約を重複させない。共通規約（OKF など）は元スキルを参照し、差分だけを書く。

## Skill を追加・変更したとき

**必ず README.md の「収録している Skill」表を更新する。** スキルの追加・削除・概要の変更時に忘れやすい。

frontmatter を編集したら YAML として妥当か確認する（過去に frontmatter の YAML 崩れを修正したコミットがある）。`description` は 1 行の値。コロンや引用符を含む場合の扱いに注意。

## Git

- コミットメッセージは英語・命令形の 1 行サマリ（例: `Add quants skill for curating quant trading research`）。
- `main` で作業してよい。コミット・プッシュはユーザーから指示があったときだけ行う。
