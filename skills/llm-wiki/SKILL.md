---
name: llm-wiki
description: Build and maintain a persistent, LLM-curated knowledge base (LLM Wiki pattern) conformant with the Open Knowledge Format (OKF v0.1). Wiki bundles live under /home/jacopen/vault/06_Wiki/. Use this when the user wants to ingest a new source, query an existing KB, lint/health-check a KB, or add a new theme. Themes currently 01_PlatformEngineering and 02_IncidentManagement. Trigger phrases include "ingest", "取り込み", "wikiに入れて", "KBに追加", "query the wiki", "wikiで調べて", "lint the wiki", "wikiを整えて".
---

# llm-wiki — OKF-conformant knowledge base skill

This skill implements the **LLM Wiki** pattern (persistent, LLM-maintained, compounding KB) using the **Open Knowledge Format (OKF v0.1)** as the on-disk representation.

**You (the LLM) own the wiki.** The user curates sources and asks questions. You do the summarizing, cross-referencing, filing, and bookkeeping.

## OKF v0.1 — the rules you MUST follow

Reference: `https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md`

1. **Every non-reserved `.md` file has a YAML frontmatter block.**
2. **Every frontmatter block contains a non-empty `type` field.**
3. **Reserved filenames** — do NOT add frontmatter to these; they follow their own conventions:
   - `index.md` — directory listing for progressive disclosure.
   - `log.md` — chronological update history.
4. **Semantic relationships use standard Markdown links.** Obsidian wikilinks are permitted as a supplemental Obsidian-ergonomic layer — see "Hybrid link policy" below.
   - Standard Markdown, absolute from bundle root: `[Backstage](/entities/backstage.md)`
   - Standard Markdown, relative: `[Backstage](../entities/backstage.md)`
   - A link from A to B asserts a relationship. Consumers MUST tolerate broken links (so may you — flag them at lint time).
5. **Unknown keys are preserved.** You MAY add producer-specific keys; consumers won't reject them.

### Hybrid link policy (standard Markdown + Obsidian wikilinks)

OKF requires *semantic relationships* to be expressed via standard Markdown so any OKF consumer can parse them. Obsidian wikilinks are non-standard Markdown; a strict OKF consumer will treat them as opaque text or "broken links" (which the spec says consumers MUST tolerate). This wiki treats wikilinks as a **producer-specific extension for Obsidian ergonomics** — allowed alongside, not in place of, standard Markdown.

**Rules:**
1. **Every relationship you rely on MUST appear at least once as a standard Markdown link on the same page.** Typically in a `# Relationships`, `# Sources`, `# Related concepts`, or `# Entities & concepts` section. This is the OKF-visible edge.
2. **Additional inline mentions in prose MAY use Obsidian wikilinks** — e.g. "…similar to how [[entities/spotify|Spotify]] did it…". This gives you autocompletion, rename-safety, and graph-view density without duplicating the canonical link.
3. **Never introduce a new entity or concept only via wikilink.** The first mention on a page must be a standard Markdown link so the relationship exists at the OKF layer.
4. **Wikilink syntax** — prefer path form with display: `[[entities/backstage|Backstage]]`. Do not use bare-title wikilinks (`[[Backstage]]`) because they rely on Obsidian's global resolver rather than a file path.
5. **Lint checks BOTH.** Both link forms are lint-checked: standard Markdown for OKF conformance and graph completeness; wikilinks for target existence.

### Recommended frontmatter (use whenever meaningful)

| Field | Notes |
|---|---|
| `type` | **Required.** Short string, e.g. `Summary`, `Tool`, `Concept`, `Synthesis`, `Postmortem`, `Person`. |
| `title` | Human-readable display name. Fallback to filename if omitted. |
| `description` | Single sentence summary. Used by index generators and previews. |
| `resource` | URI to the underlying asset (source URL, doc, product page). |
| `tags` | YAML list of short strings for cross-cutting categorization. |
| `timestamp` | ISO 8601 of last meaningful change (`YYYY-MM-DDTHH:MM:SSZ`). |

### Producer-specific keys used in this wiki (all optional)

- `authors` — list of authors of the source.
- `source_date` — ISO 8601 date of the source itself (distinct from `timestamp`).
- `ingested_at` — ISO 8601 datetime this source was first ingested.
- `aliases` — list of alternative names (entity pages).
- `related` — list of paths this concept relates to.
- `severity` — for `Postmortem` type (e.g., `SEV-1`).
- `date` — ISO 8601 date of an incident.
- `last_reviewed` — ISO 8601 date of last synthesis review.
- `sources_count` — integer count of underlying sources for a synthesis.

## Layout

Root: `/home/jacopen/vault/06_Wiki/`

Each theme is a self-contained OKF bundle:

```
06_Wiki/
├── 01_PlatformEngineering/       # bundle root
│   ├── AGENTS.md                 # theme schema (has frontmatter, type: Schema)
│   ├── index.md                  # bundle-level catalog (reserved, no frontmatter)
│   ├── log.md                    # bundle-level activity log (reserved, no frontmatter)
│   ├── raw/                      # immutable sources (never edit)
│   │   ├── index.md              # (reserved, no frontmatter)
│   │   └── assets/
│   ├── summaries/
│   │   └── index.md              # (reserved, no frontmatter)
│   ├── entities/
│   │   └── index.md
│   ├── concepts/
│   │   └── index.md
│   ├── syntheses/
│   │   └── index.md
│   └── queries/
│       └── index.md
└── 02_IncidentManagement/        # same structure
```

The directory partitioning (`summaries/`, `entities/`, …) is a producer choice; OKF does not mandate it. Keep it consistent within each theme.

## Operations

The user's request will usually map to one of four ops. Route accordingly.

### 1. ingest — process a new source

Trigger: "ingest X", "この記事を取り込んで", "wikiに入れて", user drops a URL / file / paste.

Steps:
1. **Confirm theme.** If ambiguous, ask (`PlatformEngineering` or `IncidentManagement`?). If obvious, proceed.
2. **Read the theme's `AGENTS.md`** at `<theme>/AGENTS.md` — it defines the theme's taxonomy and conventions. Do this before every ingest.
3. **Capture the raw source** under `<theme>/raw/<slug>.md`.
   - URLs: WebFetch, save fetched markdown with OKF frontmatter:
     ```yaml
     ---
     type: RawSource
     title: <title>
     description: <one sentence>
     resource: <original URL>
     source_date: <YYYY-MM-DD if known>
     ingested_at: <YYYY-MM-DDTHH:MM:SSZ>
     tags: [...]
     ---
     ```
   - Pasted / uploaded: same frontmatter, omit `resource` if none.
   - Images: place under `raw/assets/` if user wants them local.
   - Slug: lowercase-kebab, ≤60 chars. Prefix with date if chronological ordering matters (e.g., postmortems).
4. **Discuss key takeaways with the user** in 3–6 bullets *before* mass-editing pages. Let the user redirect.
5. **Write the summary page** at `<theme>/summaries/<slug>.md`:
   ```yaml
   ---
   type: Summary
   title: <title>
   description: <one sentence TL;DR>
   resource: <source URL>
   authors: [...]
   source_date: <YYYY-MM-DD>
   ingested_at: <YYYY-MM-DDTHH:MM:SSZ>
   timestamp: <YYYY-MM-DDTHH:MM:SSZ>
   tags: [...]
   ---
   ```
   Body: `# TL;DR`, `# Key claims`, `# Entities & concepts` (markdown links), `# Notable quotes` (optional), `# Source` linking `[raw source](/raw/<slug>.md)`.
6. **Update entity and concept pages.** For each entity/concept touched:
   - Missing → create with the appropriate `type` (`Tool`, `Person`, `Company`, `Team`, `Incident`, `Concept`, `Pattern`, `Anti-pattern`, …). Follow the theme's `AGENTS.md` template.
   - Exists → integrate new information; append the summary link to the `# Sources` section; bump `timestamp`.
   - **Contradicts existing claims** → add a `# Contradictions / open questions` section with both sides cited. Never silently overwrite.
7. **Update syntheses.** If this source strengthens/weakens/complicates an existing synthesis, revise its body and update `sources_count` and `last_reviewed`. Bump `timestamp`.
8. **Update `index.md`** for the bundle root and every touched subdirectory. One line per entry: `- [<title>](/<path>) — one-line hook`.
9. **Append to `log.md`**:
   ```
   ## [YYYY-MM-DD] ingest | <slug> — <one-line note>
   - Added: <paths>
   - Updated: <paths>
   - Flagged: <contradictions or gaps>
   ```
10. **Report back** to the user: pages created / updated / flagged, and 1–2 follow-up questions.

### 2. query — answer a question from the wiki

Trigger: "wikiで調べて", "what does the KB say about X", "compare X and Y across sources".

Steps:
1. **Confirm theme** if not obvious.
2. **Read `index.md`** for the theme first — fastest route to relevant pages.
3. **Read candidate pages**, follow markdown links 1–2 hops.
4. **Answer with citations.** Every non-trivial claim cites its page via markdown link: `([Team Topologies notes](/summaries/2026-06-30-team-topologies.md))`. If falling back to `raw/`, cite that too.
5. **Note gaps honestly.** If the wiki can't answer, say so and suggest what source would fill it.
6. **Offer to file the answer.** If the answer is worth keeping (synthesis, comparison, discovered connection), ask "file this under `/queries/<slug>.md`?" If yes, save it with `type: Query` frontmatter and update `index.md` + `log.md` (`## [DATE] query | <slug>`).

Answer format follows the question: table for comparisons, bullets for enumerations, prose for explanations, a Marp deck for "make me slides".

### 3. lint — health-check the wiki

Trigger: "lint the wiki", "wikiを整えて", "health check", periodic upkeep.

**OKF conformance checks (do these first, they are fast):**
- Every non-reserved `.md` has parseable YAML frontmatter.
- Every frontmatter has a non-empty `type`.
- `index.md` / `log.md` have no frontmatter.
- All standard Markdown links resolve to files in the bundle (broken links are permitted by OKF but should be surfaced).
- All Obsidian wikilinks (`[[path|display]]`) resolve to files in the bundle.
- **Wikilink-only relationships** — for each page, every relationship expressed via a wikilink must also appear as a standard Markdown link somewhere on the same page (hybrid link policy). Flag violations.

**Content health:**
- **Contradictions** — pages making conflicting claims. Surface, don't reconcile silently.
- **Stale claims** — older claims superseded by newer sources.
- **Orphan pages** — no inbound markdown links. Either link from `index.md` or a hub page, or flag for removal.
- **Missing pages** — entities/concepts mentioned in ≥2 sources but no dedicated page.
- **Index drift** — files on disk missing from `index.md`, or `index.md` entries pointing to missing files.
- **Data gaps** — topics the user cares about but with thin coverage. Suggest sources.

Report as a punch list grouped by severity. Don't auto-fix contradictions — flag and ask.

Append `## [DATE] lint | <one-line summary>` to `log.md`.

### 4. new-theme — add a new theme

Trigger: "add theme X", "新しいテーマ X を作って".

Steps:
1. Pick the next numeric prefix (`03_`, `04_`, …).
2. Create the directory structure (`raw/assets`, `summaries`, `entities`, `concepts`, `syntheses`, `queries`).
3. Drop the AGENTS.md template (below) and let the user customize taxonomy.
4. Seed `index.md` (bundle root + one per subdirectory) and `log.md` with a `bootstrap` entry.

## Templates

### AGENTS.md (per theme)

```markdown
---
type: Schema
title: <ThemeName> — wiki schema
description: OKF-conformant schema and conventions for the <ThemeName> KB.
tags: [schema, okf]
timestamp: <YYYY-MM-DDTHH:MM:SSZ>
---

# <ThemeName> — wiki schema

## Purpose
<1–2 sentences: what this KB is for, whose perspective it takes>

## Taxonomy
- **Entity types used** — `Tool`, `Platform`, `Company`, `Team`, `Person`, `Incident`, ...
- **Concept types used** — `Concept`, `Pattern`, `Anti-pattern`, ...
- **Syntheses** — <topics worth cross-source synthesis pages>

## Page conventions
### summaries/<slug>.md — type: Summary
Recommended keys: title, description, resource, authors, source_date, ingested_at, timestamp, tags.
Body sections: TL;DR / Key claims / Entities & concepts / Notable quotes / Source.

### entities/<slug>.md — type: <Tool|Person|Company|Team|Incident|...>
Recommended keys: title, description, resource (canonical URL if any), aliases, tags, timestamp.
Body sections: Overview / Notable facts / Relationships / Sources.

### concepts/<slug>.md — type: <Concept|Pattern|Anti-pattern>
Recommended keys: title, description, related, tags, timestamp.
Body sections: Definition / Why it matters / Variants / Related / Sources.

### syntheses/<slug>.md — type: Synthesis
Recommended keys: title, description, tags, last_reviewed, sources_count, timestamp.
Body sections: Thesis / Evidence for / Evidence against / Open questions / Sources.

### queries/<slug>.md — type: Query
Recommended keys: title, description, tags, timestamp.
Body: the answer, with citations.

## Conventions
- Links: standard Markdown, absolute from bundle root: `[Backstage](/entities/backstage.md)`.
- Dates: ISO 8601.
- Language: <日本語 or English>.
- Contradictions: keep both sides under `# Contradictions / open questions`; never overwrite.

## Theme-specific notes
<Anything unique to this theme's domain>
```

### index.md (bundle root and subdirectories) — reserved, no frontmatter

```markdown
# <Bundle or subdirectory name> — index

> One line per entry: `- [Title](/absolute/path.md) — one-line hook`.

## <Category>
_（未登録 / entries appear here）_
```

### log.md — reserved, no frontmatter

```markdown
# <Bundle name> — activity log

> Append-only. `## [YYYY-MM-DD] <op> | <slug or note>` を先頭に、bullets を続ける。

## [YYYY-MM-DD] bootstrap | <theme> initialized
- ...
```

## Bookkeeping rules (always)

- **Never edit files under `raw/`** — immutable ground truth.
- **Every wiki page edit is a bookkeeping event** — update `index.md` and `log.md` in the same turn, and bump the touched page's `timestamp`.
- **Cite everything.** No orphan claims. If it isn't in a source, mark it clearly as your inference.
- **Prefer edits over new pages.** Only create a new entity/concept page when the topic is genuinely recurrent (≥2 sources or user-requested).
- **Preserve user intent.** Hand-edited pages are sacred — integrate around them.
- **Preserve unknown frontmatter keys** on round-trip (OKF rule).
- **Ask before big restructures.** Renames, splits, deletions — confirm first.
- **Wikilinks are supplemental only.** Every semantic relationship must also exist as a standard Markdown link on the same page (see "Hybrid link policy").

## Interaction style

- Show what you touched. After ingest: "Created X, updated Y and Z, flagged one contradiction on W."
- Stay in one language per response — match the user's language.
- Keep the discussion at ingest step 4 short: bullets, not essays.
- When in doubt about taxonomy or scope, re-read the theme's `AGENTS.md`.

## First-run bootstrap

If a theme's `AGENTS.md`, `index.md`, or `log.md` is missing when you start work on it, create them from the templates above. Log the event: `## [DATE] bootstrap | <theme> initialized`.
