---
tags:
  - csharp
  - ado-net
  - disconnected-layer
aliases:
  - DataView
  - DataRowView
  - In-Memory View
---

## DataView

```ad-note
title: What You'll Learn
A `DataView` provides a ==sortable, filterable, bindable view== over a `DataTable` without modifying the underlying data. It is the in-memory equivalent of a SQL `VIEW`. This note covers creating views, sorting with `Sort`, filtering with `RowFilter` and `RowStateFilter`, finding rows, editing through `DataRowView`, creating multiple views on the same table, LINQ integration via `AsDataView()`, and data binding in WinForms/WPF.
```

---

## Table of Contents

- [[#What is a DataView?]]
- [[#Creating a DataView]]
- [[#Sorting]]
- [[#Filtering with RowFilter]]
  - [[#RowFilter Expression Syntax]]
  - [[#Common Filter Patterns]]
- [[#Filtering by Row State]]
- [[#Iterating and Accessing Rows]]
- [[#Finding Rows]]
- [[#Editing Through DataRowView]]
- [[#Adding and Deleting Rows Through DataView]]
- [[#Multiple Views on One Table]]
- [[#LINQ to DataSet and AsDataView]]
- [[#Data Binding]]
- [[#Performance Considerations]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## What is a DataView?

A `DataView` is a **lightweight window** over a [[DataSet and DataTable|DataTable]]. It does not copy data — it maintains an internal index of pointers into the underlying `DataTable`'s rows, filtered and sorted according to your specifications.

Key characteristics:

| Feature | Details |
|---|---|
| **Data ownership** | Does NOT copy data — references the underlying `DataTable` |
| **Sorting** | Sorts rows by one or more columns without modifying the table |
| **Filtering** | Filters rows by expression or row state without removing rows from the table |
| **Data binding** | The primary mechanism for binding `DataTable` to UI controls (WinForms, WPF) |
| **Editability** | Can add, modify, and delete rows through `DataRowView` (changes affect the underlying table) |
| **Multiple views** | You can create multiple `DataView` objects on the same `DataTable`, each with different sort/filter |

The relationship:

```
DataTable "Users"
├── DataRow {1, "Long", 28}
├── DataRow {2, "Alice", 17}
├── DataRow {3, "Bob", 35}
├── DataRow {4, "Pham", 25}
└── DataRow {5, "Charlie", 22}

DataView (RowFilter: "Age >= 18", Sort: "Name ASC")
├── Index[0] → DataRow {3, "Bob", 35}        (pointer, not copy)
├── Index[1] → DataRow {5, "Charlie", 22}     (pointer, not copy)
├── Index[2] → DataRow {1, "Long", 28}        (pointer, not copy)
└── Index[3] → DataRow {4, "Pham", 25}        (pointer, not copy)
```

```ad-note
title: Section Summary
- `DataView` is a sorted/filtered window over a `DataTable` — it does not copy data
- Multiple views can reference the same table with different sort/filter criteria
- Changes through a `DataView` affect the underlying `DataTable` directly
```

---

## Creating a DataView

There are three ways to create a `DataView`:

```csharp
// Method 1: Constructor with DataTable
var view = new DataView(usersTable);

// Method 2: Constructor with sort, filter, and row state
var view = new DataView(
    usersTable,                          // source table
    "Age > 20",                          // RowFilter expression
    "Name ASC",                          // Sort expression
    DataViewRowState.CurrentRows         // RowStateFilter
);

// Method 3: DefaultView property — every DataTable has one
DataView defaultView = usersTable.DefaultView;
```

```ad-warning
title: DefaultView is a Shared Singleton
`table.DefaultView` always returns the ==same `DataView` instance==. If you set `Sort` or `RowFilter` on it, those settings affect every consumer of `DefaultView` (including data-bound controls). If you need independent filter/sort settings, create a **new** `DataView(table)` instead of using `DefaultView`.
```

```ad-note
title: Section Summary
- Create with `new DataView(table)`, with parameters, or via `table.DefaultView`
- `DefaultView` is a shared singleton — changes to it affect all consumers
- Use `new DataView(table)` for independent sort/filter settings
```

---

## Sorting

The `Sort` property accepts a comma-separated list of column names with optional `ASC` (default) or `DESC` direction:

```csharp
var view = new DataView(usersTable);

// Single column sort (ascending is the default)
view.Sort = "Name";
view.Sort = "Name ASC";     // explicit ascending
view.Sort = "Name DESC";    // descending

// Multi-column sort
view.Sort = "Age DESC, Name ASC";

// Clear sort (revert to table's natural order)
view.Sort = "";
```

Important details:

- Sorting is ==case-insensitive by default== (follows the `DataTable.CaseSensitive` property, which defaults to `false`)
- Setting `Sort` rebuilds the internal index — it is not free, but it is O(n log n) and typically fast for moderate-size tables
- After setting `Sort`, the view's `Find()` and `FindRows()` methods use the sort columns as lookup keys

```ad-note
title: Section Summary
- `Sort` accepts column names with `ASC`/`DESC`, comma-separated for multi-column sorts
- Setting `Sort` rebuilds the internal index and enables efficient `Find()` / `FindRows()`
- Sorting is case-insensitive by default (controlled by `DataTable.CaseSensitive`)
```

---

## Filtering with RowFilter

The `RowFilter` property accepts an expression string that follows a SQL-like syntax. Only rows matching the expression are visible in the view:

```csharp
var view = new DataView(usersTable);

// Basic comparison
view.RowFilter = "Age > 25";
view.RowFilter = "Name = 'Long'";

// Clear filter (show all rows)
view.RowFilter = "";
```

### RowFilter Expression Syntax

The expression syntax is defined by the `DataColumn.Expression` property grammar (the same syntax used for computed columns):

**Comparison operators:**

| Operator | Example |
|---|---|
| `=` | `"Age = 25"` |
| `<>` | `"Name <> 'Test'"` |
| `<`, `>` | `"Age > 18"` |
| `<=`, `>=` | `"Age >= 21"` |

**Logical operators:**

| Operator | Example |
|---|---|
| `AND` | `"Age > 18 AND Age < 65"` |
| `OR` | `"Country = 'US' OR Country = 'UK'"` |
| `NOT` | `"NOT (Age < 18)"` |

**String operators:**

| Operator | Example | Matches |
|---|---|---|
| `LIKE` | `"Name LIKE 'L%'"` | Starts with 'L' |
| `LIKE` | `"Name LIKE '%ham'"` | Ends with 'ham' |
| `LIKE` | `"Name LIKE '%on%'"` | Contains 'on' |
| `LIKE` | `"Name LIKE 'L_ng'"` | 'L' + any single char + 'ng' |

**Wildcard characters for `LIKE`:**

| Wildcard | Meaning |
|---|---|
| `%` | Zero or more characters |
| `*` | Same as `%` (both work) |
| `_` | Exactly one character |

**NULL handling:**

```csharp
// Check for NULL (DBNull)
view.RowFilter = "Email IS NULL";
view.RowFilter = "Email IS NOT NULL";

// IIF for conditional logic
view.RowFilter = "IIF(Email IS NULL, 'Missing', Email) <> 'Missing'";
```

**IN operator:**

```csharp
view.RowFilter = "Country IN ('US', 'UK', 'CA')";
view.RowFilter = "Id IN (1, 2, 3, 4, 5)";
```

**Arithmetic and functions:**

```csharp
// Arithmetic
view.RowFilter = "Price * Quantity > 100";

// String functions
view.RowFilter = "LEN(Name) > 5";
view.RowFilter = "SUBSTRING(Name, 1, 1) = 'L'";
view.RowFilter = "TRIM(Name) <> ''";

// Date functions
view.RowFilter = "CreatedAt > #2024-01-01#";   // date literals use # delimiters
view.RowFilter = "CreatedAt > #01/15/2024#";    // MM/dd/yyyy format also works

// Type conversion
view.RowFilter = "CONVERT(Age, 'System.String') LIKE '2%'";
```

### Common Filter Patterns

```csharp
// Numeric range
view.RowFilter = "Age BETWEEN 18 AND 65";
// Note: BETWEEN is NOT supported — use AND instead:
view.RowFilter = "Age >= 18 AND Age <= 65";

// Multiple string matches
view.RowFilter = "Name IN ('Long', 'Pham', 'Alice')";

// Parent-child relationship filter (when table is in a DataSet with relations)
view.RowFilter = "Parent(FK_Orders_Users).Name = 'Long'";

// Aggregate over child rows
view.RowFilter = "SUM(Child(FK_OrderItems).Quantity) > 10";

// Escape single quotes in string values
view.RowFilter = "Name = 'O''Brien'";  // double the single quote
```

```ad-warning
title: BETWEEN is Not Supported
Unlike SQL, the `DataView.RowFilter` expression syntax ==does NOT support the `BETWEEN` keyword==. Use `column >= low AND column <= high` instead. This catches many people who assume the expression syntax matches SQL exactly — it does not.
```

```ad-warning
title: RowFilter Is Not SQL
The expression syntax looks like SQL but has significant differences:
- No `BETWEEN`, no `EXISTS`, no subqueries
- String comparison is case-insensitive by default
- Date literals use `#` delimiters, not quotes
- Wildcard is `%` or `*` (not just `%`)
- `IS NULL` works, but `= NULL` does not (same as SQL, actually)
- Column names with spaces or special characters need square brackets: `[First Name]`
```

```ad-note
title: Section Summary
- `RowFilter` uses a SQL-like expression syntax with comparison, logical, string, and NULL operators
- Supports `LIKE`, `IN`, `IS NULL`, `IIF`, `LEN`, `SUBSTRING`, `TRIM`, and other functions
- Does NOT support `BETWEEN`, subqueries, or `EXISTS`
- Date literals use `#` delimiters; string values use single quotes; escape with doubled quotes
- Column names with spaces need `[square brackets]`
```

---

## Filtering by Row State

The `RowStateFilter` property filters rows by their `DataRowState`, which is useful for viewing only added, modified, or deleted rows:

```csharp
var view = new DataView(usersTable);

// Show only added rows (not yet in the database)
view.RowStateFilter = DataViewRowState.Added;

// Show only modified rows
view.RowStateFilter = DataViewRowState.ModifiedCurrent;  // show current values
view.RowStateFilter = DataViewRowState.ModifiedOriginal;  // show original values

// Show only deleted rows (their Original values)
view.RowStateFilter = DataViewRowState.Deleted;

// Show only unchanged rows
view.RowStateFilter = DataViewRowState.Unchanged;

// Show all current rows (Added + Modified + Unchanged) — this is the default
view.RowStateFilter = DataViewRowState.CurrentRows;

// Show all original rows (Modified original + Unchanged + Deleted)
view.RowStateFilter = DataViewRowState.OriginalRows;

// Combine flags (it's a flags enum)
view.RowStateFilter = DataViewRowState.Added | DataViewRowState.ModifiedCurrent;
```

| DataViewRowState | Shows | Row Version Used |
|---|---|---|
| `CurrentRows` (default) | `Added` + `ModifiedCurrent` + `Unchanged` | `Current` |
| `Added` | Only `Added` rows | `Current` |
| `ModifiedCurrent` | Only `Modified` rows | `Current` (new values) |
| `ModifiedOriginal` | Only `Modified` rows | `Original` (old values) |
| `Deleted` | Only `Deleted` rows | `Original` |
| `Unchanged` | Only `Unchanged` rows | `Current` |
| `OriginalRows` | `ModifiedOriginal` + `Unchanged` + `Deleted` | `Original` |
| `None` | No rows | N/A |

```ad-info
title: Practical Use Case
`RowStateFilter` is particularly useful for displaying pending changes to the user. For example, in a WinForms application:
- Bind one DataGridView to a view with `RowStateFilter = DataViewRowState.CurrentRows` (what the data looks like now)
- Bind another to `RowStateFilter = DataViewRowState.Deleted` (rows pending deletion)
- Use `ModifiedOriginal` vs `ModifiedCurrent` to show before/after comparisons
```

```ad-note
title: RowFilter + RowStateFilter Work Together
Both filters are applied simultaneously. Setting `RowFilter = "Age > 25"` and `RowStateFilter = DataViewRowState.Added` shows only newly added rows where Age > 25. Both conditions must be satisfied for a row to appear in the view.
```

```ad-note
title: Section Summary
- `RowStateFilter` filters by `DataRowState` — useful for viewing added, modified, or deleted rows
- Default is `CurrentRows` (shows `Added` + `Modified` + `Unchanged`)
- `ModifiedCurrent` shows new values; `ModifiedOriginal` shows values before modification
- `RowFilter` and `RowStateFilter` are combined — both must match for a row to appear
```

---

## Iterating and Accessing Rows

A `DataView` exposes its rows as `DataRowView` objects, not `DataRow`:

```csharp
var view = new DataView(usersTable);
view.Sort = "Name ASC";
view.RowFilter = "Age > 18";

// Iterate with foreach — yields DataRowView
foreach (DataRowView rowView in view)
{
    Console.WriteLine($"{rowView["Name"]}: {rowView["Age"]}");
}

// Access by index
DataRowView first = view[0];
Console.WriteLine(first["Name"]);

// Get the count
Console.WriteLine($"Visible rows: {view.Count}");

// Access the underlying DataRow from a DataRowView
DataRow underlyingRow = first.Row;
Console.WriteLine(underlyingRow.RowState);

// Access a specific row version
Console.WriteLine(first["Name"]);  // returns the version dictated by RowStateFilter
```

```ad-info
title: DataRowView vs DataRow
`DataRowView` is a **wrapper** around `DataRow` that respects the `DataView`'s current context (sort position, filter, row version based on `RowStateFilter`). When you access `rowView["Column"]`, it returns the appropriate row version. The underlying `DataRow` is always accessible via `rowView.Row`.
```

```ad-note
title: Section Summary
- `DataView` yields `DataRowView` objects, not `DataRow` — access via foreach or index
- `DataRowView` respects the view's sort, filter, and row state context
- Use `rowView.Row` to access the underlying `DataRow` directly
- `view.Count` gives the number of visible rows after filtering
```

---

## Finding Rows

`DataView` provides `Find()` and `FindRows()` for efficient lookups based on the `Sort` columns:

```csharp
var view = new DataView(usersTable);

// Sort MUST be set before using Find/FindRows
view.Sort = "Id";

// Find() — returns the index of the first matching row, or -1 if not found
int index = view.Find(42);
if (index != -1)
{
    DataRowView found = view[index];
    Console.WriteLine(found["Name"]);
}

// FindRows() — returns all matching DataRowView objects (for non-unique sort columns)
view.Sort = "Age";
DataRowView[] matches = view.FindRows(28);
foreach (DataRowView match in matches)
{
    Console.WriteLine($"{match["Name"]}, age {match["Age"]}");
}

// Composite sort key — pass an object array
view.Sort = "LastName, FirstName";
int idx = view.Find(new object[] { "Pham", "Long" });
DataRowView[] rows = view.FindRows(new object[] { "Pham", "Long" });
```

```ad-warning
title: Sort Must Be Set Before Find/FindRows
Calling `Find()` or `FindRows()` without setting `Sort` first throws an `ArgumentException`. The sort columns define what you're searching by — they ==are the lookup key==.
```

**Performance of Find:**

| Method | Algorithm | Performance |
|---|---|---|
| `Find()` | Binary search on the sorted index | O(log n) |
| `FindRows()` | Binary search + scan for duplicates | O(log n + k) where k = matches |
| `DataTable.Select()` | Full table scan | O(n) |
| `DataTable.Rows.Find()` | Hash table lookup on primary key | O(1) |

```ad-info
title: When to Use Which Lookup
- Need to find by **primary key**: Use `table.Rows.Find()` — O(1) hash lookup
- Need to find by **non-key column(s)**: Use `DataView.Find()` / `FindRows()` after setting `Sort` — O(log n) binary search
- Need **complex expression filter**: Use `table.Select("expression")` or `view.RowFilter` — O(n) scan, but more flexible
```

```ad-note
title: Section Summary
- `Find()` returns the index of the first match (or -1); `FindRows()` returns all matching `DataRowView` objects
- Both use binary search on the `Sort` columns — O(log n), much faster than `Select()`
- `Sort` must be set before calling `Find()` / `FindRows()` — the sort columns are the search key
- For primary key lookups, prefer `table.Rows.Find()` (O(1)); for non-key lookups, use `DataView.Find()`
```

---

## Editing Through DataRowView

`DataRowView` supports editing that flows through to the underlying `DataTable`:

```csharp
var view = new DataView(usersTable);
DataRowView rowView = view[0];

// Direct column modification
rowView["Name"] = "Updated Name";
// This modifies the underlying DataRow — row.RowState becomes Modified

// Transactional edit (like DataRow.BeginEdit/EndEdit)
rowView.BeginEdit();
rowView["Name"] = "Tentative";
rowView["Age"] = 99;
rowView.EndEdit();     // commits — DataRow.RowState becomes Modified
// OR
rowView.CancelEdit();  // reverts — no change to DataRow
```

The `AllowEdit`, `AllowNew`, and `AllowDelete` properties control what operations are permitted through the view:

```csharp
var view = new DataView(usersTable)
{
    AllowEdit = true,     // default: true — allow modifying existing rows
    AllowNew = true,      // default: true — allow adding new rows via AddNew()
    AllowDelete = true    // default: true — allow deleting rows via Delete()
};

// Read-only view
var readOnlyView = new DataView(usersTable)
{
    AllowEdit = false,
    AllowNew = false,
    AllowDelete = false
};
```

```ad-note
title: Section Summary
- Modifications through `DataRowView` flow directly to the underlying `DataRow`
- `BeginEdit()` / `EndEdit()` / `CancelEdit()` provide transactional editing
- `AllowEdit`, `AllowNew`, `AllowDelete` control what operations are permitted through the view
```

---

## Adding and Deleting Rows Through DataView

```csharp
var view = new DataView(usersTable);

// Add a new row through the view
DataRowView newRow = view.AddNew();
newRow["Id"] = 10;
newRow["Name"] = "New User";
newRow["Age"] = 30;
newRow.EndEdit(); // commits the new row to the underlying DataTable
// The underlying DataRow has RowState == Added

// Delete a row through the view
view[0].Delete(); // marks the underlying DataRow as Deleted
// Equivalent to: view[0].Row.Delete();
```

```ad-warning
title: AddNew Without EndEdit
If you call `view.AddNew()` without calling `EndEdit()`, the row remains in a `Detached` state. Calling `AddNew()` again before `EndEdit()` on the previous row automatically calls `EndEdit()` on the previous row (implicit commit). To discard the new row, call `CancelEdit()` before any other operation.
```

```ad-note
title: Section Summary
- `AddNew()` creates a new row through the view — call `EndEdit()` to commit or `CancelEdit()` to discard
- `Delete()` on a `DataRowView` marks the underlying `DataRow` as `Deleted`
- All additions and deletions through the view affect the underlying `DataTable`
```

---

## Multiple Views on One Table

You can create multiple independent views on the same `DataTable`, each with its own sort and filter:

```csharp
var allUsers = new DataView(usersTable)
{
    Sort = "Name ASC"
};

var adultsOnly = new DataView(usersTable)
{
    RowFilter = "Age >= 18",
    Sort = "Age DESC"
};

var pendingChanges = new DataView(usersTable)
{
    RowStateFilter = DataViewRowState.Added | DataViewRowState.ModifiedCurrent,
    Sort = "Id"
};

Console.WriteLine($"All users: {allUsers.Count}");
Console.WriteLine($"Adults: {adultsOnly.Count}");
Console.WriteLine($"Pending: {pendingChanges.Count}");

// All three views reference the same data — modifying a row through one view
// is immediately visible in the others (if it passes their filters)
```

```ad-note
title: Section Summary
- Multiple `DataView` objects can reference the same `DataTable` independently
- Each view has its own `Sort`, `RowFilter`, and `RowStateFilter`
- Changes through any view are reflected in all views (subject to each view's filters)
```

---

## LINQ to DataSet and AsDataView

LINQ to DataSet provides LINQ query capabilities over `DataTable`, and `AsDataView()` converts a LINQ query result back into a `DataView`:

```csharp
using System.Data;  // for DataTableExtensions and DataRowExtensions

// Convert DataTable to an IEnumerable<DataRow> for LINQ
var query = usersTable.AsEnumerable()
    .Where(row => row.Field<int>("Age") > 18)
    .OrderBy(row => row.Field<string>("Name"));

// Execute LINQ and iterate
foreach (DataRow row in query)
{
    Console.WriteLine($"{row.Field<string>("Name")}: {row.Field<int>("Age")}");
}

// Convert LINQ result back to a DataView (for data binding)
DataView linqView = query.AsDataView();
// linqView can be bound to UI controls just like a regular DataView

// CopyToDataTable() — creates a NEW DataTable from the LINQ result
DataTable filteredTable = query.CopyToDataTable();
```

```ad-info
title: AsEnumerable vs AsDataView
- `table.AsEnumerable()` returns an `EnumerableRowCollection<DataRow>` — LINQ to Objects on the rows. Good for complex transformations (grouping, joining, projections) that the `RowFilter` expression syntax cannot handle.
- `query.AsDataView()` converts the LINQ result into a `DataView` — needed for data binding to UI controls.
- `query.CopyToDataTable()` creates a brand-new `DataTable` — a full copy, not a reference. Use when you need an independent table.
```

```ad-warning
title: AsDataView Requires a Source DataTable
`AsDataView()` only works on `EnumerableRowCollection<DataRow>` — the rows must originate from a `DataTable`. It does not work on arbitrary `IEnumerable<T>` collections. The returned `DataView` references the original `DataTable`, with a filter matching the LINQ query result.
```

```ad-note
title: Section Summary
- `table.AsEnumerable()` enables LINQ queries on `DataTable` rows
- `query.AsDataView()` converts LINQ results back to a `DataView` for data binding
- `query.CopyToDataTable()` creates an independent `DataTable` copy from LINQ results
- LINQ provides more expressive querying than `RowFilter` (grouping, joining, projections)
```

---

## Data Binding

`DataView` is the ==primary mechanism for binding `DataTable` data to UI controls== in WinForms and WPF:

**WinForms:**

```csharp
// Bind to DataGridView
var view = new DataView(usersTable)
{
    Sort = "Name ASC",
    RowFilter = "Age >= 18"
};
dataGridView1.DataSource = view;

// Or bind via BindingSource (recommended for WinForms)
var bindingSource = new BindingSource();
bindingSource.DataSource = view;
dataGridView1.DataSource = bindingSource;

// The grid automatically updates when Sort/RowFilter changes
view.RowFilter = "Age >= 21"; // grid refreshes to show only age 21+
```

**WPF:**

```xml
<!-- In XAML -->
<DataGrid ItemsSource="{Binding UsersView}" AutoGenerateColumns="True" />
```

```csharp
// In the ViewModel or code-behind
public DataView UsersView => usersTable.DefaultView;

// Set filter/sort
usersTable.DefaultView.Sort = "Name ASC";
usersTable.DefaultView.RowFilter = "IsActive = true";
```

```ad-info
title: Why DataView for Data Binding?
UI controls need a list that supports:
- **Sorting** — user clicks a column header to sort
- **Filtering** — user types in a search box to filter
- **Change notification** — when data changes, the UI updates
- **Editing** — user edits a cell, the change propagates to the data

`DataView` and `DataRowView` implement the `IBindingListView`, `IBindingList`, and `IEditableObject` interfaces that WinForms and WPF controls expect. Binding directly to `DataTable.Rows` does not support sorting or filtering in the UI.
```

```ad-note
title: Section Summary
- `DataView` is the standard binding source for `DataTable` in both WinForms and WPF
- Bind via `control.DataSource = view` (WinForms) or `ItemsSource="{Binding View}"` (WPF)
- `DataView` implements `IBindingListView` — supports sort, filter, and change notification for the UI
- Use `BindingSource` in WinForms for additional navigation and currency management
```

---

## Performance Considerations

| Concern | Details |
|---|---|
| **Index maintenance** | `DataView` maintains an internal sorted index. Setting `Sort` or `RowFilter` rebuilds it — O(n log n). |
| **Multiple views** | Each `DataView` on the same `DataTable` maintains its own index. 10 views = 10 indexes = 10x memory and rebuild cost. |
| **Find performance** | `Find()` is O(log n) binary search — much faster than `table.Select()` for repeated lookups on the same columns. |
| **RowFilter vs LINQ** | `RowFilter` is parsed and evaluated by the `DataView` engine. LINQ to DataSet uses compiled delegates. For complex filtering called repeatedly, LINQ may be faster; for simple filters, `RowFilter` is comparable. |
| **ListChanged event** | `DataView` fires `ListChanged` events on data changes. With large tables and frequent updates, this can cause UI lag. Consider `table.BeginLoadData()` / `table.EndLoadData()` to batch changes. |

```ad-note
title: Section Summary
- `DataView` maintains an internal index — sorting/filtering rebuilds it (O(n log n))
- Multiple views on the same table each have their own index — be mindful of memory
- `Find()` is O(log n), significantly faster than `table.Select()` for repeated lookups
- For large tables with frequent updates, batch changes with `BeginLoadData()` / `EndLoadData()`
```

---

## Comprehensive Summary

```ad-important
title: Key Takeaways
**DataView** is a lightweight, sortable, filterable view over a [[DataSet and DataTable|DataTable]]. It does not copy data — it maintains an internal index of pointers into the underlying table.

**Sorting** is set via the `Sort` property (`"Name ASC, Age DESC"`). **Filtering** uses `RowFilter` (SQL-like expression syntax) and `RowStateFilter` (filter by `DataRowState`). Both filters are combined — rows must satisfy both.

**Row access** is through `DataRowView` objects, which wrap `DataRow` and respect the view's context. Modifications through `DataRowView` flow directly to the underlying `DataRow`.

**Finding** uses `Find()` (returns index) and `FindRows()` (returns `DataRowView[]`). Both use binary search on the `Sort` columns — ==O(log n)==, much faster than `table.Select()` (O(n)).

**Data binding** is the primary use case: `DataView` implements `IBindingListView`, making it the standard binding source for `DataTable` in WinForms (`DataSource`) and WPF (`ItemsSource`). `DataTable.DefaultView` is a convenient shared instance, but create new views when you need independent sort/filter settings.

**LINQ integration**: `table.AsEnumerable()` enables LINQ queries; `query.AsDataView()` converts results back to a bindable `DataView`.

**Key rules:**
- `DefaultView` is a singleton — modifying it affects all consumers
- `Sort` must be set before calling `Find()` / `FindRows()`
- `RowFilter` does NOT support `BETWEEN` — use `column >= low AND column <= high`
- Changes through any view are immediately visible in all views on the same table
```

---

## Related Topics

- [[DataSet and DataTable]] — the underlying data structures that `DataView` references
- [[DataAdapter]] — fills `DataTable` from the database; `DataView` presents the data
- [[Connected vs Disconnected Layer]] — when to use the disconnected layer (where `DataView` lives)
- [[ADO.NET Overview]] — the overall ADO.NET architecture
- [[Entity Framework Core]] — modern alternative where LINQ replaces `DataView` for filtering/sorting
