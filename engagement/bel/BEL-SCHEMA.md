# Behavioral Equivalence Ledger — Entry Schema

One YAML file per finding: `BEL-<PROGRAM>-<NNN>.yaml`

```yaml
bel_id: BEL-CBACT04C-001
program: CBACT04C
phase_detected: P4-differential
class: TARGET-DEFECT | LEGACY-QUIRK-PRESERVED | ORACLE-GAP | OPEN-DIVERGENCE
trap_category: decimal-arithmetic | move-truncation | sign-handling | collation-encoding | control-flow | file-status | date-handling | other
legacy_evidence:
  source: legacy/cbl/CBACT04C.cbl
  lines: "310-327"
  behavior: "COMPUTE without ROUNDED truncates monthly interest toward zero at scale 2"
target_evidence:
  source: target/carddemo-batch/src/.../InterestCalcProcessor.java
  behavior: "Initial implementation used RoundingMode.HALF_UP"
observed_divergence:
  scenario: happy-path-full-file
  detail: "Per-record diffs of $0.01 on subset of accounts; file total drift accumulates"
  artifact: engagement/bel/artifacts/BEL-CBACT04C-001-diff.txt
resolution:
  action: "Changed rounding to RoundingMode.DOWN at receiver scale"
  verified_by: charz run <date>, run-id <id>
  status: CLOSED | OPEN
human_review:
  reviewer: ""
  decision: preserve-legacy-behavior | approved-behavior-change | pending
  date: ""
```

Rules: entries are append-only; a status change is a new revision block, never an edit of history.
