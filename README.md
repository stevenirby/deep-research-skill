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

## License

MIT
