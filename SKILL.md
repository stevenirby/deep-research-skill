---
name: deep-research
description: |
  Iterative deep-research workflow (TTD-DR style) that produces well-cited reports by drafting from knowledge,
  decomposing into sub-questions, searching, revising, and gap-analyzing across 1-3 rounds.
  USE WHEN the user asks for in-depth research, multi-source synthesis, cited summaries, fact-checking,
  spike investigation (HN/Reddit traffic), real-time data, or anything that benefits from
  "draft → search → revise → close gaps" rather than a single web search.
license: MIT
metadata:
  version: "2.0.0"
  technique: "TTD-DR (Test-Time Diffusion Deep Researcher)"
---

# Deep Research

You produce thorough, well-cited research reports by treating the report as a draft that gets iteratively refined through targeted search — not by searching once and summarizing.

## When to Apply

Use this skill when:
- Conducting in-depth research on a topic
- Synthesizing information from multiple sources / perspectives
- Creating research summaries with proper citations
- Evaluating source credibility or comparing competing claims
- **Spike investigation** — finding where traffic is coming from (HN threads, Reddit posts, etc.)
- **Fact-checking** — verifying claims with sources
- **Real-time data** — breaking news, current events, live prices
- **Cloudflare-blocked sites** — when one fetch fails, try another or use a subagent

For a one-shot question with an obvious answer, just use `WebSearch` directly. This skill earns its weight when iteration matters.

## Tools (built-in, no scripts)

- **`WebSearch`** — grounded search with citations. Use first.
- **`WebFetch`** — read a specific URL. Read the page; don't just rely on search snippets.
- **`Agent` (general-purpose subagent)** — spawn one for the full iterative loop when the task is large enough that doing it inline would burn the main context. The subagent runs Phases 1-7 below and returns the finished report.

## Decision: Inline vs Subagent

- **Inline** (work in the main conversation): scope is small (1-2 sub-questions), user is iterating with you, or speed matters more than thoroughness.
- **Subagent** (spawn `general-purpose` Agent): scope is broad (3+ sub-questions), task warrants ≥10 web fetches, or main context would get cluttered with raw page content. Pass it: the research question, this file's path (`~/.claude/skills/deep-research/SKILL.md`), and the explicit instruction to follow Phases 1-7. Ask it to return only the finished report.

## Research Process (TTD-DR — follow exactly)

### PHASE 1: Draft from Knowledge
Before searching, write a preliminary draft from what you already know. This draft is your skeleton — it surfaces what you know and what you DON'T. Mark unknowns with `[NEEDS RESEARCH]`.

### PHASE 2: Decompose the Query
Break the research question into 3-5 specific, independently searchable sub-questions.

### PHASE 3: Initial Search Round
For each sub-question:
1. `WebSearch` the sub-question.
2. **Read the actual pages** with `WebFetch` — don't cite snippets. The page often says something different from the snippet.
3. If a high-relevance result didn't get fetched, fetch it explicitly.

### PHASE 4: Revise Draft
Rewrite incorporating what you learned. Add inline citations with URLs. Replace `[NEEDS RESEARCH]` tags with real findings (or "could not verify").

### PHASE 5: Gap Analysis
Read your revised draft critically. Ask:
- What claims don't have sources?
- What aspects of the question haven't been covered?
- Are there contradictions to resolve?
- Has pricing / availability been verified on the actual pricing page?
- Are recommendations backed by evidence (real users, reviews, not marketing)?

List 2-4 specific gaps.

### PHASE 6: Targeted Gap Search (repeat 1-2 times)
For each gap:
1. Targeted `WebSearch` for that specific gap.
2. `WebFetch` and read the relevant pages.
3. Revise the draft again.

Repeat Phases 5-6 if significant gaps remain. **Hard cap: 3 total search rounds.**

### PHASE 7: Finalize Report
Use the output format below.

## Output Format

```markdown
## Executive Summary
[2-3 sentence overview of key findings]

## Key Findings
- **[Finding]**: [Brief explanation] — [Source](URL)
- **[Finding]**: [Brief explanation] — [Source](URL)

## Detailed Analysis

### [Subtopic 1]
[In-depth analysis with inline links]

### [Subtopic 2]
[In-depth analysis with inline links]

## Areas of Consensus
[What sources agree on]

## Areas of Debate
[Where sources disagree or uncertainty exists]

## Comparison / Recommendations (if applicable)
[Table or structured comparison with verified data]

## Sources
[1] [Full citation with credibility note] — URL
[2] [Full citation with credibility note] — URL

## Research Gaps
[What couldn't be verified or needs further investigation]
```

## Source Evaluation Criteria

When citing, weight by credibility:
- **Peer-reviewed journals** — highest
- **Official reports / statistics** — authoritative data
- **Reputable news outlets** — timely, fact-checked
- **Expert commentary** — qualified opinions
- **General websites / blogs** — verify independently before trusting
- **Marketing pages** — never cite as fact; verify with reviews / users

## ⚠️ Mandatory Rules (non-negotiable)

1. **Every tool, product, or resource MUST have a clickable URL.** No exceptions.
2. **Every price/cost claim MUST be verified on the actual pricing page.** "Contact sales" = say so. Don't recommend enterprise tools without flagging the pricing model.
3. **NEVER present unverified info as fact.** If you only saw it in a search snippet but couldn't confirm on the page, mark it "unverified" or drop it.
4. **NO hallucinated links.** Only include URLs you actually retrieved from search results or fetched pages.
5. **READ the pages.** Snippets are misleading. Fetch and read what the page actually says.
6. **"Could not find X" beats guessing.** Honest gaps are better than confident fiction.
7. **No parroting marketing copy.** Check reviews, Reddit, HN, real user experiences before recommending.

## Search Ladder (when to escalate)

1. **`WebSearch`** — start here for any question.
2. **`WebSearch` + `WebFetch`** — when you need actual page content, not snippets.
3. **Multi-round TTD-DR loop (this skill)** — when the question is broad, contested, or needs synthesis across sources.
4. **`Agent` subagent running this skill** — when the loop is heavy and would clutter main context.

## Cost / Effort Notes

- `WebSearch` and `WebFetch` are built-in — no API keys, no separate billing.
- The cost of this skill is **context tokens and time**, not dollars. The 3-round cap exists to prevent runaway loops, not to save money.
- For genuinely massive research (dozens of sources, multi-day topics), spawn a subagent so the raw page content stays out of the main thread.
