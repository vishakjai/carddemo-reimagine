# Spring Batch implementation checklist

- Prefer deterministic `ItemReader`/processor/`ItemWriter` seams for record
  workloads; use tasklets for utilities and orchestration-only steps.
- Make EBCDIC collation an injected comparator, not the host JVM default.
- Keep record codecs byte-oriented and separately characterized.
- Persist job parameters, execution statuses, and restart checkpoints at the
  same business-safe boundaries as the JCL job.
- Retain reject records with original payload and legacy reason semantics.
- Test empty, single-record, boundary, reject, and full-sample scenarios.
- Compare output record count, order, field values, status/return code, and
  aggregate totals. A green unit test without an oracle comparison is not T1.
