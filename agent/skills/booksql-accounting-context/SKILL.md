---
name: booksql-accounting-context
description: Use when interpreting BookSQL accounting tables and source fields.
---

# BookSQL accounting context — glossary plus D-only text matching

This skill is the D-only Context Layer for BookSQL. It summarizes the dataset's table roles and verified schema, and defines one approved D-only behavior: case-insensitive substring matching where the question semantics require it. It does **not** define accounting policies, derived financial metrics, sign conventions, date cutoffs, or any other business rules. Do not turn a field label into a formula.

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

## Approved D-only case-insensitive substring matching

This is the sole D-specific matching rule. When the question requires substring semantics for a text label (for example, the `instr`-style match on `Account` in the Street vending question), use Cube's native `contains` filter on the semantically appropriate string member returned by live metadata. Preserve the user's original text, including case, spaces, and punctuation; do not lowercase it first. In the deployed Cube v1.7.24 SQLite adapter, `contains` compiles to SQLite `LIKE ... COLLATE NOCASE`, equivalent to a case-insensitive `%value%` substring search. Do not use physical column names, direct SQL, or invented Cube members in D. Keep displayed source labels unchanged; this rule changes only text-filter semantics, not the measure, aggregation, join, date, or accounting interpretation.

Use `contains` only when substring matching is intended; it can match more than one full label and must not silently replace exact equality. If exact, case-insensitive equality is required but the active Cube model exposes no appropriate case-insensitive exact filter, report that limitation instead of broadening the query. If `contains` fails on the active Cube path, report the actual error and stop; do not switch data routes or silently rewrite the filter. Revalidate the adapter behavior if the Cube version or source changes.

This D-only rule can intentionally return a result where the unchanged, case-sensitive gold SQL returns no rows. Preserve the benchmark prompt and gold SQL; report such a result as the D case-insensitive substring treatment, not as an exact match to the literal gold execution.

## Interpretation safeguards

- `Amount`, `Credit`, `Debit`, `Open_balance`, entity `Balance`, `Rate`, and `Billing_rate` are distinct stored fields. Their labels alone do not establish equivalence, sign convention, accounting treatment, or a formula. Use the Cube measure definition and question; do not add or net them unless the model/question explicitly supports it.
- `AR_paid`, `AP_paid`, and `Deleted` are source fields. Do not infer their value domain or business meaning beyond what the model exposes.
- The SQLite schema declares composite references from `master_txn_table` to several dimension tables. `businessID` has different declared SQLite types across tables (including `INTEGER` and `TEXT`), and identifier/name fields may not be unique. Do not invent joins or assume one-to-one cardinality; use only joins exposed by Cube and its metadata.
- The physical table is named `products`; the dataset README describes the entity as products/services. Follow the physical Cube model, not a guessed table name.
- Do not carry over Contoso's effective-business-day, customer-recency, customer-placeholder, product-pair, ranking, or monetary-rounding policies. BookSQL has no such policy in this context.

## Layer boundary

This layer adds case-insensitive substring matching only where the question semantics require it and otherwise remains descriptive. It introduces no accounting calculation or exception policy. Preserve original benchmark prompts and gold queries unchanged; document any answer that differs from literal gold execution because of the D-only matching treatment. Freeze the exact D context revision before formal A/B/C/D evaluation.
