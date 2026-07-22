# Evidence contract

Every behavioral assertion emitted by the Code Archaeologist needs:

- `program` and `paragraph`;
- source file and exact line range;
- participating fields with copybook and PIC/USAGE;
- JCL job/step and DD/dataset bindings when I/O is involved;
- observable preconditions, outputs, side effects, status/return behavior;
- provenance tier: `T1` executed against frozen oracle, `T2` structural test,
  or `T3` analytic/static reasoning;
- confidence and a statement of what remains unverified.

The absence of an executable legacy runtime does not permit invented behavior.
Static conclusions remain T3 until a frozen oracle or structural test raises
their tier. Oracles are immutable after freeze. A legacy quirk is a requirement
until a human approves a behavior change.

The Code Archaeology Brief must expose, at minimum:

1. program/JCL/copybook/dataset inventory;
2. dependency and call graph;
3. data lineage by dataset and record layout;
4. batch business capabilities and rules;
5. semantic-trap checklist and risk hotspots;
6. evidence gaps and characterization backlog.
