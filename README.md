# deep-research

A multi-agent research skill for Claude. Instead of one agent doing everything, `deep-research` coordinates **1 Lead Researcher + 12 specialist agents** across 7 phases, with 3 quality-control checkpoints.

> Core principle: *the source verifier doesn't write the report. The writer doesn't decide which citations are valid. The critic doesn't just praise — it surfaces weak points, hidden assumptions, and reasoning risks.* Each agent is accountable for one part of the work.

## How it works

| Phase | Agent(s) | Responsibility |
|-------|----------|----------------|
| 1 | `clarifier` | Clarify the question; set scope, language, and depth |
| 2 | `methodology-designer` + `ethics-checker` | Design the search strategy + check for bias/ethics *(parallel)* |
| 3 | `web-scout` ×2 + `document-scout` + `academic-scout` | Find sources: web (broad + critical angles), user documents, academic *(parallel)* |
| 4 | `source-verifier` → `relevance-filter` | Score source credibility → rank & dedupe |
| 5 | `evidence-synthesizer` + `gap-detector` | Synthesize evidence + find research gaps *(parallel)* |
| 6 | `devils-advocate` | Adversarial review: weak points, hidden assumptions, reasoning risks |
| 7 | `report-writer` | Write the cited report, in the language of the question |

**Checkpoints:** A — halt on ethical concerns · B — warn when verified sources fall below the depth-based minimum · C — loop back if the adversarial review finds a critical flaw.

Each agent is defined with a fixed contract: **Role + Task + Input + Output + Quality Standards + Anti-Patterns.**

- **Sources:** web (search + fetch), user-provided documents, Google Drive.
- **Output:** a structured, cited research report; output language follows the question's language automatically.

## Installation

Clone the repo, then make the `deep-research/` folder available as a skill.

### Claude Code

Personal skills live in `~/.claude/skills/`. Copy (or symlink) the skill folder there:

```bash
git clone https://github.com/kaynquang/multi-agent-research.git
cp -r multi-agent-research/deep-research ~/.claude/skills/deep-research
```

Or scope it to a single project instead of globally:

```bash
mkdir -p .claude/skills
cp -r multi-agent-research/deep-research .claude/skills/deep-research
```

Restart Claude Code (or start a new session). The skill auto-activates when you ask it to research something.

### claude.ai (web/desktop)

1. Zip the skill folder: `cd multi-agent-research && zip -r deep-research.zip deep-research`
2. In claude.ai, go to **Settings → Capabilities → Skills** and upload `deep-research.zip`.

## Usage

Once installed, the skill activates automatically when you make a research request, e.g.:

```
Research the impact of AI on the Vietnamese labor market
```

The Lead Researcher then runs the full 7-phase pipeline and returns a cited report.

- **Claude Code:** dispatches real subagents, running them in parallel within each phase (Mode A).
- **claude.ai / environments without subagents:** the Lead Researcher plays each role sequentially, following the same process and checkpoints (Mode B fallback).

## Structure

```
deep-research/
├── SKILL.md              # Lead Researcher + 7-phase orchestration
├── subagents/            # 12 specialist agents (one file each)
└── references/
    ├── quality-gates.md  # Credibility rubric + checkpoint logic
    └── output-format.md  # Final report template
```
