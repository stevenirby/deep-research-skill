# Deep Research — a Claude Code skill

An iterative, **TTD-DR-style** research skill for [Claude Code](https://claude.com/claude-code). It produces thorough, well-cited research reports by treating the report as a *draft that gets refined through targeted search* — not by searching once and summarizing.

## Why this exists

Most "research" with an AI assistant is shallow by default: one web search, a glance at the snippets, a confident summary. That fails in predictable ways — snippets misrepresent pages, prices are stale, contested claims get flattened, and gaps get papered over with plausible-sounding filler.

This skill fixes that by borrowing the **TTD-DR** idea (*Test-Time Diffusion Deep Researcher*): start from a rough draft, then iteratively "denoise" it with evidence. Each search round makes the report less wrong. The result is a report where every claim traces to a source you can click, and honest gaps are stated as gaps instead of guessed.

It earns its weight when **iteration matters** — broad, contested, or synthesis-heavy questions. For a one-shot question with an obvious answer, a plain web search is still the right tool, and the skill says so.

## How it works

The skill runs a fixed 7-phase loop:

1. **Draft from knowledge** — write a preliminary draft *before* searching, marking unknowns with `[NEEDS RESEARCH]`. This surfaces what you know and, more importantly, what you don't.
2. **Decompose** — break the question into 3–5 independently searchable sub-questions.
3. **Initial search round** — search each sub-question and **read the actual pages**, not just snippets.
4. **Revise draft** — rewrite with inline citations; replace every `[NEEDS RESEARCH]` tag with a real finding or an explicit "could not verify."
5. **Gap analysis** — critically reread the draft: unsourced claims? uncovered angles? contradictions? unverified pricing?
6. **Targeted gap search** — search each gap specifically and revise again. Repeat phases 5–6 as needed.
7. **Finalize** — emit the report in a fixed structure (Executive Summary → Key Findings → Detailed Analysis → Consensus / Debate → Sources → Research Gaps).

**Hard cap: 3 total search rounds**, to keep the loop from running away.

### Inline vs. subagent

The skill decides where to run based on scope:

- **Inline** — small scope (1–2 sub-questions), or you're iterating live, or speed beats thoroughness.
- **Subagent** — broad scope (3+ sub-questions, ≥10 fetches expected). It spawns a `general-purpose` agent so raw page content stays out of the main conversation, and that agent returns only the finished report.

### Non-negotiable rules

The skill enforces a short list of rules that exist because they're the things AI research gets wrong most often:

- Every tool/product/resource gets a **clickable URL** — no exceptions.
- Every price claim is **verified on the actual pricing page**.
- Nothing unverified is presented as fact — it's labeled "unverified" or dropped.
- **No hallucinated links.** Only URLs actually retrieved or fetched.
- **Read the pages.** Snippets are misleading.
- **"Could not find X" beats guessing.**
- No parroting marketing copy — check real users, reviews, HN/Reddit.

## Tools used

Built entirely on Claude Code's **built-in** tools — no scripts, no API keys, no separate billing:

- `WebSearch` — grounded search with citations
- `WebFetch` — read a specific URL's actual content
- `Agent` — spawn a `general-purpose` subagent for heavy loops

The only cost is context tokens and time. The 3-round cap exists to prevent runaway loops, not to save money.

## Installation

Drop the skill into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/deep-research
curl -fsSL https://raw.githubusercontent.com/stevenirby/deep-research-skill/main/SKILL.md \
  -o ~/.claude/skills/deep-research/SKILL.md
```

Or clone and copy:

```bash
git clone https://github.com/stevenirby/deep-research-skill.git
mkdir -p ~/.claude/skills/deep-research
cp deep-research-skill/SKILL.md ~/.claude/skills/deep-research/SKILL.md
```

Claude Code discovers the skill automatically. Invoke it with `/deep-research <your question>`, or just ask for in-depth research and it activates on its own.

## Usage

```
/deep-research where is the best place to share skills with the world (open source)
```

Or, in plain conversation: *"do some deep research on X and give me a cited report."*

## License

MIT — see [LICENSE](LICENSE).
