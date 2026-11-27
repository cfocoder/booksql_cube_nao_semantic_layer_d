---
name: booksql-accounting-context
description: Use when interpreting BookSQL accounting tables and source fields.
---

# BookSQL accounting context — glossary plus D-only entity resolution

This skill is the D-only Context Layer for BookSQL. It summarizes the dataset's table roles and verified schema, and defines one approved D-only behavior: evidence-based entity resolution that preserves exact-versus-substring semantics, including the previously validated case-insensitive treatment. It does **not** define accounting policies, derived financial metrics, sign conventions, date cutoffs, or any other business rules. For benchmark temporal and fiscal-year interpretation, follow the shared policy in the project's `RULES.md`. Do not turn a field label into a formula.

## Data source and grain

BookSQL's `accounting.sqlite` contains records for multiple businesses. The physical `master_txn_table` is at accounting-line grain: a `(businessID, Transaction_ID)` can appear on multiple rows. In the current Cube model, `Transactions` represents source accounting lines and `AccountingTransactions` represents one row per composite transaction key. Use live Cube metadata to select the appropriate grain and measure. The transaction-grain cube deduplicates `Amount` and `Open_balance`, and sums `Quantity` within each transaction. Do not describe the physical table as one row per transaction.

## Physical tables

- `master_txn_table`: transaction rows. Fields include `businessID`, `Transaction_ID`, `Transaction_DATE`, `Transaction_TYPE`, `Amount`, `CreatedDATE`, `CreatedUSER`, `Account`, `AR_paid`, `AP_paid`, `Due_DATE`, `Open_balance`, `Customers`, `Vendor`, `Product_Service`, `Quantity`, `Rate`, `Credit`, `Debit`, `payment_method`, and `Misc`.
- `chart_of_accounts`: account names and account-type labels (`Account_name`, `Account_type`) by business.
- `customers`: customer names, billing/shipping attributes, and source `Balance`.
- `vendors`: vendor names, billing attributes, and source `Balance`.
- `employees`: employee name/ID, `Hire_date`, `Billing_rate`, and `Deleted` fields.
- `products`: product/service names and `Product_Service_type`.
- `payment_method`: payment-method labels and `Credit_card` field.

Use these exact physical names only in B's SQLite MCP. In D, use only the logical members exposed by Cube metadata.

## D-only entity resolution and text matching

Apply this procedure only to entity labels that the question asks to match. Preserve the original entity phrase, question, and Gold SQL. Use live Cube metadata; never invent members, IDs, relationships, or entity-type fields. Determine whether the question refers to a customer or vendor from its wording and context, and search the corresponding Cube member. If the role is unclear, inspect customer and vendor candidates separately and do not merge them.

### Resolution order

1. **Exact label first.** Query the relevant member with Cube `equals` using the full entity phrase as written. If one stored label or verified unique identifier matches, use it.
2. **Case-only variation second.** If no exact match exists, run a bounded candidate-discovery query with `contains` on the relevant member. Compare each complete stored label—not just the matching substring—to the full requested entity phrase case-insensitively. Accept only exact case variants of the full phrase. For the final equality query, use `equals` with the exact stored label(s) returned by Cube, or a verified unique identifier; do not use `contains` as a substitute for equality.
3. **Validated descriptor normalization third.** Only after exact and case-only resolution fail, check the explicit descriptor allowlist below. Initially, `partnership` is the only enabled descriptor. Remove it only when it is a trailing descriptor in the natural-language entity phrase, not when it is part of a quoted/stored-name request or another filter. Preserve both the original phrase and normalized base name. Search candidates with `contains`, but retain only complete stored labels that equal the normalized base name ignoring case; longer/partial labels are never eligible. Resolve only if the retained values represent one logical entity under the existing case-only policy and no live identifier/context conflict exists. Use `equals` with the selected stored label(s) or verified unique identifier in the final query. Treat `partnership` as a phrase descriptor, not an assumed Cube dimension or filter.
4. **All other descriptors remain disabled.** Do not strip titles or legal-form suffixes such as `Inc.`, `LLC`, `Ltd.`, or `S.A. de C.V.` until the same normalization is independently validated against the benchmark and explicitly added to the allowlist. Preserve them as part of the phrase or report the entity as unresolved.

The validation split supports the `partnership` rule: all 471 questions containing that trailing descriptor have Gold customer filters using the preceding name without the descriptor. This is benchmark-wide evidence, not a customer-specific exception. Keep the rule limited to that terminal descriptor until further validation.

### Candidate safety and final filters

- `contains` is for candidate discovery; it is also allowed in the final query only when the benchmark question/Gold has intentional substring semantics (for example, `instr` or `LIKE '%phrase%'`). Preserve that distinction.
- Do not accept a longer or partial label as an exact entity match. After descriptor normalization, compare the full candidate label to the entire normalized name. Never combine distinct labels just because they share a substring.
- If multiple plausible entities remain, disambiguate only with a unique identifier or other context actually exposed by Cube and supported by the question. If they remain ambiguous, or the candidate set may be truncated (including hitting the result cap), stop and report/ask rather than guessing or aggregating. A unique full-label match can be used even when no identifier is exposed.
- If Cube exposes a stable entity identifier or `businessID` needed to distinguish otherwise identical labels, use only the live member returned by metadata. Do not assume `customerName` or `vendorName` is globally unique.
- Use Cube's literal stored label in final `equals` filters; do not lowercase source values, write raw SQL, switch data routes, or claim a particular SQL translation without query-log evidence.
- Do not assume non-ASCII case variants behave identically to the tested labels; validate before claiming broader Unicode case-folding support. If candidate discovery fails, report the actual tool error and stop.

### Langfuse resolution audit

For every entity lookup, include a short, factual audit in the final response so Langfuse captures the resolution alongside the `cube_query` observations: original phrase, candidate labels actually returned, normalization (`none` if none), selected stored label/identifier (`none` if unresolved), method (`exact`, `case-insensitive exact`, `descriptor-normalized exact`, or `ambiguous`), and whether resolution was unique. Do not expose hidden reasoning, invent candidates, or claim a final query ran unless Langfuse shows its final filter and returned result.

## Interpretation safeguards

- `Amount`, `Credit`, `Debit`, `Open_balance`, entity `Balance`, `Rate`, and `Billing_rate` are distinct stored fields. Their labels alone do not establish equivalence, sign convention, accounting treatment, or a formula. Use the Cube measure definition and question; do not add or net them unless the model/question explicitly supports it.
- `AR_paid`, `AP_paid`, and `Deleted` are source fields. Do not infer their value domain or business meaning beyond what the model exposes.
- The SQLite schema declares composite references from `master_txn_table` to several dimension tables. `businessID` has different declared SQLite types across tables (including `INTEGER` and `TEXT`), and identifier/name fields may not be unique. Do not invent joins or assume one-to-one cardinality; use only joins exposed by Cube and its metadata.
- The physical table is named `products`; the dataset README describes the entity as products/services. Follow the physical Cube model, not a guessed table name.
- Do not carry over Contoso's effective-business-day, customer-recency, customer-placeholder, product-pair, ranking, or monetary-rounding policies. BookSQL has no such policy in this context.

## Layer boundary

This layer adds evidence-based entity resolution while preserving each question's exact-versus-substring semantics; otherwise it remains descriptive. It introduces no accounting calculation or exception policy. Preserve original benchmark prompts and Gold queries unchanged; document any answer that differs from literal Gold execution because of D-only entity resolution or case handling. Freeze the exact D context revision before formal A/B/C/D evaluation.
