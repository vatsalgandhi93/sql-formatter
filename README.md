# 🧱 SQL Dialect Formatter

A dependency-free SQL formatter that produces clean, executable output across 12 SQL dialects. Single HTML file. No install, no server, no internet.

🔗 Link to the [SQL Dialect Formatter](https://vatsalgandhi93.github.io/sql-formatter/)

**By EigenZ Solutions LLC** · **v1.19.0**

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

- 📐 **Clause-per-line** layout — SELECT, FROM, JOINs, WHERE, GROUP BY, HAVING, QUALIFY, ORDER BY, LIMIT
- 🪜 **One column per line**, one condition per line, split by `AND` / `OR`
- 🪟 **Window functions** — `OVER`, `PARTITION BY`, `ROWS BETWEEN`, `ROW_NUMBER`, `RANK`, `LAG`, `LEAD`
- 🌲 **CTEs & subqueries** — bracket-aligned, multi-level nesting, depth-aware
- 🌀 **CASE expressions** — nested, searched, inside function calls, with empty `ELSE`
- 💬 **Three comment styles** — `--`, `#` (MySQL/BigQuery/Hive), and multi-line `/* … */`
- 🔠 **~200 keywords uppercased** — UNNEST, SPLIT, REGEXP_*, ARRAY_*, DATE_*, JSON_*, and more
- ✅ **Always executable** — no stray trailing commas before clause boundaries
- 🎨 **Light & dark themes** — your choice persists
- ⚙️ **Configurable** — indent width, comma style, dialect selector

---

## 🚀 Use it

Open `sql-formatter-tester.html` in any browser. That's it.

- Paste SQL on the left
- Click **Format SQL**
- Copy clean SQL from the right

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

© 2026 EigenZ Solutions LLC. All rights reserved.
