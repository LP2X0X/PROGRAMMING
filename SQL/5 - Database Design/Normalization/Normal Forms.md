---
tags: [sql, normalization, database-design]
---

- **Normalization** is the process of organizing a database to reduce redundancy and improve data integrity. It works by decomposing tables into smaller, well-structured tables linked by [[Foreign Key|foreign keys]].

---

### The Practical Rule

> "Every non-key column must depend on **the key**, **the whole key**, and **nothing but the key**."

- **The key** → 1NF (has a primary key)
- **The whole key** → 2NF (no partial dependencies)
- **Nothing but the key** → 3NF (no transitive dependencies)

---

### 1NF (First Normal Form)

**Rules:**
1. Each column contains **atomic** (indivisible) values — no lists, no sets, no comma-separated values.
2. Each row is **unique** (has a [[Primary Key]]).
3. No repeating groups (no `phone1`, `phone2`, `phone3` columns).

**Violation:**

| id | name  | phones                |
|----|-------|-----------------------|
| 1  | Alice | 555-1234, 555-5678    |

**Fix:** separate table for phones:

| id | name  |
|----|-------|
| 1  | Alice |

| phone_id | employee_id | phone    |
|----------|-------------|----------|
| 1        | 1           | 555-1234 |
| 2        | 1           | 555-5678 |

---

### 2NF (Second Normal Form)

**Rules:**
1. Must be in 1NF.
2. Every non-key column depends on the **entire** primary key (no **partial dependencies**).

- Only relevant when you have a **composite primary key**.

**Violation:** PK is (student_id, course_id)

| student_id | course_id | student_name | grade |
|------------|-----------|--------------|-------|
| 1          | 101       | Alice        | A     |

- `student_name` depends only on `student_id`, not on the full key `(student_id, course_id)`.

**Fix:** move `student_name` to a `students` table.

---

### 3NF (Third Normal Form)

**Rules:**
1. Must be in 2NF.
2. No **transitive dependencies** — non-key columns must not depend on other non-key columns.

**Violation:**

| employee_id | name  | department_id | department_name |
|-------------|-------|---------------|-----------------|
| 1           | Alice | 10            | Engineering     |

- `department_name` depends on `department_id`, not on `employee_id` (the primary key).

**Fix:** move department info to a `departments` table:

| employee_id | name  | department_id |
|-------------|-------|---------------|

| department_id | department_name |
|---------------|-----------------|

---

### Beyond 3NF (Briefly)

| Form | What it adds                                      | Practical need |
| ---- | ------------------------------------------------- | -------------- |
| BCNF | Every determinant is a candidate key               | Rare edge cases |
| 4NF  | No multi-valued dependencies                       | Very rare      |
| 5NF  | No join dependencies                               | Almost never   |

- For most applications, **3NF is sufficient**. Going beyond 3NF is typically academic.

```ad-tip
Normalize your schema first for correctness and clarity. If performance becomes an issue, selectively [[When to Denormalize|denormalize]] the specific areas that need it — don't skip normalization upfront.
```
