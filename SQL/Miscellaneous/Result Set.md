---
tags: [sql, fundamentals]
---

- A **result set** is the temporary table (rows and columns) returned by a query. It is not stored anywhere permanently — it exists only for the duration of the response.

---

### Key Points

- **SELECT** queries return a result set to the client (terminal, application, GUI tool).
- **Subqueries** return a result set that the outer query uses as if it were a table. See [[Subqueries]], [[Common Table Expressions]].
- **Set operations** combine result sets from multiple queries. See [[UNION and UNION ALL]], [[INTERSECT and EXCEPT]].
- **Views** are stored queries — each time you query a view, it runs the underlying SELECT and produces a fresh result set.
- **Temporary tables** are the exception — you can explicitly store a result set with `CREATE TEMPORARY TABLE`.
- Without `ORDER BY`, the order of rows in a result set is **not guaranteed**. See [[ORDER BY]].
