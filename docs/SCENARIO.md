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

When the question requires substring matching (for example, the account phrase in an `instr`-style benchmark query), the D-only context uses Cube's `contains` filter on the semantically appropriate string member and preserves the prompt's original text. The deployed Cube v1.7.24 SQLite adapter compiles `contains` to case-insensitive SQLite `LIKE ... COLLATE NOCASE`; do not lowercase the filter value or write raw SQL. Because `contains` is broader than exact equality, use it only when substring semantics are intended. This does not change numeric measures, dates, joins, or returned labels.

## Methodology status

The D context defines one approved treatment: case-insensitive substring matching where the question's semantics require it. It preserves the original questions and gold queries. If D's filter returns data while literal case-sensitive gold SQL returns none, score and report that as a D treatment effect, not as an exact gold match. Freeze this exact context revision before formal evaluation.
