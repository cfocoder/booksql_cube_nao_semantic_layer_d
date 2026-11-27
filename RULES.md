# Scenario D rules

This project tests Cube BookSQL plus a BookSQL-specific Context Layer. The D-only layer is the skill `agent/skills/booksql-accounting-context/SKILL.md`; it provides a schema/accounting glossary and an entity-resolution policy for exact, case-insensitive-exact, and validated descriptor-normalized matches while preserving exact-versus-substring semantics. It does not redefine accounting measures.

## Required behavior

1. Use only `cube_semantic` for data access, through `cube_metadata` and `cube_query`. If Cube is unavailable, report the failure; do not fall back to direct SQLite, native SQL, or another data route.
2. Use only measures/dimensions returned by live Cube metadata. Do not invent fields, joins, currencies, sign conventions, accounting formulas, or metrics.
3. Apply the D-only entity-resolution procedure in `agent/skills/booksql-accounting-context/SKILL.md` using live Cube metadata and members. `contains` is for bounded candidate discovery or Gold-intended substring semantics; after resolving an exact entity, use `equals` with the exact stored label or a verified unique identifier. Do not aggregate ambiguous or arbitrary partial candidates.
4. Read and follow `agent/skills/booksql-accounting-context/SKILL.md` when interpreting BookSQL tables, fields, grain, relationships, or entity mentions. Its D-only treatment changes entity-to-member resolution only; it does not change measures, joins, dates, or accounting treatment.
5. Preserve source and Cube-returned values. Do not round source rows or silently apply accounting adjustments. For derived arithmetic, use `decimal_calculator.calculate` on values returned by Cube; keep intermediates unrounded and apply display rounding only when the prompt requests it.
6. Report the Cube semantic route, relevant filters, and time grain. If no supported match or definition is available, state the limitation rather than guessing.
7. Preserve all benchmark prompts and gold queries unchanged. If D's entity resolution or case-insensitive matching differs from literal Gold execution, label it as a D treatment effect rather than an exact Gold match. Freeze this D context revision before formal evaluation.

## Arithmetic boundary

`decimal_calculator.calculate` is permitted only for arithmetic over values already returned by Cube. Cube selects, filters, and aggregates data; the calculator evaluates formulas. Do not send raw rows to the calculator for database aggregation.

## Shared BookSQL benchmark temporal reference

For BookSQL benchmark questions, use `2022-06-01` as the fixed `as_of_date`, matching the date used by the official evaluator for its supported temporal replacements. This does not mean every date/time SQL expression is normalized. The anchor overrides the actual system/database clock only for interpreting temporal expressions in this benchmark.

Use this date only to interpret relative time wording or query predicates. It is not a data cutoff: do not add a blanket `transaction_date <= '2022-06-01'`, do not suppress later transactions, and do not add a date filter when the question has no temporal condition. Preserve the original questions, Gold SQL, and source data.

When forming a query for a relative period, resolve it to explicit absolute ISO dates from this anchor. Preserve the question's intended period and the Gold predicate's boundary semantics; distinguish a full calendar period (for example, “this month”) from a period-to-date (for example, “month to date”). For Cube, use absolute date-range endpoints rather than runtime-relative ranges. For SQL, do not rely on `CURRENT_DATE` or `now` at query time. If an evaluator must normalize unchanged Gold SQL, normalize a separate execution copy and use the same anchor for Gold and predictions. Temporal normalization must not alter text literals or other query semantics.

This is a date-only reference; it does not define a time of day or timezone for `CURRENT_TIMESTAMP`/`now`. Do not invent those values unless the benchmark protocol separately specifies them.

### Fiscal-year interpretation

For this BookSQL benchmark, the fiscal year runs from April 1 through March 31. With `as_of_date = 2022-06-01`, interpret “this fiscal year” and “this fiscal year to date” as 2022-04-01 through 2022-06-01, inclusive. “Last fiscal year” means 2021-04-01 through 2022-03-31, inclusive. Only an explicit request for the full FY 2022–23 uses 2022-04-01 through 2023-03-31.

In the validation split, 150 Gold queries worded “this fiscal year to date” instead cover the full FY through 2023-03-31. Preserve those questions and Gold SQL unchanged, but flag/report them separately as Gold/wording inconsistencies. Do not reproduce that full-year range when answering a genuine “to date” question or silently rewrite Gold.

## First-turn response requirement
Always answer the user's question in the current response and provide every requested field; if data is unavailable, state that explicitly without inventing values or deferring the answer to a follow-up.
