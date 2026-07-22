---
name: credit-card-domain
description: Source-grounded credit-card servicing domain language for CardDemo accounts, cards, customers, transaction posting, category balances, interest, statements, rejects, and PCI-sensitive data.
skill_version: 1.0.0
maintainer: vishakjai
---

# Credit Card Domain — CardDemo

Use this pack to interpret CardDemo mechanics in business language without
inventing product policies. Repository code and copybooks remain the authority.

## Core aggregates and relationships

- **Customer** — identity and contact record in `CVCUS01Y`; contains sensitive
  SSN/government ID and FICO fields that increase data-handling risk.
- **Account** — balance, limits, cycle credits/debits, status, dates, address
  ZIP, and disclosure group in `CVACT01Y`.
- **Card** — a payment credential associated through the card cross-reference
  in `CVACT03Y` (`card → customer → account`). Do not assume one-to-one
  cardinality without evidence from the data and program logic.
- **Transaction** — amount, type, category, merchant, card number, origin time,
  and processing time in `CVTRA05Y`.
- **Transaction-category balance** — per-account/type/category accumulated
  balance in `CVTRA01Y`; feeds interest/fee calculation.

## Business processes to identify

1. Pre-check and reject handling for incoming daily transactions.
2. Posting to transaction master plus account and category-balance effects.
3. Interest-rate lookup by disclosure group with DEFAULT fallback.
4. Interest/fee transaction generation and account-balance update.
5. Transaction sorting, account/customer enrichment, and statement generation.
6. Export/import boundary behavior and encoding preservation.

## Domain interpretation rules

- Separate card number, account ID, and customer ID everywhere. They are not
  interchangeable identifiers.
- A negative or positive signed amount is not enough to infer debit/credit
  semantics; type/category codes and the posting paragraphs supply meaning.
- A “reject” is an observable output record with reason and input payload, not
  merely an exception.
- Interest calculation is servicing behavior and must preserve the legacy rate
  lookup, scale, rounding/truncation, posting date, generated transaction, and
  account update as one traceable contract.
- Statement totals depend on selected inputs, sort order, grouping/control
  breaks, and enrichment lookups.
- Never label CardDemo as a card authorization network or real-time payment
  switch unless source evidence demonstrates those capabilities.

Read `references/glossary.md` and `references/data-risk.md` when producing
capabilities, requirements, data models, security findings, or tests.
