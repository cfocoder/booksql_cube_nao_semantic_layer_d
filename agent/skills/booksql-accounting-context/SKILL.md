---
name: booksql-accounting-context
description: Use when interpreting BookSQL accounting tables and source fields.
---

# BookSQL accounting context — descriptive glossary only

This skill is the proposed D-only Context Layer for BookSQL. It summarizes the dataset's documented table roles and the verified SQLite schema. It does **not** define accounting policy, derived metrics, sign conventions, date cutoffs, or business rules. Do not turn a field label into a formula.

## Data source and grain

BookSQL's `accounting.sqlite` contains records for multiple businesses. `master_txn_table` is the transaction-level table documented by the dataset README as the main table of business transaction records. Use Cube metadata and the question to select measures and dimensions. The skill does not add a filter, join, aggregation, or adjustment.

## Physical tables

- `master_txn_table`: transaction rows. Fields include `businessID`, `Transaction_ID`, `Transaction_DATE`, `Transaction_TYPE`, `Amount`, `CreatedDATE`, `CreatedUSER`, `Account`, `AR_paid`, `AP_paid`, `Due_DATE`, `Open_balance`, `Customers`, `Vendor`, `Product_Service`, `Quantity`, `Rate`, `Credit`, `Debit`, `payment_method`, and `Misc`.
- `chart_of_accounts`: account names and account-type labels (`Account_name`, `Account_type`) by business.
- `customers`: customer names, billing/shipping attributes, and source `Balance`.
- `vendors`: vendor names, billing attributes, and source `Balance`.
- `employees`: employee name/ID, `Hire_date`, `Billing_rate`, and `Deleted` fields.
- `products`: product/service names and `Product_Service_type`.
- `payment_method`: payment-method labels and `Credit_card` field.

Use these exact physical names only in B's SQLite MCP. In D, use only the logical members exposed by Cube metadata.

## Interpretation safeguards

- `Amount`, `Credit`, `Debit`, `Open_balance`, entity `Balance`, `Rate`, and `Billing_rate` are distinct stored fields. Their labels alone do not establish equivalence, sign convention, accounting treatment, or a formula. Use the Cube measure definition and question; do not add or net them unless the model/question explicitly supports it.
- `AR_paid`, `AP_paid`, and `Deleted` are source fields. Do not infer their value domain or business meaning beyond what the model exposes.
- The SQLite schema declares composite references from `master_txn_table` to several dimension tables. `businessID` has different declared SQLite types across tables (including `INTEGER` and `TEXT`), and identifier/name fields may not be unique. Do not invent joins or assume one-to-one cardinality; use only joins exposed by Cube and its metadata.
- The physical table is named `products`; the dataset README describes the entity as products/services. Follow the physical Cube model, not a guessed table name.
- Do not carry over Contoso's effective-business-day, customer-recency, customer-placeholder, product-pair, ranking, or monetary-rounding policies. BookSQL has no such policy in this context.

## Layer boundary

This is a descriptive schema/domain glossary only. It introduces no business calculation or exception policy. Preserve original benchmark prompts and gold queries unchanged. This draft should be reviewed and frozen by the researcher before formal A/B/C/D evaluation.
