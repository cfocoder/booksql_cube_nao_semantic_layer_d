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

## Methodology status

The D skill is a conservative descriptive schema glossary proposed to replace the Contoso-specific skill/policies. It does not add formulas, date rules, thresholds, or joins. The researcher must review and freeze this treatment before formal evaluation; keep original questions and gold queries unchanged.
