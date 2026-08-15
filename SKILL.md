---
name: deep-research
description: |
  Iterative deep-research workflow (TTD-DR style) that produces claim-verified reports by drafting a hypothesis skeleton,
  decomposing into sub-questions, collecting exact supporting evidence, searching for contradictions, and revising across 1-3 rounds.
  USE WHEN the user asks for in-depth research, multi-source synthesis, cited summaries, fact-checking,
  spike investigation (HN/Reddit traffic), real-time data, or anything that benefits from
  "draft → search → revise → close gaps" rather than a single web search.
license: MIT
metadata:
  version: "2.1.0"
  technique: "TTD-DR (Test-Time Diffusion Deep Researcher)"
---

# Deep Research

You produce thorough, well-cited research reports by treating the report as a hypothesis that gets iteratively refined against claim-level evidence — not by searching once and summarizing pages.

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

## Tools

- **`WebSearch`** — grounded search with citations. Use first.
- **`WebFetch`** — read a specific URL. Read the page; don't just rely on search snippets.
- **`curl`** — verify the final public URL resolves successfully before citing it.
- **`Agent` (general-purpose subagent)** — spawn one for the full iterative loop when the task is large enough that doing it inline would burn the main context. The subagent runs Phases 1-7 below and returns the finished report.

## Decision: Inline vs Subagent

- **Inline** (work in the main conversation): scope is small (1-2 sub-questions), user is iterating with you, or speed matters more than thoroughness.
- **Subagent** (spawn `general-purpose` Agent): scope is broad (3+ sub-questions), task warrants ≥10 web fetches, or main context would get cluttered with raw page content. Pass it: the research question, this file's path (`~/.claude/skills/deep-research/SKILL.md`), and the explicit instruction to follow Phases 1-7. Ask it to return only the finished report.

## Research Process (TTD-DR — follow exactly)

### PHASE 1: Draft a Hypothesis Skeleton
Before searching, draft the report's structure, candidate hypotheses, and unknowns. Prior knowledge may suggest what to investigate, but it is **not evidence**.

- Mark every factual assertion from prior knowledge `[UNVERIFIED]`.
- Identify what evidence would confirm or falsify each central hypothesis.
- No `[UNVERIFIED]` assertion may survive finalization.

### PHASE 2: Decompose the Query
Break the research question into 3-5 specific, independently searchable sub-questions.

### PHASE 3: Initial Search Round
For each sub-question:
1. `WebSearch` the sub-question.
2. **Read the actual pages** with `WebFetch` — don't cite snippets. The page often says something different from the snippet.
3. If a high-relevance result didn't get fetched, fetch it explicitly.
4. For each material claim worth retaining, add an entry to the internal Evidence Ledger below.

### Evidence Ledger (internal working state)

The unit of evidence is a **claim**, not a webpage. Maintain this ledger while researching; do not expose it unless the user asks.

```text
Claim ID: C1
Claim: [the precise factual assertion]
Source URL: [the fetched page]
Exact passage: [a short verbatim passage that supports the claim]
Source type: [primary / official / peer-reviewed / reputable secondary / expert / community / marketing]
Published or updated: [date, if available]
Retrieved: [today's date]
Scope: [time period, geography, population, units, methodology]
Status: supported | contradicted | uncertain | superseded
```

Create ledger entries for **material claims**: numbers, dates, comparisons, causal statements, contested claims, recommendations, and facts that materially affect the conclusion. Do not bloat the ledger with trivial prose.

Every accepted claim must pass both checks:

1. **Provenance check:** the exact passage appears in the fetched source (allow whitespace normalization only). Search snippets do not count.
2. **Entailment check:** the passage supports the claim exactly as written, including its scope and strength.

Reject or rewrite a claim when its evidence changes any of these meanings:

- forecast vs. actual
- exports vs. installations or sales
- quarterly vs. annual
- stock vs. flow
- revenue vs. valuation or funding
- correlation vs. causation
- nominal vs. inflation-adjusted
- different dates, populations, regions, units, or methodologies

### PHASE 4: Revise Draft
Rewrite using **only accepted Evidence Ledger claims** for factual content. Add inline citations with URLs. Replace `[UNVERIFIED]` and `[NEEDS RESEARCH]` tags with supported findings, explicitly qualified uncertainty, or "could not verify."

Do not synthesize facts directly from snippets, remembered page content, or the preliminary draft.

### PHASE 5: Gap Analysis
Read your revised draft critically. Ask:
- What claims don't have sources?
- What aspects of the question haven't been covered?
- Are there contradictions to resolve?
- Which conclusions rely on only one source?
- Are multiple sources merely repeating the same original report?
- Which 3-5 claims are load-bearing — meaning the conclusion changes if they are wrong?
- What evidence would reverse or materially narrow the conclusion?
- Is newer evidence likely to supersede what was found?
- Has pricing / availability been verified on the actual pricing page?
- Are recommendations backed by evidence (real users, reviews, not marketing)?

List 2-4 specific gaps.

### PHASE 6: Targeted Gap Search (repeat 1-2 times)
For each gap:
1. Targeted `WebSearch` for that specific gap.
2. `WebFetch` and read the relevant pages.
3. Update the Evidence Ledger and revise the draft again.

For each load-bearing conclusion, perform at least one **adversarial search** intended to disprove, narrow, or supersede it. Classify apparent disagreement as `supports`, `contradicts`, `supersedes`, or `unrelated`. Check whether disagreement is actually caused by different dates, definitions, populations, methodologies, or units.

Repeat Phases 5-6 if significant gaps remain. **Hard cap: 3 total search rounds.** Stop earlier only when:

- every major sub-question is covered;
- load-bearing claims have strong evidence;
- important claims are independently corroborated where practical;
- contradictions are resolved or disclosed; and
- another round is unlikely to materially change the answer.

### PHASE 7: Finalize Report

**First — verify every material claim and citation.** Subagents and search tools hallucinate plausible-but-dead URLs, and a real page can still fail to support the sentence attached to it.

For each load-bearing claim, re-fetch its source immediately before finalizing and confirm:

1. The recorded exact passage is still present.
2. The passage entails the claim as written.
3. Dates, units, population, geography, and actual-vs-forecast status match.
4. No newer evidence found in the research supersedes it.

For every URL you're about to cite:
1. `curl -s -o /dev/null -w "%{http_code}" -L "URL"` — must return 200 (not 404/403/500).
2. WebFetch it and confirm the page content actually supports the claim you're citing it for.

If a source or evidence check fails, **remove or rewrite every dependent claim**. Never leave the factual statement intact with a missing citation or an "unverified" marker.

Then run the Final Integrity Gate:

- Every material factual claim has a citation.
- Every citation maps to an accepted Evidence Ledger entry.
- Every ledger entry contains a locatable supporting passage.
- Citation numbers and links resolve to the verified source; no invented or out-of-range citation survives.
- Unresolved contradictions and material gaps are disclosed.
- No `[UNVERIFIED]` or `[NEEDS RESEARCH]` marker remains.

If the polished synthesis fails this gate, return a simpler evidence-backed answer rather than the invalid synthesis.

Then use the output format below.

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

## Evidence Confidence
- High: [independently corroborated by primary or authoritative evidence]
- Moderate: [supported but dependent on a single or secondary source]
- Low: [incomplete, disputed, or rapidly changing]
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
8. **A real page is not proof of a claim.** Preserve the exact passage and verify that it entails the statement.
9. **Search against your conclusion.** Every load-bearing conclusion gets an adversarial search.
10. **Failed evidence invalidates the dependent claim.** Remove or qualify the claim, not just its citation.

## Search Ladder (when to escalate)

1. **`WebSearch`** — start here for any question.
2. **`WebSearch` + `WebFetch`** — when you need actual page content, not snippets.
3. **Multi-round TTD-DR loop (this skill)** — when the question is broad, contested, or needs synthesis across sources.
4. **`Agent` subagent running this skill** — when the loop is heavy and would clutter main context.

## Cost / Effort Notes

- `WebSearch` and `WebFetch` are built-in — no API keys, no separate billing.
- The cost of this skill is **context tokens and time**, not dollars. The 3-round cap exists to prevent runaway loops, not to save money.
- For genuinely massive research (dozens of sources, multi-day topics), spawn a subagent so the raw page content stays out of the main thread.
