---
name: carddemo-cobol-archaeology
description: Evidence-first comprehension of AWS CardDemo COBOL, JCL, VSAM, copybooks, CICS maps, EBCDIC datasets, paragraph control flow, and COBOL decimal semantics.
skill_version: 1.0.0
maintainer: vishakjai
legacy_skill_id: carddemo_cobol_modernization
legacy_skill_name: CardDemo COBOL Modernization
legacy_skill_maturity: ga
legacy_skill_extensions:
  - ".cbl"
  - ".cob"
  - ".cpy"
  - ".jcl"
  - ".bms"
  - ".prc"
  - ".ctl"
legacy_skill_content_tokens:
  - "tcatbalf"
  - "dalytran"
  - "perform 1000-trnxfile-proc thru 1999-exit"
  - "compute ws-monthly-int"
  - "exec cics"
  - "file status"
  - "redefines"
  - "comp-3"
  - "sort fields="
  - "card-xref-record"
legacy_skill_analysis_focus:
  - "program-to-JCL-step and DD dataset topology"
  - "copybook field layouts, PIC/USAGE semantics, REDEFINES, 88-levels, and record lengths"
  - "paragraph call graph including PERFORM THRU physical ranges and fall-through"
  - "VSAM access modes, alternate indexes, file-status behavior, and reject paths"
  - "fixed-scale arithmetic, receiving-field truncation, ROUNDED, sign handling, and aggregate drift"
  - "EBCDIC versus ASCII collation and mixed text/packed/binary record handling"
legacy_skill_build_kinds:
  - name: ".jcl"
    kind: ibm_jcl_job
    component_kind: mainframe_batch_job
  - name: ".cbl"
    kind: cobol_program
    component_kind: mainframe_program
legacy_skill_language_extensions:
  - ".cbl=COBOL"
  - ".cob=COBOL"
  - ".cpy=COBOL Copybook"
  - ".jcl=JCL"
  - ".bms=BMS"
  - ".prc=JCL Procedure"
  - ".ctl=Mainframe Control Card"
legacy_skill_archetypes:
  - archetype: cobol_jcl_vsam_batch
    description: "COBOL batch estate orchestrated by JCL over fixed-length and VSAM datasets."
    confidence: 0.99
    match_all_extensions: [".cbl", ".cpy", ".jcl"]
legacy_skill_scanner_ignore_paths:
  - ".git"
  - "target"
---

# CardDemo COBOL Archaeology

This pack governs source understanding for `SYN-ENG-CARDDEMO-001`. The Code
Archaeologist must prove behavior from the repository and must not convert
generic COBOL knowledge into unsupported CardDemo claims.

## Required analysis order

1. Inventory all 31 programs, then separate the 12 `CB*` batch programs from
   the CICS `CO*` programs and the `CSUTLDTC` utility.
2. For each batch program, join its `PROGRAM-ID` to every `EXEC PGM=` JCL step.
3. Resolve each JCL DD name to the matching COBOL `SELECT`/`FD`, dataset,
   organization, access mode, record length, and copybook.
4. Expand copybooks before interpreting the Procedure Division. Record every
   PIC, `USAGE`, sign, implied decimal, `REDEFINES`, 88-level, and filler span
   that affects observable behavior.
5. Build paragraph ranges from physical source order. A `PERFORM A THRU C`
   executes every paragraph in that range; do not model it as a call to `A`
   alone.
6. Trace all reads, writes, rewrites, reject outputs, status checks, `AT END`
   behavior, return codes, and abend paths.
7. Treat the original EBCDIC files as bytes. Decode text fields by layout;
   never charset-convert packed, zoned-sign, or binary fields wholesale.
8. Emit business rules with file, paragraph, and line-range citations. If a
   branch cannot be executed or fully traced, label it `T3`/`UNVERIFIED`.

## Mandatory CardDemo findings

- `CBACT04C` lines 462–467 computes monthly interest into
  `WS-MONTHLY-INT PIC S9(09)V99` without `ROUNDED`. The receiving field's scale
  is the risk. This case is not `COMP-3` and must not be described as such.
- `INTCALC.jcl` lines 22–41 proves the program's five input contracts and its
  350-byte system-transaction output.
- `CBSTM03B` lines 120–126 contains four `PERFORM ... THRU` ranges whose
  intermediate paragraphs are load-bearing.
- `CREASTMT.JCL` lines 44–61 sorts by card number and transaction ID before
  statement processing; collation is part of the behavior.
- `CBTRN02C` plus `POSTTRAN.jcl` defines validation, reject-file, account, and
  transaction-category balance side effects. A happy-path-only summary is
  insufficient.

Read `references/semantic-traps.md`, `references/evidence-contract.md`, and
`references/batch-topology.md` before producing the Code Archaeology Brief.
