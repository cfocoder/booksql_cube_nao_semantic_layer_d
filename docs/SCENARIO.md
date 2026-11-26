# Scenario D deployment contract

## Required Nao project

```text
Project: booksql-condition-d
Repository: https://github.com/cfocoder/booksql_cube_nao_semantic_layer_d.git
Branch: main
```

## Data route and context layer

```text
Nao project -> cube_semantic MCP -> cube-mcp-booksql:8787/mcp
            -> Cube BookSQL API -> SQLite accounting.sqlite
D-only context -> agent/skills/booksql-accounting-context/SKILL.md
```

The Nao project does not receive `CUBE_API_SECRET` or direct database credentials. The Cube MCP service owns the Cube API secret. Do not use the Contoso `cube-mcp-private` endpoint.

The D-only context applies case-insensitive matching to textual Cube filters by lowercasing the prompt value and using exact equality on the appropriate Cube string member. It does not change numeric measures, dates, joins, or returned labels. Do not use Cube's `contains` operator for this on the current SQLite-backed Cube; it generates unsupported `ILIKE` SQL.

## Methodology status

The D context now defines one approved treatment: case-insensitive matching of text labels. It preserves the original questions and gold queries. If D's normalized filter returns data while literal case-sensitive gold SQL returns none, score and report that as a D treatment effect, not as an exact gold match. Freeze this exact context revision before formal evaluation.
