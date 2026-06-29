# deep-research — A 13-Agent AI Research Team for Claude

**Turn one question into a fully-researched, fact-checked, cited report — automatically.**

`deep-research` is a multi-agent [Claude Skill](https://www.anthropic.com) that runs an entire research team for you: it clarifies your question, hunts down sources across the web, your own documents, and academic databases, **verifies every source's credibility**, synthesizes the evidence, hunts for gaps, **red-teams its own conclusions**, and writes a cited report — all without you lifting a finger after the first prompt.

> **Why it's different:** most "AI research" tools are a single model winging it. This is a *team of 13 specialized agents*, each accountable for one job. The source verifier doesn't write. The writer doesn't decide which citations are valid. The critic doesn't flatter — it attacks. That separation of duties is what makes the output trustworthy.

---

## What you get

- **13 specialized agents**, 7 phases, 3 quality gates — orchestrated end to end
- **Source verification built in** — every source scored 1–5 for credibility; weak sources flagged or dropped
- **Searches everywhere** — web, your uploaded documents, and Google Drive
- **Adversarial review** — a dedicated devil's-advocate agent surfaces hidden assumptions and reasoning risks before you ever see the report
- **Gap detection** — tells you what the evidence *doesn't* cover, not just what it does
- **Answers in your language** — ask in Vietnamese, get a Vietnamese report; ask in English, get English
- **Cited, structured output** — executive summary, findings, contradictions, gaps, critical assessment, references

**Keywords:** AI research agent · multi-agent research · deep research · autonomous research assistant · automated literature review · AI fact-checking · source verification · cited research reports · Claude skill · Claude Code · research automation · academic research assistant.

---

## Install

Clone the repo and drop the `deep-research/` folder into your skills directory.

### Claude Code

```bash
git clone https://github.com/kaynquang/multi-agent-research.git

# Global (available in every project):
cp -r multi-agent-research/deep-research ~/.claude/skills/deep-research

# …or per-project:
mkdir -p .claude/skills && cp -r multi-agent-research/deep-research .claude/skills/deep-research
```

Start a new session — the skill auto-activates the moment you ask for research.

### claude.ai (web / desktop)

```bash
cd multi-agent-research && zip -r deep-research.zip deep-research
```

Then upload `deep-research.zip` in **Settings → Capabilities → Skills**.

---

## Use it

Just ask:

```
Research the impact of AI on the Vietnamese labor market
```
```
What does the evidence say about intermittent fasting and longevity?
```
```
Give me a deep dive on solid-state battery commercialization
```

The Lead Researcher takes it from there and hands back a cited report.

---

## How it works

| Phase | Agent(s) | Job |
|-------|----------|-----|
| 1 | `clarifier` | Pin down the question, scope, language, depth |
| 2 | `methodology-designer` + `ethics-checker` | Design the search strategy + bias/ethics screen *(parallel)* |
| 3 | `web-scout` ×2 + `document-scout` + `academic-scout` | Hunt sources: web (broad + critical), your docs, academic *(parallel)* |
| 4 | `source-verifier` → `relevance-filter` | Score credibility → rank & dedupe |
| 5 | `evidence-synthesizer` + `gap-detector` | Synthesize evidence + find gaps *(parallel)* |
| 6 | `devils-advocate` | Attack the conclusions: weak points, hidden assumptions, risks |
| 7 | `report-writer` | Write the cited report in the question's language |

**Quality gates:** A — halt on ethical red flags · B — warn when verified sources fall below the depth-based minimum · C — loop back if the adversarial review finds a critical flaw.

Every agent follows one contract: **Role + Task + Input + Output + Quality Standards + Anti-Patterns.**

**Runs anywhere:**
- **Claude Code** — dispatches real subagents, parallelized within each phase (Mode A).
- **claude.ai** — the Lead Researcher plays each role in sequence, same process and gates (Mode B fallback).

---

## Structure

```
deep-research/
├── SKILL.md              # Lead Researcher + 7-phase orchestration
├── subagents/            # 12 specialist agents (one file each)
└── references/
    ├── quality-gates.md  # Credibility rubric + checkpoint logic
    └── output-format.md  # Final report template
```

---

*Built for Claude (Anthropic). MIT-friendly — fork it, extend it, ship your own research team.*
