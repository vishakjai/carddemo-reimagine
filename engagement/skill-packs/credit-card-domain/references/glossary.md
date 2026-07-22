# CardDemo glossary

| Term | CardDemo meaning and evidence |
|---|---|
| Account | Credit account record (`CVACT01Y`) holding balance, limits, cycle totals, status, dates, ZIP, and group ID. |
| Card cross-reference | Fixed-length relationship (`CVACT03Y`) from 16-character card number to 9-digit customer ID and 11-digit account ID. |
| Transaction master | 350-byte transaction record (`CVTRA05Y`) with type/category, signed amount, merchant, card number, and timestamps. |
| Daily transaction | Incoming batch transaction feed consumed by the posting flow. |
| Reject | Failed incoming record plus validation reason/trailer written by `CBTRN02C`. |
| Transaction-category balance | Account/type/category balance (`CVTRA01Y`) used for downstream interest and fee processing. |
| Disclosure group | Account group/rate lookup used by `CBACT04C`, including a DEFAULT fallback path. |
| Statement | Account/card transaction presentation assembled by the statement job and its `CBSTM03A`/`CBSTM03B` programs. |
