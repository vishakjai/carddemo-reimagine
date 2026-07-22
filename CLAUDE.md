# CardDemo Reimagine Engagement — IDE Agent Instructions

**Engagement ID:** SYN-ENG-CARDDEMO-001
**Track:** Reimagine (legacy modernization)
**Source system:** AWS CardDemo — mainframe credit card management application (COBOL / CICS / VSAM / JCL)
**Target system:** Java 21 / Spring Boot 3.x / Spring Batch 5.x
**Governing methodology:** Characterization-first. No target code is written for a program until its frozen oracle exists and its characterization suite passes against legacy behavior.
**Prime directive:** *Behavioral equivalence is the deliverable. Generated code is a byproduct.* Every assertion about equivalence must be backed by a Behavioral Equivalence Ledger (BEL) entry with evidence. If you cannot prove it, record it as an open divergence — never silently "fix" legacy behavior.

---

## 1. Repository layout

```
/legacy/                  READ-ONLY. Never modify anything under this path.
  cbl/                    31 COBOL programs (CB* = batch, CO* = CICS online, CS* = utility)
  cpy/                    41 copybooks (record layouts — the data dictionary of the system)
  cpy-bms/                Symbolic map copybooks for CICS screens
  jcl/                    47 JCL jobs (batch orchestration — defines job step sequence & datasets)
  proc/  ctl/  catlg/     Procedures, control cards, catalog definitions
  bms/  csd/              CICS screen maps and resource definitions (OUT OF SCOPE Phase 1)
  data/ASCII/             Sample datasets, ASCII-converted
  data/EBCDIC/            Sample datasets, original EBCDIC encoding  ← characterization inputs
  scheduler/              Batch dependency/flow definitions
/engagement/
  discovery/              Phase 1 outputs: inventory, call graphs, data lineage, BRD
  characterization/
    oracles/              Frozen oracle outputs (golden files) per program per scenario
    tests/                Executable characterization tests (JUnit 5)
  bel/                    Behavioral Equivalence Ledger entries (one file per finding)
  provenance/             Coverage provenance manifests (T1/T2/T3 per program)
/target/                  Java/Spring Batch implementation (Maven multi-module)
```

## 2. Engagement phases (execute in order — Conductor pipeline SYN-AGT-00 sequence)

### Phase 1 — Discovery & Inventory (Atlas role)
Produce into `/engagement/discovery/`:
1. **`INVENTORY.md`** — every program in `/legacy/cbl` with: program ID, type (batch/online/utility), LOC, copybooks used, files read/written (from FD/SELECT and JCL DD statements), called programs, calling JCL jobs.
2. **`CALL-GRAPH.md`** — static call graph (COBOL `CALL` statements + JCL EXEC PGM steps) as a Mermaid diagram.
3. **`DATA-LINEAGE.md`** — per VSAM/PS dataset: producing programs, consuming programs, record layout copybook, key structure. Anchor copybooks: `CVACT01Y` (account), `CVCUS01Y` (customer), `CVTRA05Y` (transaction), `CVACT03Y` (card xref).
4. **`BRD-CARDDEMO-BATCH.md`** — behavioral requirements document for the batch subsystem, derived *only* from code and data evidence. Every requirement line carries a source citation (program + paragraph + line range). Mark inferred-but-unverified behavior as `[UNVERIFIED]`.

### Phase 2 — Characterization (Validation Agent role, SYN-AGT-14 discipline)
Scope for this engagement: **the 12 batch programs only.** CICS online programs are Phase-2 scope in a later engagement; do not migrate them.

Migration order (dependency-driven):
| Order | Program | Function | Risk notes |
|---|---|---|---|
| 1 | CBACT01C | Account master read/print | Simple sequential read — pipeline shakeout |
| 2 | CBACT02C | Card file read/print | |
| 3 | CBACT03C | Card xref read/print | |
| 4 | CBCUS01C | Customer file read/print | |
| 5 | CBTRN01C | Daily transaction pre-check | Xref lookups, reject handling |
| 6 | CBTRN02C | Transaction posting | **HIGH RISK**: balance arithmetic, reject file, category balance updates |
| 7 | CBACT04C | Interest calculation | **HIGHEST RISK**: COMP-3 arithmetic, rate lookup w/ DEFAULT fallback, monthly interest = balance × rate / 1200, ROUNDED clauses |
| 8 | CBTRN03C | Transaction detail report | Control-break logic, page totals |
| 9 | CBSTM03A/B | Statement generation (calls 03B as subprogram) | Inter-program linkage, 2 output formats |
| 10 | CBEXPORT / CBIMPORT | Data export/import | Encoding-sensitive |

For each program, before any Java is written:
1. Identify input datasets from its JCL job (e.g., `CBACT04C` ← `INTCALC.jcl`... check `/legacy/jcl` for the actual job) and stage the sample data from `/legacy/data/`.
2. Execute the legacy program against sample data to capture golden outputs. Execution options, in preference order: (a) GnuCOBOL (`cobc`) compile-and-run with dialect flags `-std=ibm -fdefaultbyte=0`; document any dialect gaps; (b) where a program cannot run under GnuCOBOL, construct the oracle analytically from code trace + hand-computed expected records, and mark provenance **T3**.
3. Freeze oracle outputs into `/engagement/characterization/oracles/<PROGRAM>/<scenario>/` — byte-exact files plus a `manifest.yaml` (inputs, environment, checksums, date captured). **Oracles are immutable after freeze.** A change to an oracle requires a BEL entry with justification.
4. Write JUnit 5 characterization tests in `/engagement/characterization/tests/` that will later run the Java implementation against the same inputs and diff against the frozen oracle — field-level structured diff, not just byte diff, using record layouts generated from copybooks.

**Scenario minimums per program:** happy path (full sample dataset), empty input file, single-record file, records that trigger every reject/error path visible in the code, and boundary values for every arithmetic field (max PIC digits, zero, negative where sign permits).

### Phase 3 — Target implementation
Maven multi-module under `/target/`:
```
carddemo-parent/
  carddemo-domain/        Records mapped 1:1 from copybooks (Java records, BigDecimal for all COMP-3)
  carddemo-io/            Fixed-length + EBCDIC readers/writers (see §4), VSAM→file abstraction
  carddemo-batch/         One Spring Batch Job per JCL job; one Step per program execution
  carddemo-charz/         Characterization test harness (depends on engagement/characterization)
```
Rules:
- **One JCL job = one Spring Batch `Job`. One program step = one `Step`.** Preserve step sequencing, COND code semantics (map COND to Spring Batch flow decisions), and restart boundaries.
- Copybook → Java mapping is mechanical and documented: every field gets a `@SourceField(copybook="CVACT01Y", field="ACCT-CURR-BAL", pic="S9(10)V99", usage="COMP-3")` style annotation (define this annotation in carddemo-domain) so lineage is machine-auditable.
- All decimal arithmetic uses `BigDecimal` with **explicit scale and RoundingMode chosen to match observed COBOL behavior per statement, not a global default** (see §4.1). Never use `double`/`float` for money.
- No behavioral "improvements." If legacy truncates, target truncates. Modernization of behavior is a separate, human-approved change request — never an agent decision.

### Phase 4 — Differential execution & BEL population
Run every characterization suite: legacy oracle vs. Java output. Every mismatch becomes a BEL entry (schema in `/engagement/bel/BEL-SCHEMA.md`). Classify each as:
- `TARGET-DEFECT` → fix Java, re-run, close with evidence.
- `LEGACY-QUIRK-PRESERVED` → confirm target now reproduces the quirk; document it.
- `ORACLE-GAP` → oracle scenario insufficient; extend scenarios (new freeze, new BEL justification entry).
- `OPEN-DIVERGENCE` → unresolved; blocks program sign-off.

### Phase 5 — Provenance & closeout
- `/engagement/provenance/<PROGRAM>.yaml`: per-paragraph coverage tier — **T1** (executed against frozen oracle), **T2** (covered by structural test, no oracle), **T3** (analytic/static reasoning only). The Ai4 confidence map renders from these files — keep them accurate and machine-readable.
- Engagement Closeout Report per SYN-SCHEMA-ECR: program-by-program equivalence status, open divergences, provenance distribution, oracle manifest index.

## 3. Definition of done — per program
A program is **DONE** only when all are true:
1. Frozen oracle exists with ≥ the scenario minimums in Phase 2.
2. Java implementation passes 100% of characterization tests, byte- or field-exact as declared per output.
3. Zero `OPEN-DIVERGENCE` BEL entries.
4. Provenance file shows ≥80% of paragraphs at T1, none below T3, with every T2/T3 paragraph individually justified.
5. All BEL entries for the program are closed with linked evidence (test run ID + diff artifact).

## 4. COBOL semantic trap catalog — verify explicitly for EVERY program
These are the classes of silent behavioral drift this engagement exists to catch. For each program, produce a checklist in its BEL folder confirming each class was inspected, with line references.

### 4.1 Decimal arithmetic (COMP-3 / packed decimal)
- COBOL `COMPUTE` **without** `ROUNDED` truncates toward zero at the target field's scale. `BigDecimal` default thinking (HALF_UP) is WRONG there → use `RoundingMode.DOWN` at the receiving field's scale. `COMPUTE ... ROUNDED` = HALF_UP.
- Intermediate result precision: COBOL intermediate results follow IBM's rules (effectively high-precision), then truncate/round only on the final MOVE to the receiver. Do not round intermediates in Java.
- CBACT04C's monthly interest (`balance × rate / 1200`) is the flagship case — capture penny-level expected values in the oracle across the full account file and assert the **file total**, not just per-record values, so drift accumulation is visible.

### 4.2 MOVE truncation & padding
- Numeric MOVE to a smaller PIC silently truncates **high-order digits**. Alphanumeric MOVE truncates right and pads with spaces. Reproduce exactly; add characterization scenarios that trigger them where reachable.

### 4.3 Sign handling
- `PIC S9...` COMP-3 signs, unsigned PIC treating negatives as absolute values, and zoned-decimal overpunch signs in the EBCDIC sample data. The io module must decode overpunch correctly; verify against `/legacy/data/EBCDIC/` vs `/legacy/data/ASCII/` for the same logical records.

### 4.4 EBCDIC vs ASCII collation & encoding
- Any SORT (JCL DFSORT steps or COBOL SORT verbs) ordering character keys: EBCDIC orders lowercase < uppercase < digits; ASCII does not. Where sorted order feeds control-break logic (CBTRN03C, CBSTM03A) this changes report grouping and totals. Characterize with keys that expose the difference; target must sort using an explicit EBCDIC collator where equivalence demands it.
- Packed/binary fields in EBCDIC files are NOT charset-convertible — the io module reads them as bytes, never through a charset decoder.

### 4.5 Control flow
- `PERFORM A THRU C` executes physically contiguous paragraphs — map to explicit sequential method calls; flag any GO TO that jumps within a THRU range as a mandatory BEL checklist item.
- Fall-through into following paragraphs at end of a section; `NEXT SENTENCE` vs `CONTINUE` inside nested IFs.

### 4.6 File handling & status codes
- File status checks (or their absence — unchecked statuses that silently continue), AT END handling, and 88-level condition semantics. Empty-file behavior must be characterized for every program (headers/trailers still written? totals of zero? RC?).

### 4.7 Dates & CSUTLDTC
- CSUTLDTC wraps CEEDAYS date validation. Characterize its accept/reject matrix (leap years, month lengths, format masks) and reimplement precisely — do NOT substitute `java.time` lenient/strict parsing without diffing the accept/reject matrix.

## 5. Agent conduct rules
1. `/legacy/**` is read-only evidence. Any need to "fix" legacy code is by definition a misunderstanding — re-read.
2. Never fabricate oracle data. If you cannot execute or fully trace a path, mark it T3/`[UNVERIFIED]` and move on.
3. Every commit message references the program and phase: `[CBACT04C][P2] freeze oracle: happy-path + boundary scenarios`.
4. Small, reviewable increments: one program per PR. A PR mixing oracle changes with target-code changes will be rejected.
5. When legacy behavior looks like a bug (it will), the answer is always: preserve it, ledger it, flag it for human review. Load-bearing bugs are requirements.
6. All secrets/config via environment; no credentials in the repo, ever.

## 6. Build & run quick reference
```bash
# Legacy execution (Phase 2) — GnuCOBOL
sudo apt-get install -y gnucobol            # or brew install gnucobol
cobc -x -std=ibm -o cbact04c legacy/cbl/CBACT04C.cbl -I legacy/cpy

# Target
cd target && mvn -q verify                  # runs characterization suite
mvn -q -pl carddemo-charz test -Dcharz.program=CBACT04C
```

## 7. Ai4 demo artifacts this engagement must yield
1. Populated BEL with at least the divergence classes in §4.1, §4.4, §4.5 caught and evidenced on real runs.
2. Confidence map source data (provenance YAMLs) for all 12 batch programs.
3. One narratable "penny drift" walkthrough: CBACT04C oracle vs. naive HALF_UP implementation vs. corrected DOWN implementation, with the accumulated file-total diff artifact preserved in `/engagement/bel/`.
