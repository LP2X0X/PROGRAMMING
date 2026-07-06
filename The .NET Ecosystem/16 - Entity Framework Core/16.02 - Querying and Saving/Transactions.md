---
tags: [csharp, ef-core, transactions, saving]
---

- `SaveChanges()` already wraps all pending changes in a single database transaction. You only need **explicit transactions** when a single `SaveChanges()` call isn't enough — typically when you have multiple save points that must succeed or fail together.

---

## SaveChanges = Implicit Transaction

By default, every `SaveChanges()` call is atomic:

```csharp
db.Cars.Add(car1);
db.Cars.Add(car2);
db.Cars.Remove(oldCar);

db.SaveChanges();
// All three operations are wrapped in one transaction:
// BEGIN TRANSACTION
//   INSERT INTO Cars (...) VALUES (...)  -- car1
//   INSERT INTO Cars (...) VALUES (...)  -- car2
//   DELETE FROM Cars WHERE Id = ...      -- oldCar
// COMMIT
```

If any operation fails, **all** are rolled back. For most scenarios, this is sufficient. See [[CRUD Operations]] for the full `SaveChanges()` behavior.

---

## When You Need Explicit Transactions

1. **Multiple `SaveChanges()` calls** that must be atomic (e.g., save Order first to get its ID, then save OrderItems referencing that ID).
2. **Mixing EF Core with raw SQL** or ADO.NET commands in one atomic operation.
3. **Reading data that must remain consistent** across multiple queries (e.g., check balance then debit — no one should modify the balance in between).

---

## Using Database.BeginTransaction

```csharp
using var transaction = db.Database.BeginTransaction();

try
{
    // First operation
    var order = new Order { CustomerId = 1, Total = 250.00m };
    db.Orders.Add(order);
    db.SaveChanges();    // INSERT order, get generated Id

    // Second operation (depends on order.Id from above)
    var item = new OrderItem
    {
        OrderId = order.Id,     // now has the DB-generated value
        ProductId = 42,
        Quantity = 2
    };
    db.OrderItems.Add(item);
    db.SaveChanges();    // INSERT order item

    // Both succeeded — commit
    transaction.Commit();
}
catch (Exception)
{
    // Something failed — roll back both operations
    transaction.Rollback();
    throw;
}
```

```ad-note
title: Rollback is automatic if you don't commit
If the `transaction` is disposed without calling `Commit()` (e.g., an exception causes the `using` block to exit), the transaction is automatically rolled back. The explicit `Rollback()` in the catch block is technically redundant but makes the intent clear.
```

---

## Mixing EF Core with Raw SQL

Explicit transactions let you combine EF operations with raw ADO.NET commands in one atomic unit:

```csharp
using var transaction = db.Database.BeginTransaction();

try
{
    // EF Core operation
    var car = db.Cars.First(c => c.Id == 1);
    car.Color = "Red";
    db.SaveChanges();

    // Raw SQL operation (same transaction)
    db.Database.ExecuteSqlRaw(
        "UPDATE Inventory SET Stock = Stock - 1 WHERE CarId = {0}", car.Id);

    transaction.Commit();
}
catch
{
    transaction.Rollback();
    throw;
}
```

Both the EF update and the raw SQL run inside the same database transaction.

---

## Isolation Levels

You can control the isolation level to prevent read anomalies. For background on isolation levels and transaction anomalies, see [[Isolation Levels]].

```csharp
using var transaction = db.Database.BeginTransaction(
    System.Data.IsolationLevel.Serializable);

try
{
    // Check balance (no one can modify it until we commit)
    var account = db.Accounts.First(a => a.Id == fromId);

    if (account.Balance >= amount)
    {
        account.Balance -= amount;

        var target = db.Accounts.First(a => a.Id == toId);
        target.Balance += amount;

        db.SaveChanges();
        transaction.Commit();
    }
    else
    {
        transaction.Rollback();
        throw new InvalidOperationException("Insufficient funds");
    }
}
catch
{
    transaction.Rollback();
    throw;
}
```

| Isolation Level | Use case |
|---|---|
| `ReadCommitted` | Default for most databases. Good general-purpose level. |
| `RepeatableRead` | Prevent non-repeatable reads (re-reading gives same data). |
| `Serializable` | Maximum safety. Use for financial operations where consistency is critical. |
| `ReadUncommitted` | Fastest but can read uncommitted data. Rarely appropriate. |

```ad-warning
title: Higher isolation = more locking = more deadlocks
`Serializable` holds locks on all read rows until the transaction commits, which can block other transactions and cause deadlocks. Keep transactions short. See [[BEGIN, COMMIT, ROLLBACK]] for general transaction guidance.
```

---

## Savepoints (.NET 5+)

Savepoints let you roll back **part** of a transaction without aborting the whole thing:

```csharp
using var transaction = db.Database.BeginTransaction();

try
{
    db.Cars.Add(mainCar);
    db.SaveChanges();

    transaction.CreateSavepoint("AfterMainCar");

    try
    {
        db.Accessories.AddRange(accessories);
        db.SaveChanges();
    }
    catch
    {
        // Undo only the accessories insert, keep the car
        transaction.RollbackToSavepoint("AfterMainCar");
    }

    transaction.Commit();   // commits the car (and accessories if they succeeded)
}
catch
{
    transaction.Rollback();
    throw;
}
```

---

## TransactionScope (Distributed Transactions)

`TransactionScope` is a .NET class (not EF-specific) that can coordinate transactions across **multiple databases or resources**. EF Core supports it but it's rarely needed:

```csharp
using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);

// Operation on database 1
using (var db1 = new AppDbContext())
{
    db1.Cars.Add(newCar);
    db1.SaveChanges();
}

// Operation on database 2
using (var db2 = new InventoryDbContext())
{
    db2.Stock.Add(newStock);
    db2.SaveChanges();
}

// Both commit together (requires MSDTC on Windows for true 2PC)
scope.Complete();
```

```ad-warning
title: TransactionScope limitations
- Distributed transactions require **MSDTC** (Microsoft Distributed Transaction Coordinator) on Windows. Not available on Linux.
- Must pass `TransactionScopeAsyncFlowOption.Enabled` if using `async`/`await` inside the scope — otherwise the transaction doesn't flow across async continuations.
- Most modern architectures avoid distributed transactions in favor of eventual consistency patterns (e.g., outbox pattern, sagas).
```

---

## Practical Guidelines

1. **Start simple** — `SaveChanges()` handles most cases. Don't add explicit transactions unless you need them.
2. **Keep transactions short** — long transactions hold locks and hurt concurrency.
3. **Always use try/catch/rollback** — or rely on the `using` pattern for automatic rollback.
4. **Use async** in web apps — `BeginTransactionAsync()`, `SaveChangesAsync()`, `CommitAsync()`.
5. **Don't nest EF transactions** — calling `BeginTransaction()` while one is already active throws an exception. Use savepoints instead.

---

## See Also

- [[CRUD Operations]] — how SaveChanges() works as an implicit transaction
- [[Raw SQL]] — raw SQL inside explicit transactions
- [[BEGIN, COMMIT, ROLLBACK]] — SQL transaction fundamentals
- [[ACID Properties]] — the theory behind transactions
- [[Isolation Levels]] — controlling concurrent access
