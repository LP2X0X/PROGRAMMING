---
tags:
  - mermaid
  - er-diagram
  - diagram
  - database
---

# ER Diagrams

Mermaid's **Entity-Relationship (ER) diagram** syntax lets you model database schemas directly in Markdown. You define entities with typed attributes, then connect them with crow's foot notation to express cardinality and optionality — all rendered inline in Obsidian's reading view.

**See also:** [[Syntax Basics]], [[Styling and Themes]]

---

## 🔹 Quick Reference: Relationship Notation

Cardinality is expressed with **two markers** — one on each side of the relationship line. Read the diagram from left entity to right entity.

| Marker (Left) | Marker (Right) | Meaning          |
| -------------- | -------------- | ---------------- |
| `\|o`          | `o\|`          | Zero or one      |
| `\|\|`         | `\|\|`         | Exactly one      |
| `}o`           | `o{`           | Zero or more     |
| `}\|`          | `\|{`          | One or more      |

> **Reading order:** the marker closest to an entity describes *that* entity's participation.
> `CUSTOMER ||--o{ ORDER` reads: "each **Customer** has exactly one match, each **Order** has zero or more" — i.e., one customer to many orders.

### Relationship Line Style

| Line   | Style              | Meaning                                            |
| ------ | ------------------ | -------------------------------------------------- |
| `--`   | Solid              | **Identifying** — child cannot exist without parent |
| `..`   | Dotted / Dashed    | **Non-identifying** — child can exist independently |

### Full Syntax

```text
ENTITY1 <left-marker><line><right-marker> ENTITY2 : "label"
```

Examples:

```text
CUSTOMER ||--o{ ORDER : "places"
ORDER ||--|{ ORDER_ITEM : "contains"
PRODUCT }o..o{ TAG : "tagged with"
```

---

## 🔹 Defining Entities

### Simple Entity (No Attributes)

Just reference the entity name in a relationship — Mermaid creates the box automatically.

```text
CUSTOMER ||--o{ ORDER : "places"
```

### Entity with Attributes

Use a block with `type name` pairs inside curly braces:

```text
CUSTOMER {
    int id
    string name
    string email
    date created_at
}
```

Common **attribute types** (Mermaid treats these as labels — any string works):

| Type       | Typical Use              |
| ---------- | ------------------------ |
| `int`      | Integers, IDs            |
| `string`   | Text fields              |
| `float`    | Decimal numbers          |
| `date`     | Dates                    |
| `datetime` | Timestamps               |
| `boolean`  | True/false flags         |
| `enum`     | Enumerated values        |

### Keys and Constraints

Append `PK`, `FK`, or `UK` after the attribute name:

```text
CUSTOMER {
    int id PK
    string email UK
    string name
}

ORDER {
    int id PK
    int customer_id FK
    date order_date
}
```

### Comments on Attributes

Add a **quoted string** after the key marker (or after the name if no key) to annotate:

```text
CUSTOMER {
    int id PK "Auto-increment"
    string email UK "Must be unique"
    string name "Full legal name"
}
```

### Live Example

```mermaid
erDiagram
    CUSTOMER {
        int id PK "Auto-increment"
        string name
        string email UK
        date created_at
    }
    ORDER {
        int id PK
        int customer_id FK
        date order_date
        float total
    }
```

---

## 🔹 Relationships

Relationships connect two entities with cardinality on each side and a label describing the connection. The label should be a **verb or verb phrase** read left to right.

### One-to-One

Each entity on both sides has **exactly one** match.

```mermaid
erDiagram
    USER ||--|| PROFILE : "has"
```

### One-to-Many

One parent maps to **zero or more** children. This is the most common relationship in relational databases.

```mermaid
erDiagram
    DEPARTMENT ||--o{ EMPLOYEE : "employs"
```

### Many-to-Many

Both sides can have **multiple** matches. In a physical schema this is typically resolved with a junction/bridge table.

```mermaid
erDiagram
    STUDENT }o--o{ COURSE : "enrolls in"
```

### Identifying vs Non-Identifying

**Identifying** (solid `--`): the child's primary key includes the parent's key — the child cannot exist without the parent.

```mermaid
erDiagram
    ORDER ||--|{ ORDER_ITEM : "contains"
```

**Non-identifying** (dotted `..`): the child references the parent but has its own independent identity.

```mermaid
erDiagram
    CUSTOMER ||..o{ ORDER : "places"
```

### Zero-or-One Relationships

Use `o|` / `|o` for optional single-side participation:

```mermaid
erDiagram
    EMPLOYEE |o--|| PARKING_SPOT : "assigned"
```

This reads: an employee has **zero or one** parking spot; each parking spot belongs to **exactly one** employee.

---

## 🔹 Attribute Keys

| Key  | Meaning         | Rendered As      |
| ---- | --------------- | ---------------- |
| `PK` | Primary Key     | Marked with PK   |
| `FK` | Foreign Key     | Marked with FK   |
| `UK` | Unique Key      | Marked with UK   |

You can combine keys with comments for full documentation:

```mermaid
erDiagram
    PRODUCT {
        int id PK "Auto-generated"
        string sku UK "Stock Keeping Unit"
        string name
        float price
        int category_id FK "References CATEGORY"
    }
    CATEGORY {
        int id PK
        string name UK
        string description
    }
    CATEGORY ||--o{ PRODUCT : "contains"
```

---

## 🔹 Real-World Examples

### E-Commerce Schema

```mermaid
erDiagram
    CUSTOMER {
        int id PK
        string name
        string email UK
        string phone
        date registered_at
    }
    ORDER {
        int id PK
        int customer_id FK
        date order_date
        string status
        float total_amount
    }
    ORDER_ITEM {
        int id PK
        int order_id FK
        int product_id FK
        int quantity
        float unit_price
    }
    PRODUCT {
        int id PK
        string name
        string sku UK
        float price
        int category_id FK
    }
    CATEGORY {
        int id PK
        string name UK
        string description
    }

    CUSTOMER ||--o{ ORDER : "places"
    ORDER ||--|{ ORDER_ITEM : "contains"
    PRODUCT ||--o{ ORDER_ITEM : "appears in"
    CATEGORY ||--o{ PRODUCT : "groups"
```

### Blog / CMS Schema

This example demonstrates a **many-to-many** relationship between posts and tags, resolved through a junction table `POST_TAG`.

```mermaid
erDiagram
    USER {
        int id PK
        string username UK
        string email UK
        string password_hash
        date joined_at
    }
    POST {
        int id PK
        int author_id FK
        string title
        string body
        datetime published_at
        string status
    }
    COMMENT {
        int id PK
        int post_id FK
        int user_id FK
        string body
        datetime created_at
    }
    TAG {
        int id PK
        string name UK
        string slug UK
    }
    POST_TAG {
        int post_id FK
        int tag_id FK
    }

    USER ||--o{ POST : "writes"
    USER ||--o{ COMMENT : "authors"
    POST ||--o{ COMMENT : "receives"
    POST ||--|{ POST_TAG : "tagged via"
    TAG ||--|{ POST_TAG : "applied via"
```

### HR Schema

```mermaid
erDiagram
    DEPARTMENT {
        int id PK
        string name UK
        string location
    }
    EMPLOYEE {
        int id PK
        string first_name
        string last_name
        string email UK
        date hire_date
        float salary
        int department_id FK
        int manager_id FK "Self-ref to EMPLOYEE"
    }
    PROJECT {
        int id PK
        string name
        date start_date
        date end_date
        string status
    }
    ASSIGNMENT {
        int employee_id FK
        int project_id FK
        string role
        date assigned_date
    }

    DEPARTMENT ||--o{ EMPLOYEE : "employs"
    EMPLOYEE ||--o{ EMPLOYEE : "manages"
    EMPLOYEE ||--|{ ASSIGNMENT : "works on"
    PROJECT ||--|{ ASSIGNMENT : "staffed by"
```

---

**See also:** [[Syntax Basics]], [[Class Diagrams]], [[Styling and Themes]]
