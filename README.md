# Scenario D — Cube BookSQL plus descriptive accounting context

This repository is the Nao Multi-project context for BookSQL scenario **D**.

## Boundary

- No direct SQLite or native database connector is declared.
- BookSQL data is accessed only through `cube_semantic` and the dedicated Cube BookSQL MCP.
- The D-only `booksql-accounting-context` skill preserves evidence-based Entity Resolution and distinguishes customer-payment events from payment-method questions. It also routes accounting intent to explicit measures in the shared C/D semantic model, without inventing joins or formulas.
- `decimal_calculator.calculate` is arithmetic-only over values already returned by Cube.
- The context explicitly avoids Contoso rules and unsupported assumptions about Amount/Credit/Debit, joins, or business behavior.
- For exact entity filters, D uses a documented normalized name member with `equals` when live metadata exposes it, otherwise resolves a full stored label through bounded candidate discovery and exact `equals`. `contains` is used for candidate discovery, not as an equality substitute; intentional benchmark substring filters still use it as their final operator. The only descriptor initially enabled is a trailing `partnership`; other titles and legal suffixes remain disabled pending validation.

## Project mapping

- Nao project: `booksql-condition-d`
- Repository: `cfocoder/booksql_cube_nao_semantic_layer_d`
- Expected route: `cube_semantic` → private Cube BookSQL MCP → Cube Core → SQLite, plus the D-only glossary skill

## Entity Resolution and regression matrix

The D skill maps a natural-language entity phrase to a live Cube member without changing the question, Gold SQL, semantic model, MCP, or SQLite source. Resolution order is exact `equals`, full-label case-insensitive exact match, then normalization of an allowlisted trailing descriptor. Validation Gold shows the descriptor convention for `partnership` in 471/471 cases in the validation split; no other descriptor is enabled. The final exact-entity filter uses `equals` with the stored label or a verified unique identifier. `contains` remains the final filter only for questions whose Gold semantics are intentionally substring-based.

Regression checks (use validation examples and do not expose names or expected answers in this README):

- **Exact:** an exact stored label resolves without normalization and the final query uses `equals`.
- **Case-only:** a full-label candidate differing only in case resolves under the existing D policy; the final query uses the stored spelling with `equals`.
- **Descriptor:** a natural-language entity phrase ending in the validated descriptor `partnership` is normalized only after exact/case-only lookup fails; the normalized full label must resolve uniquely, then the final query uses `equals`.
- **Ambiguous/partial:** longer or partial labels are not treated as equal; multiple plausible entities, unresolved identity, or possibly truncated candidates cause an abstention/clarification rather than aggregation.
- **Unvalidated suffix:** titles and other legal suffixes (including `Inc.`, `LLC`, `Ltd.`, and `S.A. de C.V.`) are preserved and not stripped.

## Shared measure routing and completeness

- **Outstanding invoices:** transaction grain, `unpaidInvoices` segment (`Transaction_TYPE = invoice` and `Open_balance > 0`); do not substitute `AR_paid` when it conflicts.
- **Expense:** `BookSQLClassifiedAccountingLines.classifiedExpenseDebit`, positive debit on unambiguously classified expense accounts; not transaction `Amount`.
- **Recognized revenue:** `recognizedRevenueCredit`, positive credit on unambiguously classified income accounts; not invoice `Amount`.
- **Invoice value:** `BookSQLTransactionAnalytics.totalInvoiceAmount`, once per invoice transaction; keep distinct from recognized revenue.
- **Ambiguous account mapping:** inspect `accountClassificationIssueLineCount` and unresolved debit/credit in the same slice. Do not call a classified subtotal complete if unresolved lines could affect it.
- **Comparison completeness:** identify top-N limits; never present a capped ranking as the full comparison.

These are semantic definitions shared by C and D; the D Context Layer only guides member selection. Original prompts and official Gold queries remain unchanged.

Langfuse audit: for every entity lookup, record the original phrase, candidate labels actually returned, normalization (`none` if none), selected stored label/ID (`none` if unresolved), method, and uniqueness/ambiguity. Tool inputs/results plus this concise final-response note provide the evidence; a candidate-discovery call alone does not prove the final transaction query executed.

### Payment-event interpretation

For an unambiguous question about receiving a payment from a customer, D treats payment as an event-type filter: use the live Cube transaction-type member and an exact `equals` filter for its confirmed stored `payment` label, in addition to the independently resolved customer and requested dates. In dev Gold, all 471 questions in the “receive payment from” wording class filter `transaction_type = 'payment'`. This is not a rule for every occurrence of “payment”: all 606 dev Gold questions in the “payment method” wording class use `transaction_type = 'sales receipt'` and ask about payment method. Do not infer event type from amount or `AR_paid`/`AP_paid`; do not generalize this mapping to vendor payments without validation.

Additional regression checks:

- **Received-payment event:** retain the entity-resolution result and date filters, add the confirmed exact transaction-type filter, and establish existence from matching transaction count/IDs.
- **Payment method:** distinguish the requested payment-method attribute from a payment event; do not apply `transactionType = payment` just because “payment” appears in the phrase.
- **Unclear direction/type:** do not infer whether a payment was received from a customer or made to a vendor; use only supported metadata and question context, otherwise abstain/clarify.

## Methodology status

Hector approved D-only entity resolution that preserves exact-versus-substring semantics and initially normalizes only the validated trailing descriptor `partnership`. Shared semantic measures distinguish outstanding invoices, expense debits, income credits, and invoice totals; the payment-event rule remains limited to supported customer-received wording. Original prompts and Gold SQL remain unchanged. If entity resolution, metric/grain choice, classification, or event filtering changes a result relative to literal Gold execution, record it as a D treatment effect, not an exact Gold match. Freeze this context before formal A/B/C/D runs.
