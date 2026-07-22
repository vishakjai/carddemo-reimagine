# COBOL semantic traps to inspect explicitly

For every in-scope program, report inspected/present/not-present plus source
anchors for each class below.

## Decimal semantics

- Distinguish display numeric, zoned decimal, binary `COMP`, and packed
  `COMP-3`; never use “packed decimal” as a synonym for all COBOL numerics.
- A fixed-scale receiver controls the observable result. Without `ROUNDED`,
  discarded fractional digits are truncated toward zero; `ROUNDED` changes the
  rule. Preserve statement-level semantics rather than applying one Java
  `RoundingMode` globally.
- Assert batch/file totals as well as record values so repeated one-cent drift
  becomes visible.

## MOVE, signs, and layouts

- Numeric `MOVE` aligns decimal points and may silently discard high-order or
  fractional digits to fit the receiver.
- Alphanumeric `MOVE` truncates on the right or space-pads the receiver.
- Decode signed display overpunch and packed signs from bytes. Do not pass a
  whole record through a character decoder when non-text fields are present.
- `REDEFINES` is an alternate interpretation of the same bytes, not another
  stored field.

## Control flow

- Expand `PERFORM A THRU C` by physical paragraph order.
- Check fall-through, `GO TO` into/out of ranges, `NEXT SENTENCE` versus
  `CONTINUE`, and period scope.
- Treat 88-level condition names as predicates over their parent storage.

## Files and orchestration

- JCL step order, `COND`, DD bindings, DISP, GDG generation, SORT control cards,
  restart points, and return codes are part of the application.
- Preserve empty-input behavior, headers/trailers, end-of-file state, unchecked
  file statuses, reject records, and partial side effects.
- Character collation must be explicit when EBCDIC-sorted output drives
  control-break or statement logic.
