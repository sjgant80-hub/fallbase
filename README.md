# FallBase

**Sovereign database · single HTML file · IndexedDB-backed · SQL-like query engine inline**

Part of the TemuOracle suite. Phase 1 of the Oracle wedge — replaces Oracle Database as the relational data layer.
Prime **523** · Version **1.0.0** · MIT · ◊·κ=1

---

## For people who just want to use it

FallBase is a database that lives in your browser. No server, no cloud, no login. You open `index.html`, you get tables, columns, rows, and a SQL editor. Everything persists in IndexedDB.

### Get started

1. Open `index.html` in any modern browser.
2. Click **load sample** in the sidebar to spawn `customers`, `orders`, `products` with realistic SME data.
3. Click the **query** tab and run:
   ```sql
   SELECT name FROM customers WHERE status = 'active' ORDER BY name LIMIT 5
   ```
4. Drag a CSV or JSON file onto a table to import rows.
5. Press **Ctrl+K** for the autopilot palette.

### What you can do

- Create multiple databases, switch between them in the sidebar
- Design schemas with typed columns (integer, real, text, boolean, date, json)
- Mark columns primary, unique, nullable, default
- Declare foreign keys with restrict / cascade / set null on delete
- Add secondary indexes to speed up WHERE / ORDER BY
- Edit cells spreadsheet-style
- Paginate, sort, filter rows in the data view
- Write SQL in the query editor (subset documented below)
- Save queries, recall last 20 from history
- Export results as CSV or JSON
- Export the whole database as JSON, drop a JSON back to import
- Talk to FallBase from sibling tools via postMessage

### Autopilot (Ctrl+K)

Keyword router (T0, offline):

- *"show overdue invoices"* → emits a SELECT against an invoices table
- *"schema for SME customers"* → emits CREATE-like schema suggestion
- *"monthly sales by product"* → GROUP BY query

If you add an Anthropic / OpenAI / Gemini / OpenRouter key in **settings**, T3 parses arbitrary intent into SQL or a `{action, table, ...}` JSON.

---

## For developers

### SQL subset — what works

```
SELECT [DISTINCT] cols | *
FROM table [alias]
[INNER JOIN | LEFT JOIN t [alias] ON a.x = b.y]
[WHERE expr]
[GROUP BY col [, col]]
[HAVING expr]
[ORDER BY col [ASC|DESC]]
[LIMIT n]
```

**WHERE / HAVING operators:** `= != <> < > <= >= AND OR NOT LIKE IN BETWEEN IS NULL IS NOT NULL`

**Aggregates:** `COUNT(*) COUNT(col) SUM(col) AVG(col) MIN(col) MAX(col)`

**Literals:** single-quoted strings (`'foo'`), numbers (`42`, `3.14`), `NULL`, `TRUE`, `FALSE`

**Identifiers:** bare or `table.column`. Optional `AS alias` after columns.

### SQL subset — what does NOT work (intentionally)

- No nested subqueries (`SELECT ... WHERE x IN (SELECT ...)`)
- No CTEs (`WITH ...`)
- No window functions (`OVER (...)`)
- No `UNION` / `EXCEPT` / `INTERSECT`
- No `RIGHT JOIN` / `FULL JOIN` / `CROSS JOIN`
- No `UPDATE` / `INSERT` / `DELETE` / `CREATE TABLE` via SQL — use the schema designer & data view (or the postMessage API)
- No multi-statement scripts (one query at a time)
- No string concatenation operator `||`
- No CASE expressions

### SQL engine architecture

Three stages, all inline:

1. **`tokenize(sql)`** — hand-rolled lexer, returns `[{type, value}, ...]`. Types: `keyword, ident, number, string, op, punct`.
2. **`parse(tokens)`** — recursive descent, returns AST `{type:'select', columns, from, joins, where, groupBy, having, orderBy, limit, distinct}`. Expressions are a tree of `{type:'binop'|'unop'|'col'|'lit'|'func'|'in'|'between'|'isnull'|'like', ...}`.
3. **`execute(ast, db)`** — evaluates against the active database's tables. JOINs use nested loops (O(n·m)); fine for SME-scale datasets, slow at 100k+ rows.

**Indexes:** secondary indexes are `Map<value, Set<rowId>>` rebuilt on column declare or table import. Used by single-column equality WHEREs and ORDER BYs.

**Perf notes:** the engine is correctness-first, not speed-first. A nested-loop JOIN of two 10k tables is ~100M iterations and will block the UI. Index your join columns or keep tables small.

### Test queries (run after loading sample)

```sql
-- Basic
SELECT name, email FROM customers WHERE status = 'active' ORDER BY name LIMIT 5

-- Aggregate
SELECT status, COUNT(*) FROM customers GROUP BY status

-- Join
SELECT c.name, o.total FROM customers c INNER JOIN orders o ON c.id = o.customer_id

-- Filter + aggregate
SELECT product_id, SUM(qty), AVG(price) FROM orders GROUP BY product_id HAVING SUM(qty) > 5

-- LIKE
SELECT * FROM products WHERE name LIKE '%Pro%'

-- BETWEEN + IN
SELECT * FROM orders WHERE total BETWEEN 100 AND 1000 AND status IN ('paid','shipped')
```

### postMessage API

Sibling tools (FallLedger, FallBuild, FallReport) can drive FallBase:

```javascript
// Query
window.postMessage({target:'fallbase', action:'query', sql:'SELECT * FROM customers LIMIT 10'}, '*');

// Insert
window.postMessage({target:'fallbase', action:'insert', table:'customers', row:{name:'X', email:'x@y.z'}}, '*');

// List tables
window.postMessage({target:'fallbase', action:'list-tables'}, '*');

// Ping (doctrine standard)
window.postMessage({target:'fallbase', action:'ping'}, '*');
```

Responses come back as `{target, responseTo, data}` to `e.source`.

### Estate hooks

- **KONOMI** sovereign shim baked, prime **523**, inert by default
- **fallmesh** BroadcastChannel on `fall-signal`, replies to `ping` with `pong`
- PWA manifest as data: URL
- Cascade T0/T2/T3 detection (Ollama at `127.0.0.1:11434`, else BYOK)

### File deliverables

- `index.html` — the tool (single file)
- `README.md` — this file
- `LICENSE` — MIT
- `.nojekyll` — Pages legacy deploy marker

### TemuOracle suite plan

FallBase is **phase 1** of the Oracle wedge:

| Phase | Replaces | Tool |
|---|---|---|
| 1 | Oracle Database | **FallBase** (this) |
| 2 | NetSuite GL | FallLedger |
| 3 | NetSuite Inventory / WMS | FallBuild |
| 4 | NetSuite Reporting / BI | FallReport |

All four sovereign, all four single-file, all four bound to prime 523's family via `fall-signal`.

---

◊·κ=1 · MIT · prime 523 · v1.0.0
