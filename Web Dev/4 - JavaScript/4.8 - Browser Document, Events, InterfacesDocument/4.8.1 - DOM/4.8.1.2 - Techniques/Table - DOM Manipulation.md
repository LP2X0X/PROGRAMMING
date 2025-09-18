---
tags: 
 - js
 - DOM
 - table
 - fundamental
---

## Table-Specific DOM Properties & Collections

Some DOM elements—especially `<table>` and its related elements—provide extra navigation helpers beyond the generic child/sibling/parent ones. These let you more directly access parts of tables in a semantic way.

---

### Table-related properties

|Element|Property|What it provides / returns|
|---|---|---|
|`<table>`|`table.rows`|A collection of all `<tr>` elements in the entire table (including in `<thead>`, `<tbody>`, `<tfoot>`). |
|`<table>`|`table.caption`|The `<caption>` element of the table (if any) |
|`<table>`|`table.tHead`|The `<thead>` element (if present) |
|`<table>`|`table.tFoot`|The `<tfoot>` element (if present) |
|`<table>`|`table.tBodies`|A collection of all `<tbody>` elements. **Note: there is at least one `<tbody>` in the DOM even if not explicitly in HTML.** |

---

### Properties of sub-elements

|Sub-element|Property|What it provides / returns|
|---|---|---|
|`<tbody>`, `<thead>`, `<tfoot>`|`.rows`|Collection of `<tr>` rows within that section. |
|`<tr>`|`.cells`|Collection of its child cells: both `<td>` and `<th>` elements. |
|`<tr>`|`.sectionRowIndex`|Index of that `<tr>` within its parent section (e.g. which `<tr>` it is inside `<tbody>`, `<thead>`, etc.). Zero-based. |
|`<tr>`|`.rowIndex`|Index of the `<tr>` among _all_ rows in the table, counting all sections. |
|`<td>` / `<th>`|`.cellIndex`|The column position (index) of the cell within its row. |

---

### Example

```html
<table id="table">
  <tr>
    <td>one</td><td>two</td>
  </tr>
  <tr>
    <td>three</td><td>four</td>
  </tr>
</table>
<script>
  let table = document.getElementById("table");
  // Access the cell containing “two” (first row, second cell)
  let td = table.rows[0].cells[1];
  td.style.backgroundColor = "red"; // highlight it
</script>
```