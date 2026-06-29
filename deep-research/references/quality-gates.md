# Quality Gates

## Checkpoint A — After Phase 2 (Ethics Check)
**Trigger:** `ethics-checker` returns `proceed: false`
**Action:** Lead Researcher halts the pipeline and reports to user:
> "Research halted at ethics checkpoint. Concerns: [list flags]. [suggestion from ethics-checker]."

## Checkpoint B — After Phase 4 (Source Verification + Filtering)
**Trigger:** ANY of the following:
- Count of `verdict: accept` sources (credibility score ≥ 3) is below the depth-based minimum in the "Minimum Source Requirements by Depth" table below (quick=3, standard=5, deep=8)
- `overall_confidence: low` from source-verifier
- All 4 scout outputs are empty

**Action:** Lead Researcher reports to user:
> "Insufficient verified sources found ([N] sources passed verification). Options:
> 1. Broaden search scope — I'll re-run scouts with wider keywords.
> 2. Continue with current sources and flag confidence limitations in the report.
> Which do you prefer?"
Wait for user choice before continuing.

## Checkpoint C — After Phase 6 (Devil's Advocate)
**Trigger:** `devils-advocate` returns `overall_assessment: fundamentally_flawed` OR any `severity: critical` finding
**Action:** Lead Researcher flags to user:
> "Devil's advocate found a critical weakness: [finding]. This could undermine the report's validity. Options:
> 1. Loop back to Phase 5 with this weakness explicitly in scope.
> 2. Continue and prominently flag the limitation in the report."
Wait for user choice before continuing.

---

## Credibility Scoring Rubric (for source-verifier)

| Score | Meaning | Examples |
|-------|---------|---------|
| 5 | Highly credible | Peer-reviewed journal, government statistics, official institutional docs |
| 4 | Credible | Well-established news outlets, preprints from known research groups, official reports |
| 3 | Acceptable with caution | Expert blog posts, industry reports, conference proceedings |
| 2 | Weak — note but don't reject | Anonymous sources, opinion pieces, uncited secondary sources |
| 1 | Reject | No identifiable author, known misinformation source, broken/unverifiable citations |

**Default threshold:** Accept score ≥ 3. Score 2 sources get `verdict: accept_with_caution` and must be flagged explicitly in the report. Score 1 sources are rejected.

## Minimum Source Requirements by Depth

| Depth | Minimum verified sources (score ≥ 3) |
|-------|--------------------------------------|
| quick | 3 |
| standard | 5 |
| deep | 8 |

If minimum is not met → Checkpoint B triggers regardless of `overall_confidence` value.
