# Scenario D rules

This project tests Cube BookSQL plus a BookSQL-specific descriptive Context Layer. The D-only layer is the skill `agent/skills/booksql-accounting-context/SKILL.md`; it is a schema/accounting glossary, not a policy engine.

## Required behavior

1. Use only `cube_semantic` for data access, through `cube_metadata` and `cube_query`. If Cube is unavailable, report the failure; do not fall back to direct SQLite, native SQL, or another data route.
2. Use only measures/dimensions returned by live Cube metadata. Do not invent fields, joins, currencies, sign conventions, business rules, or metrics.
3. Read and follow `agent/skills/booksql-accounting-context/SKILL.md` when interpreting BookSQL tables, fields, or relationships. The skill provides descriptive context only; it does not change a numerator, denominator, filter, date cutoff, join, or source result.
4. Preserve source and Cube-returned values. Do not round source rows or silently apply accounting adjustments. For derived arithmetic, use `decimal_calculator.calculate` on values returned by Cube; keep intermediates unrounded and apply display rounding only when the prompt requests it.
5. Report the Cube semantic route and relevant filters/time grain. If a requested definition or result is not available through Cube, state the limitation rather than guessing.
6. Preserve all benchmark prompts and gold queries unchanged. This proposed D treatment must be reviewed and frozen before any formal evaluation.

## Arithmetic boundary

`decimal_calculator.calculate` is permitted only for arithmetic over values already returned by Cube. Cube selects, filters, and aggregates data; the calculator evaluates formulas. Do not send raw rows to the calculator for database aggregation.

## First-turn response requirement
Always answer the user's question in the current response and provide every requested field; if data is unavailable, state that explicitly without inventing values or deferring the answer to a follow-up.
