# dsh-skill-fold

DSH skill catalog folding: compress the injected skill directory to **name + ≤40-char one-line summaries**, sort by real usage frequency, and fold non-matching domains away per task. Saves persistent context tokens every round — with ~80 skills installed the official catalog costs roughly **2.7K tokens per turn**; folding cuts the bulk of that.

This is the slim successor to [`dsh-skill-lazy`](https://github.com/boomzikazita/dsh-skill-lazy): the search/authoring tools (`skill_search`, `skill_mece_check`, `cost_audit`) were split out into [`dsh-skill-search`](https://github.com/boomzikazita/dsh-skill-search) and [`dsh-skill-authoring`](https://github.com/boomzikazita/dsh-skill-authoring). This plugin registers **no tools** — it only rewrites the catalog message.

## What it does

1. **Compact catalog** (`agent/pre-step` hook, prepend): rewrites the official `skill-catalog` message into a compact `<system-reminder>` — each skill rendered as `` `name`: one-line summary `` (first sentence, ≤ 40 chars), sorted by recorded call frequency (alphabetical tie-break). `source.entries` is left untouched, so the harness's own digest/dedup logic is unaffected.
2. **Per-task domain folding**: classifies the current user message into one of 14 MECE domains (via `DOMAIN_MAP` + keyword fallback), expands only the matching domain plus the top-5 most-used skills, and collapses the rest into one line each — `⏳ 另有 N 个 XX skill（...）：需要时用 skill_search 检索`.
3. **Usage tracking** (`tools/post-execute` hook): every `skill` tool invocation is counted into `usage.json` (runtime data, gitignored); counts drive catalog ordering.
4. **Proactive match hint**: after each user message, injects a one-time hint of up to 3 relevant skills (scored by name match / substring / token overlap / Chinese bigram intersection).

## Installation

```bash
dsh plugin --profile web add github:boomzikazita/dsh-skill-fold
```

The plugin mounts via its bundle patch (`cordis.patch.yml`).

## Configuration

| Option | Default | Description |
| --- | --- | --- |
| `catalogDescriptionMaxLength` | `40` | Max characters of the one-line summary per skill. |
| `sortByUsage` | `true` | Sort catalog entries by recorded call frequency. |
| `matchThreshold` | `3` | Minimum relevance score for the proactive match hint. |
| `foldEnabled` | `true` | Enable per-task domain folding. |
| `foldKeepDomains` | `1` | Matching domains kept expanded. |
| `foldKeepFrequent` | `5` | Extra high-frequency skills kept expanded. |

## Relationship to the other plugins

| Plugin | Responsibility |
| --- | --- |
| `dsh-skill-fold` (this) | Catalog display compression + usage counting. No tools. |
| `dsh-skill-search` | `skill_search` + `cost_audit` tools (on-demand lookup, token-cost watchdog). |
| `dsh-skill-authoring` | `skill_scaffold` / `skill_check_name` / `skill_mece_check` / `skill_audit` (creation gates). |
| `dsh-skill-lazy` (retired) | Monolithic predecessor; archived. |
