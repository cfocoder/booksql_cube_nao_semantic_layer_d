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

For substring semantics (for example, an `instr`-style benchmark filter), D uses Cube `contains` on the appropriate string member with the original prompt text. For exact text equality with uncertain case, D first runs a bounded `contains` candidate query for that member and checks all distinct results. It may use `contains` in the final aggregation only if every candidate is exactly the requested label ignoring case; if any candidate is a longer/partial label, results may be truncated, or no exact candidate exists, it reports the limitation rather than broadening the query. The validated BookSQL deployment uses Cube Core v1.8.32 with `CUBEJS_TESSERACT_SQL_PLANNER=false`. A Langfuse trace confirmed that `contains` succeeded on this path, but did not expose generated SQL; do not claim a specific SQL translation. Do not lowercase the filter value or write raw SQL. This does not change numeric measures, dates, joins, or returned labels.

## Methodology status

The D context defines one approved treatment: case-insensitive text matching while preserving exact-versus-substring semantics. It preserves the original questions and gold queries. If D's filter returns data while literal case-sensitive gold SQL returns none, score and report that as a D treatment effect, not as an exact gold match. Freeze this exact context revision before formal evaluation.
