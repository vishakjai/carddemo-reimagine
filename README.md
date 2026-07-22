# CardDemo Reimagine — Synthetix Legacy Modernization Engagement

Reference engagement migrating **AWS CardDemo** (COBOL / VSAM / JCL mainframe credit card application)
to **Java 21 / Spring Batch 5** under the Synthetix Reimagine methodology: characterization-first,
evidence-ledgered, human-governed.

> **Autonomy, governed. See the evidence, not the demo.**

## What's here
| Path | Contents |
|---|---|
| `legacy/` | AWS CardDemo source (read-only): 31 COBOL programs, 30 data copybooks, 17 BMS symbolic-map copybooks, 38 JCL jobs, CICS maps, and paired EBCDIC/ASCII sample data |
| `engagement/` | Discovery outputs, frozen characterization oracles, Behavioral Equivalence Ledger (BEL), coverage provenance manifests |
| `target/` | Java/Spring Batch implementation (Maven multi-module) |
| `CLAUDE.md` | **Start here** — full IDE agent instructions for executing the engagement |

The inventory counts above are measured from this repository. Do not substitute
counts from upstream documentation in generated discovery artifacts.

## Scope
Phase 1 targets the 12 batch programs (`CB*`), orchestrated as Spring Batch jobs mirroring the
legacy JCL. CICS online programs (`CO*`) are a subsequent engagement.

## Attribution
`legacy/` contains the AWS CardDemo application, © Amazon.com Inc. or its affiliates,
licensed under Apache License 2.0 (see `LICENSE` and `NOTICE`). Original repo:
https://github.com/aws-samples/aws-mainframe-modernization-carddemo
