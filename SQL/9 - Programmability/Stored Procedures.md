---
tags: [sql, programmability]
---

- A **stored procedure** is a named block of SQL code saved in the database that can be called by name. It can accept parameters, contain control flow logic, and modify data.

---

### Creating a Stored Procedure (MySQL)

```sql
DELIMITER //
CREATE PROCEDURE get_employees_by_dept(IN dept_name VARCHAR(50))
BEGIN
    SELECT id, first_name, last_name, salary
    FROM employees
    WHERE department = dept_name
    ORDER BY salary DESC;
END //
DELIMITER ;
```

- `DELIMITER //` is needed in MySQL because the procedure body contains semicolons. PostgreSQL and SQL Server don't need this.

---

### Calling a Procedure

```sql
CALL get_employees_by_dept('Engineering');
```

---

### Parameters

| Type    | Direction                | Use                              |
| ------- | ------------------------ | -------------------------------- |
| `IN`    | Input only (default)     | Pass values to the procedure     |
| `OUT`   | Output only              | Return values to the caller      |
| `INOUT` | Both                     | Pass in and get modified back    |

```sql
DELIMITER //
CREATE PROCEDURE count_employees(IN dept VARCHAR(50), OUT total INT)
BEGIN
    SELECT COUNT(*) INTO total FROM employees WHERE department = dept;
END //
DELIMITER ;

-- Usage:
CALL count_employees('Engineering', @result);
SELECT @result;
```

---

### Variables and Control Flow

```sql
DELIMITER //
CREATE PROCEDURE give_raises(IN dept VARCHAR(50), IN pct DECIMAL(5,2))
BEGIN
    DECLARE emp_count INT;
    
    SELECT COUNT(*) INTO emp_count FROM employees WHERE department = dept;
    
    IF emp_count > 0 THEN
        UPDATE employees
        SET salary = salary * (1 + pct / 100)
        WHERE department = dept;
    END IF;
END //
DELIMITER ;
```

- MySQL supports: `IF/ELSEIF/ELSE`, `CASE`, `WHILE`, `LOOP`, `REPEAT`.

---

### Stored Procedures vs Functions

| Feature                | Stored Procedure              | Function                        |
| ---------------------- | ----------------------------- | ------------------------------- |
| Returns                | Results via OUT params or result sets | A single value              |
| Usable in SELECT       | No                            | Yes (`SELECT my_func(col)`)     |
| Can modify data        | Yes                           | Depends on RDBMS (MySQL: yes, PostgreSQL: no for SQL functions) |
| Called with             | `CALL proc_name()`            | Used in expressions             |

---

### Triggers (Brief)

- A trigger is a stored procedure that **automatically fires** on `INSERT`, `UPDATE`, or `DELETE`:
```sql
CREATE TRIGGER before_employee_delete
BEFORE DELETE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (employee_id, action, action_date)
    VALUES (OLD.id, 'DELETE', NOW());
END;
```

- Use sparingly — triggers add hidden logic that can be hard to debug and can cause performance issues.

---

### Pros and Cons

| Pros                                      | Cons                                      |
| ----------------------------------------- | ----------------------------------------- |
| Reduce network round-trips                | Hard to version control (lives in DB)     |
| Encapsulate business logic                | Vendor lock-in (syntax varies)            |
| Security (grant EXECUTE only)             | Can become hard to debug and test         |
| Reusable across applications              | Can hide complexity                       |

```ad-tip
Modern development often favors keeping logic in application code (easier to test, version, deploy) and using stored procedures primarily for performance-critical batch operations or security-sensitive data access.
```
