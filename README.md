# External Data Scout

[![GitHub release](https://img.shields.io/github/v/release/wan-huiyan/external-data-scout)](https://github.com/wan-huiyan/external-data-scout/releases)
[![license](https://img.shields.io/github/license/wan-huiyan/external-data-scout)](LICENSE)
[![last commit](https://img.shields.io/github/last-commit/wan-huiyan/external-data-scout)](https://github.com/wan-huiyan/external-data-scout/commits)
[![Claude Code](https://img.shields.io/badge/Claude_Code-skill-orange)](https://claude.com/claude-code)

Scout, document, and verify external data sources. Produces comprehensive HTML reports with field-level schemas, API response structures, copy-paste code snippets, and freshness rankings.

## Quick Start

```
You: "Find external data sources we can use to enrich our UK retail model"

Claude: [launches 5-phase workflow]
  Phase 1: Brainstorm — agrees scope and source list with you
  Phase 2: Research — parallel agents deep-dive each source
  Phase 3: Build — assembles an HTML report with source cards
  Phase 4: Verify — curl-tests every endpoint, checks field names
  Phase 5: Correct — fixes errors, marks unverifiable claims
```

## Installation

**Claude Code (plugin):**
```bash
claude plugin marketplace add wan-huiyan/external-data-scout
claude plugin install external-data-scout@wan-huiyan-external-data-scout
```

**Claude Code (git clone):**
```bash
git clone https://github.com/wan-huiyan/external-data-scout.git ~/.claude/skills/external-data-scout
```

**Cursor (per-project rule):**
```bash
mkdir -p .cursor/rules
# Copy SKILL.md content into .cursor/rules/external-data-scout.mdc with alwaysApply: true
```

## What You Get

- **Self-contained HTML report** with source cards grouped by tier (Foundation / Always-On / Enrichments)
- **Freshness ranking table** — all sources sorted by lag, with coloured badges
- **Field-level schemas** — exact column names, data types, sample values for every source
- **Copy-paste Python code** — 3-5 line fetch functions for each API/download source
- **Fact-check results** — every URL, series ID, and field name verified against live endpoints
- **Action items roadmap** — prioritised implementation steps with effort estimates

## How It Works

| Phase | What happens | Tools used |
|-------|-------------|------------|
| **1. Brainstorm** | Align on scope, source list, evaluation criteria | Conversation with user |
| **2. Research** | Parallel agents deep-dive each source | `data-researcher`, `research-analyst`, web search |
| **3. Build** | Assemble HTML report with source cards | Write, preview server |
| **4. Verify** | curl-test endpoints, check field names, verify freshness | Parallel verification agents |
| **5. Correct** | Fix errors, add warning badges, update version | Edit, preview server |

## Key Design Decisions

- **Verify everything.** Training knowledge is often close but not exact. One wrong field name wastes an engineer's afternoon. Phase 4 exists specifically to catch these.
- **Freshness-first ranking.** Sources sorted by lag (not alphabetically or by domain) because lag determines which sources cover the analysis window.
- **Transparency over confidence.** If a field name can't be verified (e.g., licensed platform with gated docs), the report says so with a warning badge rather than guessing.
- **Licensed platform escalation ladder.** For gated platforms (Pathmatics, SimilarWeb, Comscore), document what's publicly known (metrics, dimensions, export formats) and provide actionable next steps to get the exact schema.

## URL Verification Patterns

The skill includes battle-tested patterns for verifying data source URLs, learned from fact-checking 19 claims across 15 sources:

**Known-unstable (verify every time):**
- ONS API endpoints (retired Nov 2024, data still available via website JSON)
- FRED OECD-sourced series IDs (rotate when OECD restructures)
- UK building society/financial institution URL paths (change without redirects)

**Known-stable (high confidence):**
- Open-Meteo archive API, GOV.UK endpoints, FRED core US series, yfinance tickers

## Limitations

- **Cannot verify licensed platform field names** without platform access (Pathmatics, SimilarWeb, etc.)
- **Training knowledge URLs decay** — endpoints from 6+ months ago may have changed paths. Phase 4 catches this, but initial research may produce stale URLs.
- **No automated data pipeline creation** — the report documents how to fetch data; it doesn't build the pipeline for you
- **HTML output only** — no native PDF export (use browser print-to-PDF with the included print CSS)

## Dependencies

| Dependency | Required? | What happens without it |
|-----------|-----------|------------------------|
| Web search | Recommended | Research agents fall back to training knowledge (less current) |
| Preview server | Recommended | Can't verify HTML rendering during build phase |
| `curl` | Required for Phase 4 | Cannot verify API endpoints — all claims marked unverified |

## Triggers

The skill activates on phrases like:
- "Find data sources", "what APIs are available"
- "Scout enrichment sources", "data dictionary"
- "Catalog our data stack", "what open data can we use"
- "Verify these field names", "fact-check this data report"

## Related Skills

- [**agent-review-panel**](https://github.com/wan-huiyan/agent-review-panel) — Multi-agent adversarial review. Used in Phase 4 to run parallel verification agents against live endpoints.
- [**ai-trust-evaluation**](https://github.com/wan-huiyan/ai-trust-evaluation) — Evaluate trust and accuracy of AI-generated claims. Useful for scoring confidence in data source documentation.
- [**data-provenance-verifier**](https://github.com/wan-huiyan/data-provenance-verifier) — Verify that data files are genuine and have provenance docs. Complements Phase 4 for file-based sources.
- **frontend-design** / **theme-factory** — For building polished HTML reports in Phase 3 with consistent design systems.

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-04-15 | Initial release. 5-phase workflow, URL verification patterns, licensed platform escalation ladder. Battle-tested on a 15-source UK retail data dictionary. |

## License

MIT
