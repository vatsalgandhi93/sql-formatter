# 🧱 SQL One : Multi-Dialect Formatter

A dependency-free SQL formatter that produces clean, executable output across 12 SQL dialects. Single HTML file. No install, no server, no internet.

🔗 **Try it live** → [SQL One Web Tester](https://vatsalgandhi93.github.io/sql-formatter/)

**By EigenZ Solutions LLC**

---

## 📦 Get it as an editor extension

The same formatter is available as a one-click install for every major SQL-using IDE:

- 🟦 **VS Code** → [Install from VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=vatsalgandhi93.sql-dialect-formatter)
- 🟪 **Cursor, Windsurf, VSCodium, Antigravity, Gitpod, Theia** → [Install from Open VSX](https://open-vsx.org/extension/vatsalgandhi93/sql-dialect-formatter)

Once installed, right-click any SQL file → **SQL: Format Document** (or press `Shift+Alt+F`). Works with `.sql`, `.bqsql`, `.pgsql`, `.tsql`, `.mysql`, `.snowflake`, and many more.

---

## ✨ See it

Messy in:

```sql
wiTh ledger aS(sElEcT account_id,sum(amount)as spend fRoM tx whErE status='ok' grOuP bY account_id)
seLeCt cAse wHeN spend>1000 tHeN 'high' eLsE 'low' eNd as tier fRoM ledger
```

Clean out:

```sql
WITH
  ledger AS (
             SELECT
               account_id,
               SUM(amount) AS spend
             FROM tx
             WHERE
               status = 'ok'
             GROUP BY
               account_id
            )
SELECT
  CASE
    WHEN spend > 1000 THEN 'high'
    ELSE 'low'
  END AS tier
FROM ledger
```

---

## 🌐 Dialects

Auto-detected, or pick manually.

☁️ **Cloud** — Azure Synapse · Amazon Redshift · AWS Athena · Databricks · BigQuery · Snowflake

🗄️ **Traditional** — PostgreSQL · MySQL · SQL Server · Spark SQL · Generic SQL

---

## 🛠️ Features

- 📐 **Clause-per-line** layout — SELECT, FROM, JOINs, WHERE, GROUP BY, HAVING, QUALIFY, ORDER BY, LIMIT, OFFSET, FETCH NEXT
- 🪜 **One column per line**, one condition per line, split by `AND` / `OR`
- 🪟 **Window functions** — `OVER`, `PARTITION BY`, `ROWS BETWEEN`, `ROW_NUMBER`, `RANK`, `LAG`, `LEAD`
- 🌲 **CTEs & subqueries** — bracket-aligned, multi-level nesting, depth-aware
- 🌀 **CASE expressions** — nested, searched, inside function calls, with empty `ELSE`
- 🧱 **Full DDL/DML/Procedural** — CREATE TABLE/VIEW/PROCEDURE/FUNCTION, INSERT/UPDATE/DELETE/MERGE, BEGIN/END blocks, IF/WHILE, transactions
- 💬 **Three comment styles** — `--`, `#` (MySQL/BigQuery/Hive), and multi-line `/* … */` — inline trailing comments preserved with their code
- ⚠️ **Dialect compatibility warnings** — surfaces features that won't run on the selected dialect (e.g. `QUALIFY` in PostgreSQL, `IFF` in MySQL)
- 🔠 **~200 keywords uppercased** — UNNEST, SPLIT, REGEXP_*, ARRAY_*, DATE_*, JSON_*, and more
- ✅ **Always executable** — no stray trailing commas before clause boundaries
- 🎨 **Light & dark themes** — your choice persists
- ⚙️ **Configurable** — indent width, comma style, dialect selector

---

## 🚀 Use it

**In your browser** — visit the [web tester](https://vatsalgandhi93.github.io/sql-formatter/) or open `sql-formatter-tester.html` locally:
- Paste SQL on the left
- Click **Format SQL**
- Copy clean SQL from the right

**In your editor** — install the extension from [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=vatsalgandhi93.sql-dialect-formatter) or [Open VSX](https://open-vsx.org/extension/vatsalgandhi93/sql-dialect-formatter):
- Right-click any SQL file → **SQL: Format Document**
- Or press `Shift+Alt+F`
- Or enable **Format on Save** in your editor settings

---

## 🔒 Privacy

- Runs entirely client-side. Your queries never leave your browser or editor.
- No telemetry, no signup, no cloud roundtrip.
- The extension and web tester are functionally identical — same formatter, different delivery.

---

## ⚡ Performance

| Input size | Format time |
|---|---|
| 1 KB | 6 ms |
| 10 KB | 43 ms |
| 100 KB | 128 ms |
| 500 KB | 352 ms |

A 2,000-line query with 60 CTEs formats in **48 ms**. No hard character limit.

---

## 🌍 Share it

Want to host the tester online?

- 🐙 **GitHub Pages** — push to a repo, enable Pages
- 🪂 **Netlify Drop** — drag the HTML to <https://app.netlify.com/drop>
- ☁️ **Cloudflare Pages** — same, with custom domains

Zero external dependencies, so it works on any static host.

---

## 📜 License

© 2026 EigenZ Solutions LLC. All rights reserved. MIT license.
