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

For exact entity filters, D resolves a stored label through `equals`, then case-insensitive exact matching, then only a validated terminal-descriptor normalization. `contains` is for bounded candidate discovery, not an equality substitute. Intentional benchmark substring filters still use `contains` as their final operator. The initial descriptor allowlist contains only trailing `partnership`; other titles and legal suffixes remain disabled pending validation. The validated BookSQL deployment uses Cube Core v1.8.32 with `CUBEJS_TESSERACT_SQL_PLANNER=false`; a Langfuse trace confirmed that `contains` succeeded on this path, but did not expose generated SQL. Do not claim a specific SQL translation.

For a question unambiguously asking whether a payment was received from a customer, the D skill also filters the live Cube transaction-type member to its confirmed stored `payment` label using `equals`, alongside the resolved customer and requested date. Dev Gold evidence supports this only for the “receive payment from” wording class (471/471 Gold filters use `transaction_type = 'payment'`). Do not apply this rule to “payment method” questions (606/606 use `sales receipt`) or assume the same mapping for payments to vendors.

## Fiscal-year convention

BookSQL fiscal years run from April 1 through March 31, as implied by the authors' [DIN-SQL example](https://github.com/Exploration-Lab/BookSQL/blob/main/GPT/DIN-SQL.py#L1584). With the benchmark reference date 2022-06-01, “this fiscal year” and “this fiscal year to date” mean 2022-04-01 through 2022-06-01; “last fiscal year” means 2021-04-01 through 2022-03-31. A full FY 2022–23, when explicitly requested, means 2022-04-01 through 2023-03-31.

The validation split has 150 Gold queries worded “this fiscal year to date” whose SQL instead covers the full FY through 2023-03-31. Keep questions and official Gold unchanged; flag these cases separately as Gold/wording inconsistencies.

## Methodology status

The D context defines evidence-based entity resolution and the narrowly validated received-payment event filter while preserving exact-versus-substring semantics. It preserves the original questions and Gold SQL. If entity resolution, case-insensitive matching, or payment-event classification changes the result relative to literal Gold execution, report it as a D treatment effect, not an exact Gold match. Freeze this exact context revision before formal evaluation.
