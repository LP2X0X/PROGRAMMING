---
tags:
  - csharp
  - ado-net
  - disconnected-layer
aliases:
  - DataSet
  - DataTable
  - In-Memory Data
  - DataRow
  - DataColumn
  - DataRelation
---

## DataSet and DataTable

```ad-note
title: What You'll Learn
The disconnected layer in ADO.NET revolves around loading data from the database into **in-memory structures**, closing the connection, and then working with the data offline. This note covers the two central types — `DataSet` (an in-memory database) and `DataTable` (an in-memory table) — along with their supporting cast: `DataColumn`, `DataRow`, `DataRelation`, and `Constraint`. You'll learn how to create them manually, how change tracking works via `RowState` and row versions, and how to model multi-table relationships entirely in memory.
```

---

## Table of Contents

- [[#The Disconnected Model]]
- [[#DataTable — The In-Memory Table]]
  - [[#Creating a DataTable Manually]]
  - [[#Defining Columns (DataColumn)]]
  - [[#Primary Keys and Constraints]]
  - [[#Adding and Reading Rows (DataRow)]]
  - [[#Typed Accessors and DBNull Handling]]
- [[#DataRow State Tracking]]
  - [[#RowState Property]]
  - [[#DataRow Versions]]
  - [[#AcceptChanges and RejectChanges]]
  - [[#Editing Rows — BeginEdit, EndEdit, CancelEdit]]
- [[#DataSet — The In-Memory Database]]
  - [[#Creating a DataSet]]
  - [[#DataRelation — Parent-Child Relationships]]
  - [[#Navigating Relations]]
  - [[#Constraints]]
- [[#Serialization — XML and Schema]]
- [[#Typed DataSets]]
- [[#Performance Considerations]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## The Disconnected Model

The **disconnected model** in ADO.NET follows a specific workflow:

1. **Connect** — open a connection to the database
2. **Fetch** — execute a query and load results into an in-memory structure (`DataTable` or `DataSet`)
3. **Disconnect** — close the connection immediately
4. **Work offline** — read, filter, sort, and modify data entirely in memory
5. **Reconnect and sync** — open a connection again and push changes back to the database

This is fundamentally different from the [[ADO.NET Overview#Connected Layer|connected layer]], where the connection stays open while you stream through a `DbDataReader`.

The key benefit: ==your application holds database connections for the shortest possible time==, freeing them for other operations (critical in high-concurrency scenarios like web applications and services).

The in-memory object hierarchy looks like this:

```
DataSet ("MyDatabase")
├── Tables Collection
│   ├── DataTable "Users"
│   │   ├── Columns Collection
│   │   │   ├── DataColumn "Id"       (Type: Int32, PK, AutoIncrement)
│   │   │   ├── DataColumn "Name"     (Type: String, MaxLength: 100)
│   │   │   └── DataColumn "Age"      (Type: Int32, AllowDBNull: false)
│   │   ├── Rows Collection
│   │   │   ├── DataRow {1, "Long", 28}   RowState: Unchanged
│   │   │   └── DataRow {2, "Pham", 25}   RowState: Modified
│   │   ├── PrimaryKey: [Id]
│   │   └── Constraints: [PK_Users]
│   │
│   └── DataTable "Orders"
│       ├── Columns Collection
│       │   ├── DataColumn "OrderId"  (Type: Int32, PK)
│       │   ├── DataColumn "UserId"   (Type: Int32, FK)
│       │   └── DataColumn "Total"    (Type: Decimal)
│       └── Rows Collection
│           └── ...
│
├── Relations Collection
│   └── DataRelation "FK_Orders_Users"
│       (ParentTable: Users.Id → ChildTable: Orders.UserId)
│
└── ExtendedProperties Collection
```

```ad-note
title: Section Summary
- The disconnected model fetches data into memory, closes the connection, and works offline
- `DataSet` is the top-level container (in-memory database); `DataTable` is a single in-memory table
- Connections are held only during fetch and update — minimizing connection pool usage
- The hierarchy is: `DataSet` → `DataTable` → `DataColumn` (schema) + `DataRow` (data)
```

---

## DataTable — The In-Memory Table

A **`DataTable`** represents a single table of in-memory data. It has:

- A **`Columns`** collection (`DataColumnCollection`) — defines the schema
- A **`Rows`** collection (`DataRowCollection`) — holds the data
- **`PrimaryKey`** — an array of `DataColumn` objects that form the primary key
- **`Constraints`** collection — `UniqueConstraint` and `ForeignKeyConstraint`
- A **`DefaultView`** property — returns a `DataView` for sorting/filtering (see [[DataView]])

A `DataTable` can exist independently (without a `DataSet`) or inside one.

### Creating a DataTable Manually

```csharp
// Create a table with a name
var usersTable = new DataTable("Users");

// Without a name — TableName defaults to ""
var anonymous = new DataTable();
anonymous.TableName = "Orders"; // can be set later
```

The `TableName` is important when the table belongs to a `DataSet`, because you access tables by name:

```csharp
DataTable users = dataSet.Tables["Users"];
```

### Defining Columns (DataColumn)

Each `DataColumn` defines one column in the schema:

```csharp
var table = new DataTable("Users");

// Basic columns
table.Columns.Add("Id", typeof(int));
table.Columns.Add("Name", typeof(string));
table.Columns.Add("Age", typeof(int));
table.Columns.Add("Email", typeof(string));
table.Columns.Add("CreatedAt", typeof(DateTime));
```

`DataColumn` has many configurable properties:

| Property | Type | Description | Default |
|---|---|---|---|
| `ColumnName` | `string` | Name of the column | `""` |
| `DataType` | `Type` | CLR type (`typeof(int)`, `typeof(string)`, etc.) | `typeof(string)` |
| `AllowDBNull` | `bool` | Whether `DBNull.Value` is allowed | `true` |
| `DefaultValue` | `object` | Default value for new rows | `DBNull.Value` |
| `MaxLength` | `int` | Max length for string columns | `-1` (unlimited) |
| `Unique` | `bool` | Whether values must be unique | `false` |
| `AutoIncrement` | `bool` | Auto-generate incrementing values | `false` |
| `AutoIncrementSeed` | `long` | Starting value for auto-increment | `0` |
| `AutoIncrementStep` | `long` | Increment amount | `1` |
| `ReadOnly` | `bool` | Whether column values can be modified | `false` |
| `Expression` | `string` | Computed column expression | `""` |
| `Caption` | `string` | Display name (for data binding) | `ColumnName` |
| `Ordinal` | `int` | Zero-based position in the collection | (auto-assigned) |

Example with full configuration:

```csharp
var table = new DataTable("Users");

// Auto-increment primary key
var idCol = new DataColumn("Id", typeof(int))
{
    AutoIncrement = true,
    AutoIncrementSeed = 1,
    AutoIncrementStep = 1,
    AllowDBNull = false,
    ReadOnly = true
};
table.Columns.Add(idCol);

// Required string column with max length
var nameCol = new DataColumn("Name", typeof(string))
{
    AllowDBNull = false,
    MaxLength = 100
};
table.Columns.Add(nameCol);

// Optional email with uniqueness
var emailCol = new DataColumn("Email", typeof(string))
{
    AllowDBNull = true,
    Unique = true,
    MaxLength = 255
};
table.Columns.Add(emailCol);

// Column with default value
var activeCol = new DataColumn("IsActive", typeof(bool))
{
    DefaultValue = true
};
table.Columns.Add(activeCol);

// Computed column — expression-based
var displayCol = new DataColumn("Display", typeof(string))
{
    Expression = "Name + ' <' + Email + '>'"
};
table.Columns.Add(displayCol);

// Timestamp with default
var createdCol = new DataColumn("CreatedAt", typeof(DateTime))
{
    DefaultValue = DateTime.Now
};
table.Columns.Add(createdCol);
```

```ad-info
title: Computed Columns (Expression)
A `DataColumn` with an `Expression` is a **computed column** — its value is calculated from other columns, similar to a computed column in SQL. The expression syntax supports arithmetic (`+`, `-`, `*`, `/`), string concatenation (`+`), comparison operators, and aggregate functions (`SUM`, `COUNT`, `AVG`, `MIN`, `MAX`).

Examples:
- `"Price * Quantity"` — numeric calculation
- `"FirstName + ' ' + LastName"` — string concatenation
- `"IIF(Age >= 18, 'Adult', 'Minor')"` — conditional
- `"SUM(Child(FK_OrderItems).LineTotal)"` — aggregate over child relation

Computed columns are **read-only** and recalculate automatically when the source columns change.
```

### Primary Keys and Constraints

```csharp
// Single-column primary key
table.PrimaryKey = new[] { table.Columns["Id"]! };

// Composite primary key
table.PrimaryKey = new[] 
{ 
    table.Columns["OrderId"]!, 
    table.Columns["ProductId"]! 
};
```

Setting `PrimaryKey` automatically creates a `UniqueConstraint` on those columns and sets `AllowDBNull = false` for each column in the key.

```ad-warning
title: PrimaryKey Enables Find()
Without a primary key defined, calling `table.Rows.Find()` throws an `MissingPrimaryKeyException`. Always set `PrimaryKey` if you intend to look up rows by key value.
```

You can also add constraints explicitly:

```csharp
// Unique constraint (not a primary key)
table.Constraints.Add(new UniqueConstraint("UQ_Email", table.Columns["Email"]!));

// Unique constraint on multiple columns
table.Constraints.Add(new UniqueConstraint("UQ_Name_Age",
    new[] { table.Columns["Name"]!, table.Columns["Age"]! }));
```

### Adding and Reading Rows (DataRow)

There are several ways to add rows:

```csharp
var table = new DataTable("Users");
table.Columns.Add("Id", typeof(int));
table.Columns.Add("Name", typeof(string));
table.Columns.Add("Age", typeof(int));
table.PrimaryKey = new[] { table.Columns["Id"]! };

// Method 1: Rows.Add with positional values (order matches column order)
table.Rows.Add(1, "Long", 28);
table.Rows.Add(2, "Pham", 25);

// Method 2: Create a DataRow, set values, then add
DataRow row = table.NewRow();      // creates a Detached row with the table's schema
row["Id"] = 3;
row["Name"] = "Alice";
row["Age"] = 30;
table.Rows.Add(row);              // RowState changes: Detached → Added

// Method 3: Using an object array
table.Rows.Add(new object[] { 4, "Bob", 22 });

// Method 4: ImportRow — copies a row from another table (preserves RowState)
table.ImportRow(otherTable.Rows[0]);
```

Reading rows:

```csharp
// Iterate all rows
foreach (DataRow r in table.Rows)
{
    Console.WriteLine($"{r["Id"]}: {r["Name"]}, age {r["Age"]}");
}

// Access by index
DataRow first = table.Rows[0];
Console.WriteLine(first["Name"]);     // access by column name
Console.WriteLine(first[1]);          // access by ordinal (zero-based)

// Find by primary key (requires PrimaryKey to be set)
DataRow? found = table.Rows.Find(2);  // find row where PK = 2
if (found != null)
{
    Console.WriteLine(found["Name"]); // "Pham"
}

// Find by composite key
DataRow? composite = table.Rows.Find(new object[] { orderId, productId });

// Select — filter rows using an expression (returns DataRow[])
DataRow[] adults = table.Select("Age >= 18");
DataRow[] sorted = table.Select("Age > 20", "Name ASC");
DataRow[] addedRows = table.Select("", "", DataViewRowState.Added);
```

```ad-warning
title: Column Access Returns object
`row["ColumnName"]` returns `object`. You must cast to the expected type. For `NULL` database values, the value is `DBNull.Value`, **not** C# `null`. Casting `DBNull.Value` to a value type throws an `InvalidCastException`.
```

### Typed Accessors and DBNull Handling

Since column access returns `object`, you need careful casting:

```csharp
DataRow row = table.Rows[0];

// Direct cast — throws if DBNull
int id = (int)row["Id"];
string name = (string)row["Name"];

// Safe DBNull check
string? email = row["Email"] == DBNull.Value ? null : (string)row["Email"];
int? age = row.IsNull("Age") ? null : (int)row["Age"];

// Using the Field<T> extension method (System.Data.DataSetExtensions)
// Throws if DBNull for non-nullable T, returns null for nullable T
int id2 = row.Field<int>("Id");           // throws if DBNull
string? email2 = row.Field<string?>("Email"); // returns null if DBNull
int? age2 = row.Field<int?>("Age");       // returns null if DBNull

// Setting values with SetField<T>
row.SetField<string?>("Email", null);     // sets DBNull.Value
row.SetField("Name", "Updated");
```

```ad-important
title: Prefer Field<T> and SetField<T>
The `Field<T>()` and `SetField<T>()` extension methods (from `System.Data.DataSetExtensions` / built into modern .NET) are the ==preferred way to access column values==. They provide generic type safety and handle nullable types correctly — `Field<int?>()` returns `null` instead of `DBNull.Value`. Always use them over raw casts.
```

```ad-note
title: Section Summary
- `DataTable` contains `DataColumn` objects (schema) and `DataRow` objects (data)
- Columns have rich configuration: auto-increment, defaults, max length, computed expressions, nullability
- Setting `PrimaryKey` enables `Rows.Find()` for key-based lookups and adds a `UniqueConstraint`
- Column access returns `object` — use `Field<T>()` / `SetField<T>()` for type-safe, null-safe access
- `DBNull.Value` represents SQL NULL in ADO.NET — it is not the same as C# `null`
```

---

## DataRow State Tracking

One of the most powerful features of the disconnected layer is ==automatic change tracking==. Every `DataRow` tracks its own state, and the [[DataAdapter]] uses this state information to generate the correct `INSERT`, `UPDATE`, or `DELETE` SQL when syncing changes back to the database.

### RowState Property

Every `DataRow` has a `RowState` property of type `DataRowState`:

| RowState | Meaning | When It Occurs |
|---|---|---|
| `Detached` | The row is not attached to any `DataTable` | After `table.NewRow()` but before `table.Rows.Add(row)` |
| `Added` | New row, exists in the table but not yet in the database | After `table.Rows.Add(row)` |
| `Unchanged` | Row matches the database (no pending changes) | After `Fill()` loads data, or after `AcceptChanges()` |
| `Modified` | One or more column values have been changed | After modifying a column value on an `Unchanged` row |
| `Deleted` | Row is marked for deletion (still in the collection) | After calling `row.Delete()` |

Lifecycle of a row:

```csharp
var table = new DataTable("Users");
table.Columns.Add("Id", typeof(int));
table.Columns.Add("Name", typeof(string));
table.PrimaryKey = new[] { table.Columns["Id"]! };

// After NewRow() — Detached
DataRow row = table.NewRow();
Console.WriteLine(row.RowState);    // Detached

// After Rows.Add() — Added
row["Id"] = 1;
row["Name"] = "Long";
table.Rows.Add(row);
Console.WriteLine(row.RowState);    // Added

// After AcceptChanges() — Unchanged
table.AcceptChanges();
Console.WriteLine(row.RowState);    // Unchanged

// After modifying a value — Modified
row["Name"] = "Updated";
Console.WriteLine(row.RowState);    // Modified

// After Delete() — Deleted
row.Delete();
Console.WriteLine(row.RowState);    // Deleted

// After AcceptChanges() on a Deleted row — the row is removed from the collection
table.AcceptChanges();
Console.WriteLine(table.Rows.Count); // 0
```

```ad-warning
title: Delete() vs Remove()
- `row.Delete()` marks the row as `Deleted` — it stays in the `Rows` collection so the [[DataAdapter]] can generate a `DELETE` statement during `Update()`
- `table.Rows.Remove(row)` **permanently removes** the row from the collection — the adapter will never know it existed, so no `DELETE` is sent to the database

==Always use `Delete()` if you want the deletion to be synced back to the database.== Use `Remove()` only when you want to discard a row without any database effect (e.g., removing a row that was `Added` and never saved).
```

### DataRow Versions

Each `DataRow` can hold **multiple versions** of its data simultaneously. These versions are what enable change tracking — you can compare the original value (from the database) with the current value (after user modifications).

| Version | Available When | Contains |
|---|---|---|
| `Current` | `RowState` is `Added`, `Modified`, or `Unchanged` | The current column values |
| `Original` | `RowState` is `Modified`, `Unchanged`, or `Deleted` | The values before any changes (from last `AcceptChanges` or `Fill`) |
| `Proposed` | Inside a `BeginEdit()` / `EndEdit()` block | The tentative values being edited |
| `Default` | Always | Returns `Proposed` if in edit mode, otherwise `Current` |

```csharp
// After Fill() or AcceptChanges(), row is Unchanged
// Current == Original
Console.WriteLine(row["Name", DataRowVersion.Current]);    // "Long"
Console.WriteLine(row["Name", DataRowVersion.Original]);   // "Long"

// Modify the row
row["Name"] = "Updated";
// Now RowState == Modified, and versions diverge
Console.WriteLine(row["Name", DataRowVersion.Current]);    // "Updated"
Console.WriteLine(row["Name", DataRowVersion.Original]);   // "Long"

// Check if a specific version exists
bool hasOriginal = row.HasVersion(DataRowVersion.Original); // true
bool hasProposed = row.HasVersion(DataRowVersion.Proposed); // false (not in edit mode)
```

```ad-info
title: How the DataAdapter Uses Versions
When the [[DataAdapter]] calls `Update()`, it uses row versions to build parameterized commands:
- **INSERT** — uses `Current` version values for the new row
- **UPDATE** — uses `Current` for the new values and `Original` for the `WHERE` clause (optimistic concurrency)
- **DELETE** — uses `Original` values for the `WHERE` clause

This is why the `Original` version is so important — it's the basis for optimistic concurrency checks ("update this row only if it still has the values I originally read").
```

### AcceptChanges and RejectChanges

These two methods control whether pending modifications are committed or rolled back ==in memory== (not in the database):

**`AcceptChanges()`** — confirms all pending changes:

| Before | After AcceptChanges() |
|---|---|
| `Added` → | `Unchanged` (Original version set to current values) |
| `Modified` → | `Unchanged` (Original version updated to current values) |
| `Deleted` → | Row is **removed** from the `Rows` collection entirely |
| `Unchanged` → | `Unchanged` (no change) |

**`RejectChanges()`** — reverts all pending changes:

| Before | After RejectChanges() |
|---|---|
| `Added` → | Row is **removed** from the `Rows` collection |
| `Modified` → | `Unchanged` (Current version reverted to Original) |
| `Deleted` → | `Unchanged` (row is un-deleted) |
| `Unchanged` → | `Unchanged` (no change) |

```csharp
// AcceptChanges — available at three levels
row.AcceptChanges();    // single row
table.AcceptChanges();  // all rows in the table
dataSet.AcceptChanges(); // all rows in all tables

// RejectChanges — same three levels
row.RejectChanges();
table.RejectChanges();
dataSet.RejectChanges();
```

```ad-warning
title: Do NOT Call AcceptChanges Before DataAdapter.Update()
This is a ==critical mistake==. If you call `AcceptChanges()` before `adapter.Update(table)`, all rows become `Unchanged`, and the adapter sees nothing to update — **no SQL is generated**, and your changes are lost (never sent to the database).

The correct order is:
1. Modify rows (they become `Added`/`Modified`/`Deleted`)
2. Call `adapter.Update(table)` — adapter reads `RowState`, generates SQL, executes it
3. The adapter automatically calls `AcceptChanges()` on each successfully updated row

Only call `AcceptChanges()` manually when you are working with `DataTable` purely in memory (no database sync).
```

### Editing Rows — BeginEdit, EndEdit, CancelEdit

For more controlled editing (especially useful with data-bound UI controls), use the edit transaction methods:

```csharp
row.BeginEdit();                // creates the Proposed version
row["Name"] = "Tentative";     // changes go into Proposed, not Current
row["Age"] = 99;

// At this point:
Console.WriteLine(row["Name", DataRowVersion.Current]);   // still "Long"
Console.WriteLine(row["Name", DataRowVersion.Proposed]);  // "Tentative"

row.EndEdit();                  // Proposed → Current, RowState becomes Modified
// OR
row.CancelEdit();               // Proposed is discarded, Current unchanged
```

```ad-note
title: Why Use BeginEdit/EndEdit?
In data-binding scenarios (WinForms/WPF), `BeginEdit()` suppresses column-change and constraint-validation events until `EndEdit()` is called. Without it, every individual column assignment fires events and checks constraints immediately — which can cause UI flicker, validation errors on partially-filled rows, and performance issues when modifying multiple columns.
```

```ad-note
title: Section Summary
- `RowState` tracks whether each row is `Added`, `Modified`, `Deleted`, `Unchanged`, or `Detached`
- `DataRow` stores multiple versions (`Original`, `Current`, `Proposed`) to support change tracking
- `AcceptChanges()` commits in-memory changes; `RejectChanges()` rolls them back — neither touches the database
- ==Never call `AcceptChanges()` before `adapter.Update()`== or your changes will be silently lost
- Use `Delete()` (not `Remove()`) to mark rows for database deletion
- `BeginEdit()` / `EndEdit()` provides transactional editing with deferred validation
```

---

## DataSet — The In-Memory Database

A **`DataSet`** is an in-memory relational database. It can hold multiple `DataTable` objects and define relationships between them using `DataRelation` objects. Think of it as a lightweight, disconnected snapshot of part of your database schema.

### Creating a DataSet

```csharp
// Create with a name (appears in XML serialization)
var ds = new DataSet("CompanyDB");

// Create and populate tables
var usersTable = new DataTable("Users");
usersTable.Columns.Add("Id", typeof(int));
usersTable.Columns.Add("Name", typeof(string));
usersTable.PrimaryKey = new[] { usersTable.Columns["Id"]! };
usersTable.Rows.Add(1, "Long");
usersTable.Rows.Add(2, "Pham");

var ordersTable = new DataTable("Orders");
ordersTable.Columns.Add("OrderId", typeof(int));
ordersTable.Columns.Add("UserId", typeof(int));
ordersTable.Columns.Add("Total", typeof(decimal));
ordersTable.PrimaryKey = new[] { ordersTable.Columns["OrderId"]! };
ordersTable.Rows.Add(100, 1, 49.99m);
ordersTable.Rows.Add(101, 1, 19.99m);
ordersTable.Rows.Add(102, 2, 99.99m);

// Add tables to the DataSet
ds.Tables.Add(usersTable);
ds.Tables.Add(ordersTable);

// Access tables
DataTable users = ds.Tables["Users"]!;    // by name
DataTable orders = ds.Tables[1];          // by index
```

### DataRelation — Parent-Child Relationships

A `DataRelation` defines a parent-child (one-to-many) relationship between two tables, similar to a foreign key in SQL:

```csharp
// Define a relation: Users.Id (parent) → Orders.UserId (child)
var relation = new DataRelation(
    "FK_Orders_Users",                    // relation name
    usersTable.Columns["Id"]!,            // parent column
    ordersTable.Columns["UserId"]!        // child column
);
ds.Relations.Add(relation);

// Multi-column relation (composite key)
var compositeRelation = new DataRelation(
    "FK_OrderItems_Orders",
    new[] { ordersTable.Columns["OrderId"]!, ordersTable.Columns["LineId"]! },  // parent
    new[] { itemsTable.Columns["OrderId"]!, itemsTable.Columns["LineId"]! }     // child
);
ds.Relations.Add(compositeRelation);
```

Adding a `DataRelation` automatically creates:

- A `ForeignKeyConstraint` on the child table
- A `UniqueConstraint` on the parent table (if one doesn't already exist)

### Navigating Relations

Once relations are defined, you can navigate between parent and child rows:

```csharp
// Get all child rows for a parent
DataRow user = usersTable.Rows[0]; // User "Long"
DataRow[] userOrders = user.GetChildRows("FK_Orders_Users");

foreach (DataRow order in userOrders)
{
    Console.WriteLine($"Order {order["OrderId"]}: ${order["Total"]}");
}
// Output:
// Order 100: $49.99
// Order 101: $19.99

// Get the parent row for a child
DataRow order102 = ordersTable.Rows[2]; // Order 102
DataRow parentUser = order102.GetParentRow("FK_Orders_Users")!;
Console.WriteLine(parentUser["Name"]); // "Pham"

// Get parent rows (for many-to-many via junction table)
DataRow[] parents = childRow.GetParentRows("RelationName");
```

```ad-info
title: Navigating Relations with Row Versions
`GetChildRows` and `GetParentRow` accept an optional `DataRowVersion` parameter, letting you navigate relationships based on original or current values:

`DataRow[] originalOrders = user.GetChildRows("FK_Orders_Users", DataRowVersion.Original);`

This is useful when a child row's foreign key value has been modified — you can still find its original parent.
```

### Constraints

`DataSet` supports two types of constraints, and enforces them by default:

**`UniqueConstraint`** — ensures column values are unique:

```csharp
// Explicitly add (PrimaryKey and DataRelation create these automatically)
usersTable.Constraints.Add(new UniqueConstraint("UQ_Email", usersTable.Columns["Email"]!));
```

**`ForeignKeyConstraint`** — enforces referential integrity and defines cascade rules:

```csharp
var fk = new ForeignKeyConstraint(
    "FK_Orders_Users",
    usersTable.Columns["Id"]!,       // parent
    ordersTable.Columns["UserId"]!   // child
);

// Cascade behavior (like SQL ON DELETE / ON UPDATE)
fk.DeleteRule = Rule.Cascade;        // Delete parent → delete children
fk.UpdateRule = Rule.Cascade;        // Update parent key → update children FK
fk.AcceptRejectRule = AcceptRejectRule.Cascade; // AcceptChanges cascades

ds.Tables["Orders"]!.Constraints.Add(fk);
```

| Rule | Behavior |
|---|---|
| `Rule.Cascade` | Propagate the change to child rows (default for `UpdateRule`) |
| `Rule.SetNull` | Set child FK columns to `DBNull` |
| `Rule.SetDefault` | Set child FK columns to their `DefaultValue` |
| `Rule.None` | Throw a `ConstraintException` if children exist (default for `DeleteRule`) |

```ad-note
title: EnforceConstraints
`DataSet.EnforceConstraints` (default: `true`) controls whether constraints are checked. Setting it to `false` temporarily disables all constraint validation — useful during bulk loading where intermediate states may violate constraints. Always set it back to `true` after loading.

```csharp
ds.EnforceConstraints = false;
// ... bulk load data ...
ds.EnforceConstraints = true; // validates all constraints; throws if any are violated
```
```

```ad-note
title: Section Summary
- `DataSet` holds multiple `DataTable` objects and `DataRelation` objects — an in-memory relational model
- `DataRelation` defines parent-child relationships; `GetChildRows()` and `GetParentRow()` navigate them
- Adding a relation automatically creates `ForeignKeyConstraint` and `UniqueConstraint`
- Cascade rules (`DeleteRule`, `UpdateRule`) control how changes propagate to related rows
- `EnforceConstraints` can be temporarily disabled during bulk loading
```

---

## Serialization — XML and Schema

`DataSet` and `DataTable` have built-in XML serialization, which was a major design feature when ADO.NET was introduced:

```csharp
// Write DataSet to XML
ds.WriteXml(@"C:\data\company.xml");                          // data only
ds.WriteXml(@"C:\data\company.xml", XmlWriteMode.WriteSchema); // data + schema

// Write schema only
ds.WriteXmlSchema(@"C:\data\company.xsd");

// Read back
var ds2 = new DataSet();
ds2.ReadXml(@"C:\data\company.xml");
ds2.ReadXmlSchema(@"C:\data\company.xsd");

// DataTable also supports XML serialization independently
table.WriteXml(@"C:\data\users.xml");
table.ReadXml(@"C:\data\users.xml");

// Serialize to string
using var writer = new StringWriter();
ds.WriteXml(writer);
string xml = writer.ToString();
```

The `DiffGram` format captures both original and modified data, making it useful for syncing changes:

```csharp
// Write changes only (DiffGram captures RowState and versions)
ds.WriteXml(@"C:\data\changes.xml", XmlWriteMode.DiffGram);

// GetChanges() — returns a new DataSet containing only modified rows
DataSet? changes = ds.GetChanges();                       // all changes
DataSet? addedOnly = ds.GetChanges(DataRowState.Added);   // only added rows
DataSet? modifiedOnly = ds.GetChanges(DataRowState.Modified);

if (changes != null)
{
    // Send only the changes to the server (smaller payload)
    adapter.Update(changes);
}
```

```ad-note
title: Section Summary
- `DataSet` and `DataTable` support built-in XML serialization (`WriteXml` / `ReadXml`)
- `DiffGram` format preserves `RowState` and row versions for change synchronization
- `GetChanges()` returns a lightweight `DataSet` with only modified rows — useful for sending minimal data over the wire
```

---

## Typed DataSets

A **Typed DataSet** is a class generated from an XSD schema that provides strongly-typed access to tables, columns, and rows — replacing string-based indexing with real properties.

```csharp
// Untyped — string-based, error-prone, no IntelliSense
string name = (string)dataSet.Tables["Users"]!.Rows[0]["Name"];

// Typed — strongly-typed, compile-time checked, IntelliSense
string name = typedDataSet.Users[0].Name;
```

Typed DataSets are generated by:

1. Adding an `.xsd` file to a Visual Studio project
2. Using the `xsd.exe` tool: `xsd.exe schema.xsd /dataset`

```ad-note
title: Modern Perspective on Typed DataSets
Typed DataSets were popular in .NET Framework 2.0-4.x (mid-2000s to 2010s), especially with the Visual Studio DataSet Designer. In modern .NET development, they are ==rarely used in new projects== — POCOs with Dapper or EF Core entities serve the same purpose with less ceremony. However, you will encounter them in legacy codebases, especially WinForms applications. The `xsd.exe` tool is still available but is a .NET Framework-only tool.
```

```ad-note
title: Section Summary
- Typed DataSets provide compile-time safety and IntelliSense by generating classes from XSD schemas
- They were common in .NET Framework era; modern code prefers POCOs + Dapper or EF Core entities
- Still encountered in legacy WinForms and WebForms applications
```

---

## Performance Considerations

`DataTable` and `DataSet` load ==all data into memory==. Keep these points in mind:

| Concern | Details |
|---|---|
| **Memory** | Each `DataRow` has significant overhead: multiple version storage, state tracking, original/current value arrays. A 1-million-row `DataTable` can consume several GB of memory. |
| **Large result sets** | For large data volumes, prefer `DbDataReader` (connected layer) which streams one row at a time. |
| **Find vs Select** | `Rows.Find()` (primary key lookup) is O(1) using an internal hash table. `Select()` (expression filter) scans all rows — O(n). |
| **DataView** | For repeated filtering/sorting, use a [[DataView]] with `Sort` set — `Find()` on a sorted `DataView` uses binary search. |
| **BeginLoadData / EndLoadData** | For bulk loading, call `table.BeginLoadData()` before and `table.EndLoadData()` after to disable indexing, constraint checking, and notifications. |
| **Merge** | `DataSet.Merge()` and `DataTable.Merge()` can combine data from multiple sources but can be expensive with large tables. |

```csharp
// Bulk loading optimization
table.BeginLoadData();
try
{
    for (int i = 0; i < 100_000; i++)
    {
        table.Rows.Add(i, $"User{i}", i % 100);
    }
}
finally
{
    table.EndLoadData(); // re-enables indexing, checks constraints
}
```

```ad-note
title: Section Summary
- `DataTable` / `DataSet` consume significant memory — not suitable for very large result sets
- `Rows.Find()` is O(1) (hash table); `Select()` is O(n) (full scan)
- Use `BeginLoadData()` / `EndLoadData()` to optimize bulk loading
- For large data volumes, prefer `DbDataReader` in the [[ADO.NET Overview#Connected Layer|connected layer]]
```

---

## Comprehensive Summary

```ad-important
title: Key Takeaways
**DataTable** is an in-memory table with `DataColumn` objects defining the schema and `DataRow` objects holding the data. **DataSet** is an in-memory relational database that contains multiple `DataTable` objects and `DataRelation` objects defining parent-child relationships.

**Change tracking** is the central feature of the disconnected layer. Every `DataRow` has a `RowState` (`Added`, `Modified`, `Deleted`, `Unchanged`, `Detached`) and maintains multiple row versions (`Original`, `Current`, `Proposed`). The [[DataAdapter]] uses this state to generate the correct `INSERT`, `UPDATE`, and `DELETE` SQL during synchronization.

**Critical rules:**
- ==Never call `AcceptChanges()` before `adapter.Update()`== — it resets all states to `Unchanged` and the adapter sees nothing to sync
- Use `row.Delete()` (not `table.Rows.Remove()`) to mark rows for database deletion
- Use `Field<T>()` and `SetField<T>()` for type-safe, null-safe column access
- `DBNull.Value` represents SQL NULL — it is not C# `null`

**DataRelation** enables in-memory navigation between parent and child rows via `GetChildRows()` and `GetParentRow()`, with configurable cascade rules for delete and update operations.

**Performance**: `DataTable` loads all data into memory with significant per-row overhead. Use `Rows.Find()` (O(1)) over `Select()` (O(n)) for key lookups. For large result sets, prefer `DbDataReader` in the connected layer. Use `BeginLoadData()` / `EndLoadData()` for bulk loading optimization.

**Modern context**: `DataSet`/`DataTable` is the classic ADO.NET approach. Modern .NET apps typically use POCOs + Dapper or EF Core. However, `DataSet`/`DataTable` remains relevant for reporting, data import/export, dynamic schemas, legacy codebases, and scenarios requiring an in-memory relational model.
```

---

## Related Topics

- [[DataAdapter]] — the bridge that fills `DataTable` from the database and pushes changes back
- [[DataView]] — sortable, filterable view over a `DataTable`
- [[Connected vs Disconnected Layer]] — when to use each approach
- [[ADO.NET Overview]] — the overall ADO.NET architecture
- [[DbDataReader]] — the connected-layer alternative for streaming reads
- [[Parameters and SQL Injection]] — parameterized queries for safe database interaction
- [[Entity Framework Core]] — the modern ORM alternative to `DataSet`/`DataTable`
