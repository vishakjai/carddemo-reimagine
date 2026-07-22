# Card data and control risks

- Card numbers, SSNs, government-issued identifiers, dates of birth, FICO
  scores, addresses, merchant data, and account balances require explicit data
  classification in modern target models.
- Keep source evidence separate from compliance conclusions. PCI scope and
  retention decisions require human/security review; this pack only identifies
  likely sensitive fields and flows.
- Preserve fixed-length record boundaries, filler, masking rules, and
  cross-reference keys during characterization.
- Tests and demo artifacts must use the supplied public sample data or synthetic
  data only. Never add real cardholder data.
