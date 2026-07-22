# CardDemo batch topology anchors

The in-scope batch chain contains twelve programs:

`CBACT01C`, `CBACT02C`, `CBACT03C`, `CBCUS01C`, `CBTRN01C`,
`CBTRN02C`, `CBACT04C`, `CBTRN03C`, `CBSTM03A`, `CBSTM03B`,
`CBEXPORT`, and `CBIMPORT`.

High-value evidence joins:

- `POSTTRAN.jcl` → `CBTRN02C`: daily transactions, transaction master, card
  cross-reference, rejects, accounts, and transaction-category balances.
- `INTCALC.jcl` → `CBACT04C`: category balances, card cross-reference,
  accounts, disclosure groups/rates, and generated interest transactions.
- `CREASTMT.JCL` → SORT → `CBSTM03A`/`CBSTM03B`: transaction ordering,
  keyed lookups, customer/account enrichment, and statement outputs.
- `TRANREPT.jcl`/`TRANREPT.prc` → SORT → `CBTRN03C`: report ordering and
  control-break behavior.

Do not infer these relationships from filenames alone. Cite `EXEC PGM`, DD
names, `SELECT`/`FD`, `CALL`, and paragraph-level I/O statements.
