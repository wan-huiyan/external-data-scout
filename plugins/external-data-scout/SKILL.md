---
name: external-data-scout
description: "Scout, document, and verify external data sources — both open/free and paid/licensed. Produces comprehensive reports with field-level schemas, API response structures, code snippets, and freshness rankings through a 5-phase workflow: brainstorm, research, build, verify, correct. Use this skill whenever the user wants to discover what external data is available, document data sources for a pipeline, catalog APIs/datasets/enrichment sources, or answer 'what data can we use and how do we access it?'. Also triggers for fact-checking or verifying claims about data source schemas, field names, or API structures. Triggers on: 'find data sources', 'what APIs are available', 'scout enrichment sources', 'data dictionary', 'catalog our data stack', 'verify these field names', 'fact-check this data report', 'what open data can we use'."
---

# External Data Scout

Build comprehensive, field-level data source documentation through a structured
5-phase workflow: brainstorm scope with the user, research each source in parallel,
build a polished report, verify every claim against live endpoints, and correct
any errors.

The output is a self-contained HTML report (or markdown) with source cards
containing exact field names, API response structures, code snippets, freshness
rankings, and licensing details — designed to make implementation a copy-paste
exercise.

## When to use

- User wants to document external data sources for a project
- User asks "what data can we use?" or "what APIs are available?"
- User wants a data dictionary or enrichment catalog
- User wants to fact-check a report's claims about data schemas
- User has a report with [VERIFY] flags or unconfirmed field names

## The 5-Phase Workflow

### Phase 1: Brainstorm (scope the research)

Before launching any research, align with the user on exactly what to investigate.
This prevents wasted agent work and ensures the report covers what matters.

**Steps:**
1. Ask the user what domain they're documenting (e.g., "UK retail enrichment
   sources", "marketing attribution data stack", "climate data for agriculture")
2. List sources you already know about from the project context (memory files,
   existing docs, code imports)
3. Ask the user to confirm/edit the list and add any they know about
4. Agree on evaluation criteria — what makes a source worth including? Common
   criteria for data sources:
   - **Freshness**: How recent is the latest data point? (daily > weekly > monthly)
   - **Access method**: API vs browser download vs manual
   - **Cost**: Free vs registration vs paid
   - **Licensing**: Open vs restricted vs commercial
   - **Relevance**: How directly does it help the use case?
5. Agree on the tier/layer architecture — how should sources be grouped? Examples:
   - By freshness (daily / weekly / monthly)
   - By layer (foundation / always-on / optional enrichment)
   - By domain (weather / macro / sentiment / market)
6. Ask if there are specific unknowns to investigate (e.g., "is Pathmatics daily
   or weekly?", "does the ONS API require auth?")

**Output**: A shared understanding of scope, a source list, and evaluation criteria.
Do not proceed to Phase 2 until the user confirms the scope.

### Phase 2: Research (parallel deep-dives)

Launch one research subagent per source (or per category of sources). Each agent
gathers field-level detail that the report will need.

**What each agent should return:**
- Provider name and URL
- Access method (REST API endpoint, download URL, Python library)
- Authentication requirements (none, API key, OAuth, login)
- Exact field names / column headers
- Data format (JSON structure, CSV columns, Excel sheet layout)
- Units and value ranges (index 100 = baseline, percentage, absolute)
- Date range (historical depth)
- Update frequency (daily, weekly, monthly)
- Lag (how many days/weeks behind is the latest data point?)
- Licensing terms
- Sample data snippet or API response
- How it helps the use case (what does it control for? why include it?)

**For discovering NEW sources** (not just documenting known ones), launch an
additional research agent with the brief: "Find sources I don't already know about.
Focus on [freshness criteria]. Areas to explore: [list of domains]."

**Agent types to use:**
- `voltagent-research:data-researcher` — for detailed source documentation
- `voltagent-research:research-analyst` — for discovering new sources
- General-purpose agent — when web search is needed

**Important**: Research agents often rely on training knowledge when web tools
are unavailable. Mark any claim that wasn't verified against a live endpoint
as needing verification in Phase 4.

**Parallelism**: Launch all agents simultaneously. While they run, start
scaffolding the HTML structure (Phase 3 prep).

### Phase 3: Build (assemble the report)

Synthesize all agent findings into a structured HTML report.

**Report structure:**

```
1. Header (title, subtitle, meta chips with counts)
2. Freshness ranking table (all sources sorted by lag, with badges)
3. Layer/tier architecture diagram (clickable, links to detail sections)
4. Source detail sections (grouped by tier):
   For each source:
   - Card with provider, access, frequency, lag, depth, licensing
   - Schema block with exact field names
   - Sample API response (JSON) or file structure
   - "Why it matters" explanation
   - Footer tags (effort, status, licensing)
5. Code snippets section (copy-paste Python for each source)
6. Action items table (prioritised implementation roadmap)
7. Footer (version, date, source documents)
```

**Styling guidance:**
- If the project has an existing design system (check for prior HTML deliverables
  in `deliverables/`), match it
- Otherwise use a clean default: system fonts, cream background (#faf8f4),
  dark table headers, colored tier/effort/lag badges
- Every source card title should be a clickable link (target="_blank") to the
  source's main page, with a ↗ indicator
- Use `<pre>` blocks for API responses with dark background for contrast
- Mobile-responsive (test at 768px breakpoint)

**Output format priority:**
1. HTML (preferred — best for visual quality and interactivity)
2. Markdown (fallback for simpler needs or when HTML isn't appropriate)
3. PDF (via HTML → print. Beware page breaks: add `page-break-inside: avoid`
   to source cards; test with a print preview before delivering)

**Verify in browser**: Always start a preview server and check the report renders
correctly. Check: no console errors, all sections visible, badges colored, links
clickable, code blocks formatted, tables not overflowing.

### Phase 4: Verify (fact-check every claim)

This is what separates a good report from a great one. Every factual claim in
the report should be checked against reality.

**Verification strategy by source type:**

| Source type | Verification method |
|---|---|
| REST API (free, no auth) | `curl` the endpoint, compare response to report |
| REST API (needs API key) | Use the library (e.g., `fredapi`), check series IDs exist |
| File download (free) | Download the file, read headers, compare to report |
| Licensed platform | Check public documentation; if unresolvable, add ⚠️ badge |
| Manual lookup | Visit the page, confirm data is where the report says |

**What to verify for each source:**
- [ ] Endpoint responds (200, not 404/403)
- [ ] Field names match exactly (spelling, casing, underscores)
- [ ] Data format matches (JSON structure, CSV columns)
- [ ] Units match (index vs percentage vs absolute)
- [ ] Latest data point date → actual lag matches claimed lag
- [ ] Licensing terms are as described

**Verification agents**: Launch a review panel or parallel verification agents.
Suggested panel:
- **API Verifier** — hits every REST endpoint with `curl`
- **Download Verifier** — downloads files, reads headers
- **Freshness Auditor** — checks latest data point for each source
- **Series Auditor** — verifies database/series IDs exist (FRED, ONS CDIDs, etc.)

**For claims that can't be verified** (e.g., licensed platform fields without
access), add a warning to the source card:
```html
<div class="callout amber">
  <strong>⚠️ Unverified:</strong> [specific claim] could not be confirmed
  without platform access. Verify with [who has access].
</div>
```

### Phase 5: Correct (update the report)

For each verification finding:

1. **Confirmed** — no change needed
2. **Wrong** — update the source card with the correct information
3. **Unverifiable** — add ⚠️ warning badge to the specific claim
4. **Source defunct** — remove the card, note in a "Removed sources" section

After corrections:
- Update the footer version (e.g., "v1" → "v1.1 — fact-checked")
- Re-verify in browser
- Produce a summary table: `Claim | Status (✓/✗/⚠️) | Correction`

## Freshness-Based Tier Classification

When the user hasn't specified their own tier system, use this default for
data source projects:

| Tier | Criteria | Auto-apply? |
|---|---|---|
| **L0 Foundation** | Core data the product can't work without | Always on |
| **L2 Always-On** | Daily frequency + free API/library + exogenous | Auto-apply |
| **L3 Optional** | Weekly/monthly, may need downloads, analyst chooses | Opt-in |

The key insight: **daily frequency + programmatic access = always-on**. If you can
fetch it in one line of Python with no auth, it should probably be auto-applied.
Weekly/monthly sources that require file downloads or have >2 week lag are optional
enrichments the analyst enables when relevant.

## Source Card Template

Each source card should follow this structure (adapt as needed):

```html
<div class="source-card" style="border-top: 3px solid [tier-color];">
  <div class="card-top">
    <div>
      <h3><a href="[source-url]" target="_blank">[Source Name] ↗</a></h3>
      <div class="card-meta">[Provider] · [Frequency] · [Lag] · [License]</div>
    </div>
    <span class="tier-badge">[TIER]</span>
  </div>
  <div class="field-grid">
    [Key metadata: provider, endpoint, frequency, lag, depth, licensing]
  </div>
  <div class="schema-block">
    [Exact field names, data types, sample values]
  </div>
  <div class="card-body">
    [Why it matters for the use case]
  </div>
  <div class="card-footer">
    [Effort badge] [License badge] [Other tags]
  </div>
</div>
```

## URL Verification Patterns (learned from v3 fact-check, April 2026)

Training knowledge about data source **existence** is usually correct —
institutions like ONS, FRED, BoE are stable. But training knowledge about
**URL paths and API endpoints** decays fast. The fact-check found:

### Known-Unstable Endpoints (verify every time)

| Provider | What changes | Pattern |
|---|---|---|
| **ONS** | API endpoints retire without redirect. The Beta API (`api.beta.ons.gov.uk/v1/timeseries/{cdid}/data`) was retired Nov 2024 with no successor at the same path. | Always verify the current API. The ONS *website* JSON endpoint (`www.ons.gov.uk/{topic_path}/timeseries/{cdid}/{dataset}/data`) has been stable longer but requires knowing the full topic path per CDID. |
| **FRED** | OECD-sourced series IDs get retired when OECD restructures data. 3 of 8 UK series IDs from training knowledge were dead. | Always `curl` the FRED series page before including it. Replacement series often exist with different IDs covering the same concept. |
| **Nationwide** | URL paths change (e.g., `/about/house-price-index` → `/house-price-index`). | Check the main page URL. Data download paths also shift. |

### Known-Stable Endpoints (high confidence, still verify)

| Provider | Endpoint | Notes |
|---|---|---|
| **Open-Meteo** | `archive-api.open-meteo.com/v1/archive` | Field names stable. API accepts variant spellings (e.g., both `windgusts_10m_max` and `wind_gusts_10m_max`). |
| **GOV.UK** | `gov.uk/bank-holidays.json` | Unchanged for years. Structure: `england-and-wales.events[]`. |
| **FRED** (core series) | `api.stlouisfed.org/fred/series/observations` | Core US series are rock-solid. UK/OECD-sourced series are the fragile ones. |
| **DESNZ Fuel** | `gov.uk/government/statistical-data-sets/oil-and-petroleum-products-weekly-statistics` | GOV.UK statistical datasets are stable. |
| **yfinance** | `yf.download("^FTMC")` | Yahoo Finance tickers are stable for major indices. |

### Systematic Anti-Patterns Discovered

1. **OECD-sourced FRED series are fragile.** FRED republishes OECD data, and
   when OECD restructures (which happens every few years), the FRED series IDs
   break. Always verify UK/international FRED series — US-domestic ones are safe.

2. **Training knowledge fabricates plausible-but-wrong series IDs.** Example:
   `EAFP` was listed as an ONS CDID but doesn't exist. The alphabetical
   sequence goes EAFS, EAFT, EAFU, EAFV, EAFW — no EAFP. The ID *looked*
   plausible but was hallucinated. Always verify series IDs against the
   provider's own catalog, not just check if the concept exists.

3. **Training knowledge can mislabel series.** `GBRPRCNTO01IXOBSAM` was
   labelled "PPI Non-Food" but is actually "Construction Production Volume".
   The OECD code `PRCNTO01` means Production:Construction:Total. Always
   check the actual title returned by the API, not just the series ID.

4. **Government data sources get discontinued.** The BoE CHAPS card spending
   series was active in training data but permanently discontinued April 2025.
   For any government/central bank data source, check whether it's still being
   updated — look for the latest data point date, not just whether the page loads.

5. **API retirement != data disappearance.** When ONS retired its Beta API,
   the data remained accessible via the website JSON endpoint at different URLs.
   If an API returns 404, search for the data through the provider's main site
   before concluding the data is gone.

6. **Licensed platforms rarely have public field schemas.** Pathmatics/Sensor
   Tower's API docs and data dictionaries are entirely gated behind their
   customer portal (`help.sensortower.com`). When documenting a licensed
   source, use this escalation ladder:
   1. Search for public API/developer docs (often gated — confirm this)
   2. Check public blog posts and product pages for metrics & dimensions
   3. Look for third-party integrations (R/Python packages, MCP servers,
      Domo/Fivetran connectors) that may expose partial schemas
   4. State clearly what IS known (metrics, dimensions, export formats)
      vs what ISN'T (exact column names) — don't invent field names
   5. Provide an actionable next step: "do a CSV export to see headers"
      or "check the customer help centre at [URL]"

   This applies broadly: SimilarWeb, Comscore, Nielsen, Kantar, and most
   commercial data platforms gate their schemas. The report should document
   what's publicly verifiable and flag the rest as "verify with platform access."

## Anti-Patterns

- **Don't fabricate field names.** If you can't verify a field name, say so.
  A report with "⚠️ unverified" badges is more trustworthy than one with
  confident-but-wrong column names. For licensed platforms, document the
  *concepts* (estimated spend, SOV, channel breakdown) and note that exact
  column names require platform access — this is more useful than plausible
  but potentially wrong names like `competitor_spend_display`.
- **Don't skip the verification phase.** Training knowledge is often close but
  not exact (e.g., `windgusts_10m_max` vs `wind_gusts_10m_max`). One wrong
  parameter name wastes an engineer's afternoon.
- **Don't list sources without field-level detail.** "ONS has retail data" is
  useless. "ONS CDID `EAFU` returns clothing & footwear volume index (2019=100),
  monthly, via the ONS website JSON endpoint" is actionable.
- **Don't mix up frequency and lag.** Frequency = how often new data appears.
  Lag = how far behind the latest point is. A monthly source with 2-week lag
  is fresher than a weekly source with 4-week lag.
- **Don't forget licensing.** An engineer who builds a pipeline on data they
  can't legally use commercially has wasted more than their time.
- **Don't trust OECD-sourced FRED series IDs from memory.** Always verify
  against FRED's live catalog. These IDs rotate when OECD restructures.

## Tips

- **Start the preview server early** — verify the HTML as you build, not just
  at the end
- **Use the freshness table as the executive summary** — stakeholders scan this
  first to understand what's available
- **Make every source title a clickable link** — this is the single most useful
  thing for verification (someone can click and check immediately)
- **Include code snippets** — the report should be actionable, not just
  informational. A 3-line Python fetch function eliminates "how do I use this?"
- **Group by tier, then by frequency within tier** — this gives the report a
  natural reading order from most important to least
