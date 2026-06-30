---
name: research-knowledge-work
description: "Class-level umbrella for research discovery, monitoring, prediction-market evidence, persistent knowledge bases, and ML/AI paper-writing workflows."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [research, arxiv, rss, knowledge-base, papers, polymarket, literature-review]
    related_skills: [mlops-model-operations, youtube-content]
---

# Research Knowledge Work

## Overview
Use this umbrella for research tasks that gather, monitor, structure, or turn evidence into deliverables. The class-level workflow is: discover sources, preserve citations/URLs, synthesize into a durable structure, and verify claims against primary material.

## When to Use
- Academic/literature search, especially arXiv.
- Monitoring blogs/RSS feeds for new posts.
- Building or querying a persistent markdown research wiki.
- Pulling prediction market probabilities/orderbooks/history.
- Planning, drafting, revising, or submitting ML/AI research papers.

## Source Discovery

### arXiv
No API key needed. Use the Atom API for paper search and exact ID fetches:
```bash
curl -s "https://export.arxiv.org/api/query?search_query=all:QUERY&max_results=5&sortBy=submittedDate&sortOrder=descending"
curl -s "https://export.arxiv.org/api/query?id_list=2402.03300"
```
Read abstracts from `/abs/<id>` and PDFs from `/pdf/<id>`. Preserve paper IDs and versions in summaries.

### Blog/RSS monitoring
Use `blogwatcher-cli` for feed discovery, OPML import, read/unread state, and scraping fallbacks.
```bash
blogwatcher-cli --help
blogwatcher-cli add <url>
blogwatcher-cli scan
blogwatcher-cli unread
```
Use a persistent data volume/path for recurring monitoring so read state is not lost.

### Prediction markets / Polymarket
Use Polymarket for market-implied probabilities, not ground truth. Public APIs are read-only and usually unauthenticated.
- Gamma API: discovery/search/events/markets.
- CLOB API: orderbooks/prices.
- Data API: history.

Prices are probabilities (Yes 0.65 ≈ 65%). Parse double-encoded JSON fields such as `outcomePrices` and `clobTokenIds` carefully.

## Persistent Knowledge Bases
Use the LLM Wiki pattern when knowledge should compound across sessions rather than be rediscovered.
```bash
WIKI="${WIKI_PATH:-$HOME/wiki}"
```
Maintain source/raw notes separately from synthesized pages. Keep frontmatter, cross-links, contradictions, and update history. Resume existing wikis before ingesting new material.

## Research Paper Writing Pipeline
Use for ML/AI papers targeting NeurIPS, ICML, ICLR, ACL, AAAI, COLM, etc. Treat paper writing as an iterative loop, not a linear document dump.

Phases:
1. Project setup: repo/workspace, contribution, TODOs, compute budget, collaborators.
2. Literature review: source discovery, related-work matrix, citation hygiene.
3. Experiment design: hypotheses, baselines, ablations, evaluation metrics.
4. Execution/monitoring: reproducible scripts, logs, artifacts, W&B or equivalent.
5. Analysis: tables, figures, error analysis, limitations.
6. Drafting: conference template, claims tied to evidence, clear narrative.
7. Self-review and revision: reviewer simulation, checklist, appendix/supplement.
8. Submission: formatting, anonymization, PDF checks, source package.

## Google Ads / PPC landing-page experience audits
Use this workflow when investigating Google Ads Quality Score, “Landing page experience: below average”, or PPC landing-page relevance/performance problems. See `references/google-ads-landing-page-experience.md` for a concrete Teoria.no-style diagnostic pattern and API field checklist.

1. Start from the actual keyword/ad group and final URL, not from the brand website you remember. If the user corrects the domain/product, pivot immediately.
2. If Google Ads account/API access exists, use it before making strong claims: pull campaign performance, keyword Quality Score components, expanded final URLs, ad final URLs, search terms, negatives, and conversion-goal configuration.
3. Test the exact final URL, mobile final URL, tracking-template-expanded URL, and obvious keyword URL variants with AdsBot/Googlebot user agents.
4. Check whether keyword-like paths incorrectly return `200` with a generic homepage (soft-404/fallback). Prefer relevant 301 redirects or true 404s, but do not overstate this if Google Ads expanded final URLs are already correct.
5. Compare message match above the fold: keyword → ad copy → `<title>`/meta description → H1 → intro → CTA. Exact commercial phrases often matter more than broad semantic relevance.
6. Run technical checks: HTTP status, robots/noindex, sitemap presence, canonical, mobile render, Lighthouse/mobile performance, FCP/LCP/CLS/TBT, and third-party JS weight.
7. Prioritize fixes in this order: correct final URLs/redirects if actually wrong, stronger above-the-fold message match, mobile performance/third-party script deferral, then conversion-goal cleanup.
8. Report separately: evidence from Google Ads API, evidence from live fetches, likely Google Ads interpretation, and recommended campaign/config changes.

## Evidence Rules
- Prefer primary sources (papers, docs, official APIs) over summaries.
- Include URLs/IDs/dates for every claim likely to age.
- Distinguish market belief, model output, experimental result, and human judgment.
- For papers, never invent citations or results; mark unknowns explicitly.

## Verification Checklist
- [ ] Source queries were run against live APIs/pages when current facts matter.
- [ ] URLs/IDs/versions are preserved.
- [ ] Syntheses separate evidence from interpretation.
- [ ] Persistent wiki/paper artifacts are written to the intended directory.
- [ ] Final deliverable includes open questions and next validation steps.
