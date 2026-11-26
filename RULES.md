# Scenario D rules

This project tests Cube BookSQL plus a BookSQL-specific Context Layer. The D-only layer is the skill `agent/skills/booksql-accounting-context/SKILL.md`; it provides a schema/accounting glossary and one explicitly approved policy for case-insensitive matching of text labels. It does not redefine accounting measures.

## Required behavior

1. Use only `cube_semantic` for data access, through `cube_metadata` and `cube_query`. If Cube is unavailable, report the failure; do not fall back to direct SQLite, native SQL, or another data route.
2. Use only measures/dimensions returned by live Cube metadata. Do not invent fields, joins, currencies, sign conventions, accounting formulas, or metrics.
3. Apply the skill's sole D-only matching policy to user-provided text labels: lowercase only the filter value and use exact `equals` on the appropriate Cube string dimension. Do not use `contains` on the current SQLite-backed Cube; it fails because the generated SQL uses unsupported `ILIKE`. Do not broaden a failed match or switch data routes.
4. Read and follow `agent/skills/booksql-accounting-context/SKILL.md` when interpreting BookSQL tables, fields, grain, or relationships. The skill's case rule changes text matching only; it does not change measures, joins, dates, or accounting treatment.
5. Preserve source and Cube-returned values. Do not round source rows or silently apply accounting adjustments. For derived arithmetic, use `decimal_calculator.calculate` on values returned by Cube; keep intermediates unrounded and apply display rounding only when the prompt requests it.
6. Report the Cube semantic route, relevant filters, and time grain. If no normalized exact match or no supported definition is available, state the limitation rather than guessing.
7. Preserve all benchmark prompts and gold queries unchanged. If D's case-insensitive match differs from literal case-sensitive gold execution, label that as the approved D treatment effect rather than an exact gold match. Freeze this D context revision before formal evaluation.

## Arithmetic boundary

`decimal_calculator.calculate` is permitted only for arithmetic over values already returned by Cube. Cube selects, filters, and aggregates data; the calculator evaluates formulas. Do not send raw rows to the calculator for database aggregation.

## First-turn response requirement
Always answer the user's question in the current response and provide every requested field; if data is unavailable, state that explicitly without inventing values or deferring the answer to a follow-up.
