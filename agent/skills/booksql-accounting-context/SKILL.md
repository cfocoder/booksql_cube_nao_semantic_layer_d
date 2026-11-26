---
name: booksql-accounting-context
description: Use when interpreting BookSQL accounting tables and source fields.
---

# BookSQL accounting context — glossary plus D-only text matching

This skill is the D-only Context Layer for BookSQL. It summarizes the dataset's table roles and verified schema, and defines one approved D-only behavior: case-insensitive matching for user-provided text labels. It does **not** define accounting policies, derived financial metrics, sign conventions, date cutoffs, or any other business rules. Do not turn a field label into a formula.

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

## Approved D-only case-insensitive matching

This is the sole D-specific matching rule. When a prompt provides a textual label for a Cube string dimension (for example vendor, customer, account, product/service, or payment method), normalize only the letter case of the user-provided value to lowercase, preserve its spaces and punctuation, and use an exact `equals` filter on the semantically appropriate Cube member returned by live metadata. Do not use physical column names or direct SQL in D. Keep displayed source labels unchanged; this rule changes only how the filter value is matched, not the measure, aggregation, join, date, or accounting interpretation.

The relevant label fields in the current BookSQL SQLite data were checked and are stored in lowercase. On the current Cube v1.7.24 + SQLite path, the Cube `contains` filter generated `ILIKE` and failed with a SQLite syntax error, so do not use `contains` as a case-insensitive workaround. If exact equality with the normalized value returns no rows, report that; do not fall back to another data route or broaden the filter silently. If a future source contains mixed-case stored labels, revalidate this policy rather than assuming lowercasing the prompt alone is sufficient.

This D-only rule can intentionally return a result where the unchanged, case-sensitive gold SQL returns no rows. Preserve the benchmark prompt and gold SQL; report such a result as the D case-insensitive treatment, not as an exact match to the literal gold execution.

## Interpretation safeguards

- `Amount`, `Credit`, `Debit`, `Open_balance`, entity `Balance`, `Rate`, and `Billing_rate` are distinct stored fields. Their labels alone do not establish equivalence, sign convention, accounting treatment, or a formula. Use the Cube measure definition and question; do not add or net them unless the model/question explicitly supports it.
- `AR_paid`, `AP_paid`, and `Deleted` are source fields. Do not infer their value domain or business meaning beyond what the model exposes.
- The SQLite schema declares composite references from `master_txn_table` to several dimension tables. `businessID` has different declared SQLite types across tables (including `INTEGER` and `TEXT`), and identifier/name fields may not be unique. Do not invent joins or assume one-to-one cardinality; use only joins exposed by Cube and its metadata.
- The physical table is named `products`; the dataset README describes the entity as products/services. Follow the physical Cube model, not a guessed table name.
- Do not carry over Contoso's effective-business-day, customer-recency, customer-placeholder, product-pair, ranking, or monetary-rounding policies. BookSQL has no such policy in this context.

## Layer boundary

This layer adds case-insensitive matching of text labels and otherwise remains descriptive. It introduces no accounting calculation or exception policy. Preserve original benchmark prompts and gold queries unchanged; document any answer that differs from literal gold execution because of case normalization. Freeze the exact D context revision before formal A/B/C/D evaluation.
