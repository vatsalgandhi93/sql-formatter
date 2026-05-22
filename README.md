# SQL Dialect Formatter

A lightweight, dependency-free SQL formatter that produces clean, readable, executable output across all major SQL dialects. Ships as a single-file HTML tester for the browser and as a VS Code extension.

**By EigenZ Solutions LLC** &nbsp;·&nbsp; Current version: **v1.19.0**

---

## What it does

Takes messy SQL like this:

```sql
wiTh monthly_ledger aS(sElEcT account_id,coalesce(signup_country,'Unknown')as country,
-- baseline filter
# tracks only active enterprise tiers
sum(total_purchase_amount)as gross_spend,avg(total_purchase_amount)over(
pArTiTiOn bY signup_country rOwS bEtWeEn 3 pReCeDiNg aNd cUrReNt rOw)as rolling_avg
fRoM transaction_ledger whErE transaction_status='Settled'and log_date>=
date_add('day',-90,current_date())grOuP bY account_id,signup_country)
seLeCt r.account_id,cAse wHeN r.tier_rank<=5 tHeN 'Tier 1'eLsE 'Tier 3'eNd AS perf
fRoM monthly_ledger r wHeRe r.tier_rank<=10
```

And produces clean, executable SQL:

```sql
WITH
  monthly_ledger AS (
                     SELECT
                       account_id,
                       COALESCE(signup_country,'Unknown') AS country,
                       -- baseline filter
                       # tracks only active enterprise tiers
                       SUM(total_purchase_amount) AS gross_spend,
                       AVG(total_purchase_amount) OVER (PARTITION BY signup_country ROWS BETWEEN 3 PRECEDING AND CURRENT ROW) AS rolling_avg
                     FROM transaction_ledger
                     WHERE
                       transaction_status = 'Settled'
                       AND log_date >= DATE_ADD('day',-90,CURRENT_DATE())
                     GROUP BY
                       account_id,
                       signup_country
                    )
SELECT
  r.account_id,
  CASE
    WHEN r.tier_rank <= 5 THEN 'Tier 1'
    ELSE 'Tier 3'
  END AS perf
FROM monthly_ledger r
WHERE
  r.tier_rank <= 10
```

---

## Supported dialects (12)

Auto-detected from query content, or selectable manually.

**Cloud warehouses**
- Azure Synapse · Amazon Redshift · AWS Athena · Databricks · BigQuery · Snowflake

**Traditional**
- PostgreSQL · MySQL · SQL Server (T-SQL) · Spark SQL · Generic SQL

Dialect detection uses keyword signatures: e.g., `qualify` plus backtick-quoted three-part names suggests BigQuery; `nolock` or `select top N` suggests SQL Server; `distkey` / `sortkey` suggests Redshift.

The dialect selector both labels the output header (`-- Dialect: BigQuery (auto-detected)`) and shows whether your manual selection agrees with what auto-detect would have chosen.

---

## Formatter capabilities

### Clause structure
- Each top-level clause on its own line: `SELECT`, `FROM`, `JOIN` variants, `WHERE`, `GROUP BY`, `HAVING`, `QUALIFY`, `ORDER BY`, `LIMIT`, `OFFSET`
- One column per line in SELECT
- One condition per line in `WHERE` / `HAVING` / `QUALIFY` / `JOIN ON`, split by `AND` / `OR`
- One source per line in multi-source `FROM` (e.g. `FROM t, UNNEST(SPLIT(...)) AS x`)
- Configurable indent (2, 4, or 6 spaces)
- Configurable comma style (trailing or leading)

### Subqueries & CTEs
- Subqueries in `WHERE`, `JOIN`, or `WITH` are bracket-aligned — content indents to the column right after the opening `(`, closing `)` aligns under it
- Subquery inside `JOIN (...) alias ON ...` is expanded correctly with depth-aware `ON` / `USING` detection
- Multiple CTEs, nested CTEs, and CTEs referencing other CTEs all handled

### CASE expressions
- `CASE` / `WHEN` / `THEN` / `ELSE` / `END` properly indented
- Searched `CASE` (with case-expression before first `WHEN`) supported
- Nested `CASE` expressions supported
- `CASE` inside function parens (e.g. `LOWER(CASE WHEN ... END)`) bracket-aligned
- Empty `ELSE` (just `ELSE END`) handled
- Token-boundary aware — recognizes `'value'END` and `)END` correctly without requiring whitespace

### Window functions
- Full `OVER (...)` support with `PARTITION BY`, `ORDER BY`, frame clauses (`ROWS BETWEEN`, `RANGE BETWEEN`, `UNBOUNDED PRECEDING`, `CURRENT ROW`)
- `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE`, and more

### Comments — all three styles
- `-- line comments` (standard SQL)
- `# line comments` (MySQL, BigQuery, Hive, Snowflake)
- `/* block comments */` including multi-line block comments
- Header comments before the query stay at top
- Comments between SELECT columns / WHERE conditions / FROM sources / ORDER BY items get their own indented line
- Comments inside string literals (`'--prefix'`, `'#hash'`, `'/* fake */'`) are correctly preserved, not mis-classified
- Multi-line block comments are re-indented to align with their surroundings

### Always produces executable SQL
- Comments after the last real column never cause a trailing comma before `FROM` / `GROUP BY` / `ORDER BY` / `LIMIT` — the formatter filters out comment-only items when determining list-terminating columns, so output is always parseable by real SQL engines.

### Keyword recognition (~200 keywords/functions)
- Aggregate: `AVG`, `SUM`, `COUNT`, `STRING_AGG`, `ARRAY_AGG`, `APPROX_COUNT_DISTINCT`, `PERCENTILE_CONT`, `STDDEV`
- String: `SPLIT`, `SUBSTRING`, `REGEXP_EXTRACT`, `REGEXP_REPLACE`, `STARTS_WITH`, `ENDS_WITH`, `LPAD`, `INITCAP`
- Array: `UNNEST`, `ARRAY_LENGTH`, `ARRAY_CONCAT`, `GENERATE_ARRAY`, `WITH OFFSET`
- Date/time: `DATE_TRUNC`, `DATE_DIFF`, `TIMESTAMP_ADD`, `EXTRACT`, `FORMAT_DATE`, `PARSE_DATE`, `CURRENT_DATE`, `UNIX_SECONDS`
- Math: `ROUND`, `CEIL`, `FLOOR`, `SQRT`, `POWER`, `GREATEST`, `LEAST`
- Conversion / null: `SAFE_CAST`, `TRY_CAST`, `IFNULL`, `NVL`, `COALESCE`, `DECODE`
- JSON: `JSON_EXTRACT`, `JSON_VALUE`, `PARSE_JSON`, `TO_JSON_STRING`, `FLATTEN`
- Set operators: `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT`

---

## Browser tester

Open `sql-formatter-tester.html` in any modern browser. No server, no install, no internet required.

**Layout**
- Side-by-side input and output panels, each 600px tall (≈ 6 inches)
- Line numbers on both panels
- Output line numbers are pinned to the left edge — they stay visible while horizontally scrolling long lines
- Always-visible scrollbars (no hover-to-reveal)
- Light and dark mode toggle (persisted across sessions)
- Live stats: lines before/after, clause count, subquery/CTE count

**Controls**
- Dialect selector (auto-detect or manual)
- Comma position (trailing `col,` or leading `,col`)
- Indent width (2/4/6 spaces)
- Format · Load sample · Clear · Copy

**Performance**

| Input size | Format time |
|---|---|
| 1 KB / ~100 cols | 6 ms |
| 10 KB / ~1,000 cols | 43 ms |
| 100 KB / ~10,000 cols | 128 ms |
| 500 KB / ~50,000 cols | 352 ms |

A real-world 2,000-line query with 60 CTEs (~30 KB) formats in **48 ms**. No hard character limit — the textarea has no `maxlength` and there is no input cap in the formatter logic. Browser DOM performance becomes the practical bottleneck above ~5 MB.

---

## VS Code extension

The extension package (`sql-dialect-formatter.zip`) provides the same formatter inside VS Code:

- Command Palette: **`Format SQL`** and **`Select SQL Dialect`**
- Right-click in any `.sql` file → **Format SQL**
- Status bar shows current dialect (click to change)
- Per-document dialect override
- Pure JS, zero npm dependencies

### Install locally (sideload)

```bash
unzip sql-dialect-formatter.zip
cd sql-dialect-formatter
npm install -g @vscode/vsce
vsce package        # produces a .vsix file
code --install-extension sql-dialect-formatter-*.vsix
```

### Publish to VS Code Marketplace

1. Create a Publisher account at <https://marketplace.visualstudio.com/manage>
2. Edit `package.json` — set the `publisher` field
3. Create a Personal Access Token at <https://dev.azure.com> with Marketplace > Manage scope
4. Run `vsce login <publisher>` and paste the PAT
5. Add required fields to `package.json` if missing: `repository`, `license`, `icon` (128×128 PNG in `images/`)
6. `vsce publish` (or `vsce publish patch` to bump version automatically)
7. For Open VSX (used by Cursor, VSCodium, etc.): separately `ovsx publish`

---

## Architecture

The formatter is a single pure JS function (`formatSQL`). It runs a 10-stage pipeline that's easy to reason about and modify.

1. **Normalize line endings** — CRLF → LF
2. **Protect string literals** — quoted strings become `§n§` placeholders (escaped quotes `''` preserved)
3. **Protect comments** — `/* … */`, `--`, and `#` all become `«CMTn»` placeholders. Block comments on their own source line get padded so per-clause logic later peels them onto their own output line
4. **Collapse whitespace** — placeholders are opaque so nothing is lost
5. **Peel off leading header comments** — emitted at top of output
6. **Uppercase keywords** — case-insensitive regex replace against ~200 keyword set
7. **Split into clauses** — depth-aware so `ORDER BY` inside an `OVER(...)` window isn't confused with a top-level clause
8. **Render each clause** — SELECT columns, FROM (single or multi-source), JOIN with `ON`/`USING` and subquery expansion, WHERE/HAVING/QUALIFY split by AND/OR, GROUP BY / ORDER BY with comma-aware comment handling, LIMIT/OFFSET. CASE expressions expanded recursively
9. **Restore string placeholders**
10. **Restore comment placeholders** — multi-line block comments re-indented to align with their column position in the output

Key helpers: `splitByCommaDepth0`, `splitByAndOrDepth0`, `findCloseParen`, `renderSubquery`, `renderExprWithCase`, `splitCaseClauses`, `stripTrailingCmts`. Each is small and focused.

---

## Hosting

For sharing the browser tester:

- **GitHub Pages** — drop the HTML in a repo, enable Pages, share the URL
- **Netlify Drop** — drag the HTML file at <https://app.netlify.com/drop> for an instant URL
- **Cloudflare Pages** — same, with custom domain support
- **Any static host** — the file has zero external dependencies

For VS Code distribution: the Marketplace (via `vsce publish`) and Open VSX Registry (via `ovsx publish`).

---

## Version history

| Version | Highlights |
|---|---|
| **v1.19.0** | Trailing-comma suppression filters comment-only items across SELECT / GROUP BY / ORDER BY / FROM — output is now always executable SQL |
| v1.18.0 | CASE keyword boundary recognizes non-whitespace token boundaries (`'value'END`, `)END`); fixed spurious extra `END` |
| v1.17.0 | `#` line comments (MySQL/BigQuery/Hive/Snowflake); string-literal protection moved before comment capture; dialect selector now passed to formatter and visible in output |
| v1.16.0 | Multi-line `/* … */` block comments fully supported with re-indentation on restore |
| v1.15.0 | WHERE/HAVING/QUALIFY trailing-comment extraction |
| v1.14.0 | ~200 SQL functions added to keyword list — UNNEST / SPLIT / REGEXP_* / ARRAY_* / DATE_* / JSON_* all uppercased and highlighted |
| v1.13.0 | Output line numbers stay pinned to left edge via `position: sticky` |
| v1.12.0 | FROM with comma-separated sources splits to one source per line — UNNEST table-source patterns work |
| v1.11.0 | UNNEST + CROSS JOIN + OFFSET keywords; box height bumped to 600px |
| v1.10.0 | QUALIFY clause (Snowflake / BigQuery / Databricks / Teradata); window function keywords |
| v1.9.0 | Fixed-size scrollbar tracks; CASE inside function parens; multi-condition JOIN ON split |
| v1.8.0 | Syntax highlighter rewritten token-by-token — keywords inside comments/strings never highlighted |
| v1.7.0 | CASE / WHEN / THEN / ELSE / END formatting with nested + searched CASE |
| v1.6.0 | JOIN with subquery table expression; depth-aware ON detection |
| v1.5.0 | Visible version stamp for cache invalidation |
| v1.4.0 | Per-document dialect override (extension); status bar indicator |
| v1.3.0 | CTE bracket alignment in WITH clause |
| v1.2.0 | Dialect auto-detection across 12 dialects |
| v1.1.0 | Browser tester with light/dark mode |
| v1.0.0 | Initial release |

---

## License

© 2026 EigenZ Solutions LLC. All rights reserved.

SQL Dialect Formatter by EigenZ Solutions LLC.
