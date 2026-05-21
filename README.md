# SQL Dialect Formatter — Browser Tester

A zero-dependency, browser-based SQL formatter that understands the quirks of 12 SQL platforms. Open the HTML file locally or host it anywhere — no server, no build step, no install required.

---

## What it does

Paste a messy single-line SQL query on the left and get clean, structured SQL on the right with:

- **Clause-per-line formatting** — `SELECT`, `FROM`, `JOIN`, `WHERE`, `GROUP BY`, `ORDER BY`, `HAVING`, and `LIMIT` each start on their own line
- **One column per line** in `SELECT`, `GROUP BY`, and `ORDER BY`
- **One condition per line** in `WHERE` and `HAVING`
- **Bracket-aligned subqueries** — content inside `(SELECT ...)` indents to the column right after the opening parenthesis, with the closing `)` snapping back underneath
- **Bracket-aligned CTEs** — each `WITH cte AS (...)` body follows the same bracket-alignment rule
- **Keyword uppercasing** — all SQL keywords normalised to uppercase
- **Dialect auto-detection** — scans the query for platform-specific syntax and picks the right dialect automatically
- **Configurable** — switch dialect, comma style (trailing vs leading), and indent size on the fly

---

## Supported dialects

### Cloud data warehouses

| Dialect | Detection signals |
|---|---|
| **Azure Synapse** | `DISTRIBUTION = HASH/ROUND_ROBIN/REPLICATE`, `CLUSTERED COLUMNSTORE INDEX`, `sys.dm_pdw_*` |
| **Amazon Redshift** | `DISTKEY`, `SORTKEY`, `DISTSTYLE`, `UNLOAD`, `STV_*` system tables |
| **AWS Athena** | `STORED AS PARQUET/ORC`, `LOCATION 's3://…'`, `ROW FORMAT SERDE`, `EXTERNAL TABLE`, `MSCK REPAIR TABLE` |
| **Databricks** | `ZORDER BY`, `DESCRIBE HISTORY`, `VACUUM … RETAIN`, `MERGE INTO … WHEN MATCHED`, `USING DELTA` |
| **BigQuery** | Backtick `` `project.dataset.table` ``, `STRUCT<>`, `UNNEST()`, `QUALIFY`, `FARM_FINGERPRINT` |
| **Snowflake** | `VARIANT`, `OBJECT_CONSTRUCT`, `FLATTEN()`, `IFF()`, `$1` positional binds, `ZEROIFNULL` |

### Traditional platforms

| Dialect | Detection signals |
|---|---|
| **PostgreSQL** | `::cast`, `RETURNING`, `ILIKE`, `JSONB`, `ON CONFLICT`, `GENERATE_SERIES` |
| **MySQL / MariaDB** | `AUTO_INCREMENT`, `ENGINE=`, `GROUP_CONCAT`, `LIMIT offset, count` |
| **SQL Server / T-SQL** | `NOLOCK`, `NVARCHAR`, `SELECT TOP n`, `CROSS APPLY`, `OUTER APPLY` |
| **Spark SQL** | `EXPLODE()`, `COLLECT_LIST`, `LATERAL VIEW`, `USING DELTA` |
| **Generic SQL** | Fallback when no platform-specific signals are found |

---

## Example

**Input** (real query, no formatting):

```sql
with active_users as (select id,name,email from users where active=true) select u.id,u.name from active_users u where u.id>(select min(id) from active_users) order by u.name asc
```

**Output** (2-space indent, trailing commas, auto-detected as PostgreSQL):

```sql
-- Dialect: PostgreSQL
WITH
  active_users AS (
                   SELECT
                     id,
                     name,
                     email
                   FROM users
                   WHERE
                     active = TRUE
                  )
SELECT
  u.id,
  u.name
FROM active_users u
WHERE
  u.id > (
          SELECT
            MIN(id)
          FROM active_users
         )
ORDER BY
  u.name ASC
```

---

## Usage

### Run locally

```bash
# Clone the repo
git clone https://github.com/your-username/sql-dialect-formatter-tester.git
cd sql-dialect-formatter-tester

# Open in browser — no server needed
open sql-formatter-tester.html        # macOS
xdg-open sql-formatter-tester.html   # Linux
start sql-formatter-tester.html       # Windows
```

### Host it

Because it is a single self-contained HTML file with no external dependencies, it can be hosted anywhere:

- **GitHub Pages** — drop it in `/docs` or the root of a `gh-pages` branch and enable Pages in repo settings
- **Netlify / Vercel** — drag-and-drop deploy
- **Any static host** — copy the file to your web root

---

## Controls

| Control | Options | Default |
|---|---|---|
| Dialect | Auto-detect, Azure Synapse, Amazon Redshift, AWS Athena, Databricks, BigQuery, Snowflake, PostgreSQL, MySQL, SQL Server, Spark SQL, Generic SQL | Auto-detect |
| Commas | Trailing `col,` · Leading `,col` | Trailing |
| Indent | 2 · 4 · 6 spaces | 2 spaces |

- **Load sample** — loads a multi-CTE query with a subquery in `WHERE` to demonstrate all formatting features at once
- **Copy** — copies the formatted output to your clipboard
- **Clear** — resets both panels

---

## How the formatter works

The formatter is ~250 lines of pure JavaScript with no dependencies. The pipeline for each query is:

1. Uppercase all SQL keywords
2. Collapse whitespace and normalise operator spacing
3. Protect string literals so their contents are never modified
4. Split the query into top-level clauses by walking character-by-character and tracking parenthesis depth — this ensures clause keywords inside subqueries are never treated as clause boundaries
5. Render each clause with the appropriate indentation rules
6. For any `(SELECT ...)` encountered inside a condition or column expression, measure the column position of the `(` and indent the inner query to align with the character immediately after it, with the closing `)` snapping back to the column of the `(`
7. Apply the same bracket-alignment logic to each CTE body in a `WITH` block
8. Restore protected string literals

Auto-detection runs before formatting. The detector checks for platform-specific keywords and syntax patterns in a priority order that prevents false positives — for example, Azure Synapse is checked before SQL Server (which it is a superset of), and Databricks before Spark SQL.

---

## Relation to the VS Code extension

This tester runs the exact same formatting engine as the **SQL Dialect Formatter** VS Code extension (coming soon to the VS Code Marketplace and Open VSX Registry). It is useful for:

- Quickly testing formatting behaviour without opening an editor
- Sharing formatted SQL snippets with teammates who don't have the extension
- Validating queries before pasting them into documentation or pull requests

---

## Contributing

Bug reports and pull requests are welcome. If you find a SQL pattern that formats incorrectly, please open an issue and include the input query and what you expected the output to look like.

---

## License

MIT
