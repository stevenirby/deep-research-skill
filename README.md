# Deep Research

A [Claude Code](https://claude.com/claude-code) skill that does actual research instead of one web search and a confident summary.

## Why

One search, a glance at the snippets, a tidy answer. That's how AI research usually goes, and it's wrong a lot — snippets misread pages, prices are stale, contested stuff gets flattened, gaps get filled with plausible nonsense.

This skill treats the report as a draft and refines it with evidence until it stops being wrong. Every claim has a clickable source. Gaps are stated, not guessed.

## What's TTD-DR

TTD-DR is **Test-Time Diffusion Deep Researcher** — the technique this skill is built on. The idea: treat writing a report like image diffusion. You start with a noisy draft and "denoise" it in passes. Here, each pass is a round of targeted search, and the evidence is what removes the noise. The draft is wrong at first by design — every round just makes it less wrong. That's why drafting comes *before* searching, not after.

## How it works

A 7-phase loop, capped at 3 search rounds:

1. **Draft** from what Claude already knows — mark unknowns `[NEEDS RESEARCH]`.
2. **Decompose** into 3–5 searchable sub-questions.
3. **Search** each one — and read the actual pages, not snippets.
4. **Revise** the draft with inline citations.
5. **Gap check** — what's unsourced, uncovered, or contradictory?
6. **Search the gaps**, revise again. Repeat 5–6 if needed.
7. **Finalize** — summary, findings, analysis, sources, gaps.

Small questions run inline. Big ones (3+ sub-questions, lots of fetches) get handed to a subagent so page content stays out of your conversation.

The rules it won't break: every product gets a URL, every price gets verified on the pricing page, nothing unverified gets stated as fact, no hallucinated links, "couldn't find it" beats guessing.

Runs entirely on Claude Code's built-in `WebSearch`, `WebFetch`, and `Agent`. No scripts, no API keys.

## Install

```bash
mkdir -p ~/.claude/skills/deep-research
curl -fsSL https://raw.githubusercontent.com/stevenirby/deep-research-skill/main/SKILL.md \
  -o ~/.claude/skills/deep-research/SKILL.md
```

## Use

```
/deep-research where is the best place to share skills with the world
```

Or just ask Claude for a cited deep-dive on something — it'll pick up the skill on its own.

## Example: before & after

Same question both times — *how much battery storage was installed in Q1 2026?* — answered the lazy way, then with the skill.

### Before — one search, quick summary

How most AI research goes. Search once, summarize the snippets, sound confident:

> Global battery storage installations hit about **360 GWh in 2026**, up 33% year-on-year. China led with **27.3 GWh** of storage batteries in Q1 2026, and the US installed a record **18.9 GW** of energy storage.

Reads fine. It's wrong three times:

- **360 GWh** is a full-*year* 2026 *forecast*, not Q1, and not actual.
- **27.3 GWh** is what China *exported* — not what was installed anywhere.
- **18.9 GW** is the US total for *all of 2025*, not Q1 2026.

Every number came from a real snippet. None of them answers the question.

### After — the deep-research skill

`/deep-research how much battery storage has been installed in Q1 2026?`

<details>
<summary>Show the report</summary>

> ## Executive Summary
> Fully reconciled Q1 2026 installation totals aren't published yet — quarterly install data lags the quarter's end by 1–3 months. What's confirmed: the US closed 2025 at a record 18.9 GW / 51 GWh and entered 2026 strong, China's monthly new-storage commissioning grew 60%+ YoY in January and 120%+ in February 2026, and full-year 2026 global deployment is forecast around 360 GWh.
>
> ## Key Findings
> - **US 2025 was a record 18.9 GW / 51 GWh**, +52% over 2024; Q4 2025 alone hit a record 5.8 GW. Q1 2026 actuals not yet published. — [Wood Mackenzie](https://www.woodmac.com/press-releases/2025-u.s.-energy-storage-installations-set-new-record-surpass-2024-by-52)
> - **US 2026 plan: ~24.3 GW** of new utility-scale battery storage (EIA); SEIA's Q1 2026 outlook projects 70 GWh / 35 GW for the full year. — [ESS News](https://www.ess-news.com/2026/02/26/new-us-battery-capacity-in-2026-24-3-gw-of-new-battery-storage-to-come-online/)
> - **China entered 2026 at 136 GW installed** (end-2025, +84% YoY); newly commissioned storage grew 60%+ YoY in Jan and 120%+ in Feb 2026. — [CGTN](https://news.cgtn.com/news/2026-04-03/China-s-new-type-energy-storage-capacity-to-exceed-370-GW-by-2030-1M2hwadn6QU/p.html)
> - **China exported 27.3 GWh** of stationary-storage batteries in Q1 2026 (+15% YoY) — that's exports, not domestic installs. — [Energy-Storage.news](https://www.energy-storage.news/chinas-cumulative-ess-battery-exports-hit-27-3gwh-in-q1-2026/)
> - **Global 2026 forecast: ~123 GW / 360 GWh** of deployment (BNEF), ~33% above 2025. — [Utility Dive](https://www.utilitydive.com/news/us-energy-storage-market-looks-resilient-amid-global-growth-bnef/803368/)
>
> ## Areas of Debate
> Forecasts vary widely by methodology — ~360 GWh (BNEF, systems) up to 600+ GWh (cell shipments). Wood Mackenzie warns US grid-scale could *shrink* ~⅓ in 2026 on policy/tariff headwinds, which sits against EIA's growth projection.
>
> ## Research Gaps
> No source published a clean, reconciled "X GWh installed globally in Q1 2026" figure as of mid-May 2026. China reports monthly YoY percentages but not a Q1 GWh total; the US Q1 2026 monitor edition mostly covers 2025; global Q1 actuals are unpublished. Treat any single Q1 2026 global number circulating now as an estimate.

</details>

Same sources. The difference is the skill *read* them, caught that exports ≠ installs and forecast ≠ actual, and said "not published yet" instead of guessing.

## License

MIT
