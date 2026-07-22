---
name: java-spring-batch-modernization
description: Java 21, Spring Boot 3, and Spring Batch 5 target rules for behavior-preserving migration of COBOL/JCL/VSAM batch workloads.
skill_version: 1.0.0
maintainer: vishakjai
---

# Java/Spring Batch Modernization

This pack constrains the target design for `SYN-ENG-CARDDEMO-001`. Generated
Java is not proof of modernization; characterization evidence and BEL closure
are the proof.

## Target mapping

- One JCL job becomes one Spring Batch `Job`.
- Each executable program or utility step becomes an explicit `Step`.
- Preserve JCL order, `COND` flow, DD input/output contracts, restart boundaries,
  return statuses, GDG generation intent, and SORT semantics.
- Copybook records become Java records/value types with a source annotation
  carrying copybook, field, PIC, USAGE, offset, length, and scale.
- VSAM/file access sits behind explicit readers/writers; fixed-length byte
  layouts remain testable independently of business processors.

## Numeric rules

- Use `BigDecimal`; never `double`/`float` for COBOL fixed-scale money.
- Apply scale and `RoundingMode` at the same receiving operation where COBOL
  makes the result observable. No global rounding policy.
- `COMPUTE` without `ROUNDED` generally requires truncation toward zero at the
  receiving scale (`RoundingMode.DOWN` for the corresponding `BigDecimal`
  operation). Prove it against the frozen oracle before finalizing.
- Decode display/zoned, binary, and packed-decimal fields according to their
  actual PIC/USAGE. Do not call a display numeric `COMP-3`.

## Control-flow rules

- A `PERFORM A THRU C` maps to explicit sequential calls for every paragraph in
  the physical range, including load-bearing fall-through.
- Translate 88-levels as named predicates and preserve period/conditional scope.
- Model COBOL file status, `AT END`, reject output, return code, and abend
  behavior as observable outcomes—not incidental logging.

## Verification gate

For each migrated program:

1. frozen legacy oracle exists;
2. Java runs the same input scenario;
3. structured and byte-level differences are retained as artifacts;
4. all target defects are fixed and re-run;
5. BEL entry links source span, target span, test, oracle, diff, and reviewer;
6. provenance coverage meets the engagement T1/T2/T3 threshold.

See `references/spring-batch-patterns.md` for the implementation checklist.
