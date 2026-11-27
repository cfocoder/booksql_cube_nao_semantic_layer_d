# Scenario D — Cube BookSQL plus descriptive accounting context

This repository is the Nao Multi-project context for BookSQL scenario **D**.

## Boundary

- No direct SQLite or native database connector is declared.
- BookSQL data is accessed only through `cube_semantic` and the dedicated Cube BookSQL MCP.
- The D-only `booksql-accounting-context` skill corrects the fact-table grain description and adds one approved treatment: case-insensitive text matching that preserves exact-versus-substring semantics. It does not add accounting formulas, joins, or sign policies.
- `decimal_calculator.calculate` is arithmetic-only over values already returned by Cube.
- The context explicitly avoids Contoso rules and unsupported assumptions about Amount/Credit/Debit, joins, or business behavior.
- For substring filters, D uses Cube `contains` on the correct member with the original prompt text. For exact text filters with uncertain case, D first discovers candidate labels through a bounded `contains` query and uses `contains` in the final aggregate only if every returned distinct label is an exact case variant of the requested label. The validated BookSQL deployment uses Cube Core v1.8.32 with `CUBEJS_TESSERACT_SQL_PLANNER=false`. A Langfuse trace confirmed that `contains` succeeded on this path, but did not expose generated SQL; do not claim a specific SQL translation. Do not lowercase the input or silently include partial/longer labels. Returned labels and numeric/date semantics remain unchanged.

## Project mapping

- Nao project: `booksql-condition-d`
- Repository: `cfocoder/booksql_cube_nao_semantic_layer_d`
- Expected route: `cube_semantic` → private Cube BookSQL MCP → Cube Core → SQLite, plus the D-only glossary skill

## Methodology status

Hector approved case-insensitive text matching as the D-only treatment, while preserving each question's exact-versus-substring semantics. The original prompts and gold SQL remain unchanged; if D's case-insensitive result differs from literal case-sensitive gold SQL, record that as a treatment effect, not an exact gold match. Freeze the updated D context before formal A/B/C/D runs.
