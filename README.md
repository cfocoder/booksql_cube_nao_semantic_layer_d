# Scenario D — Cube BookSQL plus descriptive accounting context

This repository is the Nao Multi-project context for BookSQL scenario **D**.

## Boundary

- No direct SQLite or native database connector is declared.
- BookSQL data is accessed only through `cube_semantic` and the dedicated Cube BookSQL MCP.
- The D-only `booksql-accounting-context` skill corrects the fact-table grain description and adds one approved treatment: case-insensitive substring matching where question semantics require it. It does not add accounting formulas, joins, or sign policies.
- `decimal_calculator.calculate` is arithmetic-only over values already returned by Cube.
- The context explicitly avoids Contoso rules and unsupported assumptions about Amount/Credit/Debit, joins, or business behavior.
- For substring text filters, D uses Cube's `contains` operator on the appropriate string member with the original prompt text. Cube v1.7.24's SQLite adapter compiles this to case-insensitive `LIKE ... COLLATE NOCASE`; do not lowercase the filter value or use `contains` when exact equality is required. Returned labels and numeric/date semantics remain unchanged.

## Project mapping

- Nao project: `booksql-condition-d`
- Repository: `cfocoder/booksql_cube_nao_semantic_layer_d`
- Expected route: `cube_semantic` → private Cube BookSQL MCP → Cube Core → SQLite, plus the D-only glossary skill

## Methodology status

Hector approved case-insensitive substring matching of text labels as the D-only treatment where substring semantics are intended. The original prompts and gold SQL remain unchanged; if the D `contains` filter yields rows where literal case-sensitive gold SQL returns none, record that as a treatment effect, not an exact gold match. Freeze the updated D context before formal A/B/C/D runs.
