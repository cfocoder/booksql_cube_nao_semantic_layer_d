# Scenario D — Cube BookSQL plus descriptive accounting context

This repository is the Nao Multi-project context for BookSQL scenario **D**.

## Boundary

- No direct SQLite or native database connector is declared.
- BookSQL data is accessed only through `cube_semantic` and the dedicated Cube BookSQL MCP.
- The D-only `booksql-accounting-context` skill corrects the fact-table grain description and adds one approved treatment: case-insensitive matching of user-provided text labels. It does not add accounting formulas, joins, or sign policies.
- `decimal_calculator.calculate` is arithmetic-only over values already returned by Cube.
- The context explicitly avoids Contoso rules and unsupported assumptions about Amount/Credit/Debit, joins, or business behavior.
- For text filters, D lowercases only the user-provided label and uses exact equality against the appropriate Cube string member; it does not change returned labels or numeric/date semantics.

## Project mapping

- Nao project: `booksql-condition-d`
- Repository: `cfocoder/booksql_cube_nao_semantic_layer_d`
- Expected route: `cube_semantic` → private Cube BookSQL MCP → Cube Core → SQLite, plus the D-only glossary skill

## Methodology status

Hector approved case-insensitive matching of text labels as the D-only treatment. The original prompts and gold SQL remain unchanged; if case normalization yields rows where literal gold SQL returns none, record that as a treatment effect, not an exact gold match. Freeze the updated D context before formal A/B/C/D runs.
