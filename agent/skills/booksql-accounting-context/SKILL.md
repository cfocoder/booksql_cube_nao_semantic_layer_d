---
name: booksql-accounting-context
description: Use when interpreting BookSQL accounting tables and source fields.
---

# BookSQL accounting context — glossary plus D-only text matching

This skill is the D-only Context Layer for BookSQL. It summarizes the dataset's table roles and verified schema, and defines one approved D-only behavior: case-insensitive text matching that preserves exact-versus-substring semantics. It does **not** define accounting policies, derived financial metrics, sign conventions, date cutoffs, or any other business rules. Do not turn a field label into a formula.

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

## Approved D-only case-insensitive text matching

This is the sole D-specific matching rule. Preserve the benchmark's intended exact-versus-substring semantics and use only Cube members returned by live metadata. For substring semantics (for example, an `instr`-style match on `Account`), use Cube's native `contains` filter on the appropriate string member with the user's original text. For exact label equality when casing may differ, first run a bounded candidate-discovery query selecting only that string member and filtering it with `contains` using the original text. Inspect the complete set of distinct returned labels. Use `contains` in the final measure query only if every candidate is an exact case variant of the requested full label; this permits case variants without including longer or partial labels. If there is no exact candidate, any partial/longer candidate, or possible result truncation, report the limitation/ambiguity instead of aggregating a broader set. Cube v1.7.24's SQLite adapter compiles `contains` to `LIKE ... COLLATE NOCASE`. Preserve the original prompt value; do not lowercase it or use physical column names, direct SQL, or invented Cube members. Keep displayed source labels unchanged; this rule does not alter measures, aggregation, joins, dates, or accounting interpretation.

SQLite `NOCASE` is ASCII-oriented; do not assume it handles non-ASCII case variants without validation. If `contains` fails on the active Cube path, report the actual error and stop; do not switch data routes or silently rewrite the filter. Revalidate the adapter behavior if the Cube version or source changes.

This D-only rule can intentionally return a result where the unchanged, case-sensitive gold SQL returns no rows. Preserve the benchmark prompt and gold SQL; report such a result as the D case-insensitive text-matching treatment, not as an exact match to the literal gold execution.

## Interpretation safeguards

- `Amount`, `Credit`, `Debit`, `Open_balance`, entity `Balance`, `Rate`, and `Billing_rate` are distinct stored fields. Their labels alone do not establish equivalence, sign convention, accounting treatment, or a formula. Use the Cube measure definition and question; do not add or net them unless the model/question explicitly supports it.
- `AR_paid`, `AP_paid`, and `Deleted` are source fields. Do not infer their value domain or business meaning beyond what the model exposes.
- The SQLite schema declares composite references from `master_txn_table` to several dimension tables. `businessID` has different declared SQLite types across tables (including `INTEGER` and `TEXT`), and identifier/name fields may not be unique. Do not invent joins or assume one-to-one cardinality; use only joins exposed by Cube and its metadata.
- The physical table is named `products`; the dataset README describes the entity as products/services. Follow the physical Cube model, not a guessed table name.
- Do not carry over Contoso's effective-business-day, customer-recency, customer-placeholder, product-pair, ranking, or monetary-rounding policies. BookSQL has no such policy in this context.

## Layer boundary

This layer adds case-insensitive text matching while preserving each question's exact-versus-substring semantics; otherwise it remains descriptive. It introduces no accounting calculation or exception policy. Preserve original benchmark prompts and gold queries unchanged; document any answer that differs from literal gold execution because of the D-only matching treatment. Freeze the exact D context revision before formal A/B/C/D evaluation.
