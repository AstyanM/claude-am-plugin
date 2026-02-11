---
name: dbml
description: "Use this skill whenever the user wants to design, create, edit, or visualize a database schema using DBML (Database Markup Language). Triggers include: any mention of 'dbml', 'database schema', 'ERD', 'entity relationship diagram', 'table structure', 'data model', 'database design', 'db schema', or requests to model tables, relationships, foreign keys, or database architectures. Also use when converting SQL DDL to DBML, generating schemas from descriptions, or when the user has the DBML Live Preview VS Code extension installed. Do NOT use for writing raw SQL queries, Prisma schemas, actual database migrations, or ORM model code."
---

# DBML Database Schema Design

## Overview

DBML (Database Markup Language) is a human-readable DSL for defining database structures. Files use the `.dbml` extension and can be visualized as ERD diagrams in VS Code via the **DBML Live Preview** extension (`nicolas-liger.dbml-viewer`).

DBML is database-agnostic: focus on structure, not SQL dialect specifics.

## Quick Reference

| Task                       | Approach                                                          |
| -------------------------- | ----------------------------------------------------------------- |
| New schema from scratch    | Write `.dbml` file following structure below                      |
| Convert SQL → DBML         | Translate CREATE TABLE / ALTER TABLE into DBML syntax             |
| Convert description → DBML | Extract entities, attributes, relationships from natural language |
| Edit existing schema       | Parse `.dbml` file, modify tables/refs, write back                |

---

## File Structure

Every `.dbml` file follows this general order:

```dbml
// 1. Project definition (optional but recommended)
Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Description of the project'
}

// 2. Enum definitions
Enum status_type {
  active
  inactive
  archived
}

// 3. Table definitions
Table users {
  id integer [pk, increment]
  name varchar [not null]
  status status_type [default: 'active']
  created_at timestamp [default: `now()`]
}

// 4. Standalone relationships (alternative to inline refs)
Ref: posts.user_id > users.id

// 5. Table groups (optional)
TableGroup core {
  users
  posts
}
```

---

## Project Definition

Always start with a Project block to give context:

```dbml
Project ecommerce {
  database_type: 'PostgreSQL'   // or 'MySQL', 'SQLite', 'SQL Server'
  Note: '''
    # E-Commerce Database
    Designed for a multi-tenant SaaS platform.
    Last updated: 2025-01.
  '''
}
```

---

## Table Definition

```dbml
Table schema_name.table_name as Alias [headercolor: #3498DB] {
  column_name column_type [settings]
}
```

- `schema_name` is optional (defaults to `public`)
- `as Alias` is optional — useful for shortening long names in Refs
- `headercolor` changes the table header color in the ERD visualization

### Table Notes

```dbml
Table users {
  id integer [pk]

  Note: 'Stores all registered users'
}
```

---

## Column Definition

```dbml
Table example {
  column_name column_type [setting1, setting2, ...]
}
```

### Column Types

Use any standard SQL type as a single word. If the type contains spaces, wrap it in double quotes.

| Category  | Types                                                          |
| --------- | -------------------------------------------------------------- |
| Integer   | `integer`, `int`, `smallint`, `bigint`, `serial`, `bigserial`  |
| Decimal   | `decimal(10,2)`, `numeric`, `float`, `double`, `real`, `money` |
| String    | `varchar`, `varchar(255)`, `char(10)`, `text`, `citext`        |
| Boolean   | `boolean`, `bool`                                              |
| Date/Time | `date`, `time`, `timestamp`, `timestamptz`, `interval`         |
| Binary    | `bytea`, `blob`, `binary`                                      |
| JSON      | `json`, `jsonb`                                                |
| UUID      | `uuid`                                                         |
| Array     | `"integer[]"`, `"varchar[]"`                                   |
| Custom    | `"bigint unsigned"`, any enum name                             |

### Column Settings

| Setting               | Description             | Example                              |
| --------------------- | ----------------------- | ------------------------------------ |
| `pk` or `primary key` | Primary key             | `id integer [pk]`                    |
| `increment`           | Auto-increment          | `id integer [pk, increment]`         |
| `not null`            | NOT NULL constraint     | `name varchar [not null]`            |
| `null`                | Explicitly nullable     | `bio text [null]`                    |
| `unique`              | Unique constraint       | `email varchar [unique, not null]`   |
| `default: value`      | Default value           | `status varchar [default: 'active']` |
| `note: 'text'`        | Column note/description | `role varchar [note: 'User role']`   |
| `ref: > table.col`    | Inline relationship     | `user_id int [ref: > users.id]`      |

### Default Values

```dbml
Table example {
  // Number
  rating integer [default: 10]

  // String (single quotes)
  source varchar [default: 'direct']

  // Expression (backticks)
  created_at timestamp [default: `now()`]
  valid_until date [default: `now() + interval '30 days'`]

  // Boolean / null
  is_active boolean [default: true]
  deleted_at timestamp [default: null]
}
```

---

## Relationships

### Relationship Types

| Symbol | Meaning      | Example                       |
| ------ | ------------ | ----------------------------- |
| `>`    | Many-to-one  | `posts.user_id > users.id`    |
| `<`    | One-to-many  | `users.id < posts.user_id`    |
| `-`    | One-to-one   | `users.id - profiles.user_id` |
| `<>`   | Many-to-many | `students.id <> courses.id`   |

### Three Ways to Define Relationships

**1. Inline (in column settings):**

```dbml
Table posts {
  id integer [pk]
  user_id integer [ref: > users.id]
}
```

**2. Standalone Ref:**

```dbml
Ref: posts.user_id > users.id
```

**3. Standalone named Ref with block:**

```dbml
Ref user_posts {
  posts.user_id > users.id
}
```

### Relationship Settings

```dbml
Ref: posts.user_id > users.id [delete: cascade, update: no action]
```

Available actions: `cascade`, `restrict`, `set null`, `set default`, `no action`

### Composite Foreign Keys

```dbml
Ref: order_items.(order_id, product_id) > products.(order_id, id)
```

### Many-to-Many

DBML supports direct many-to-many with `<>`, but for explicit junction tables:

```dbml
Table students_courses {
  student_id integer [ref: > students.id]
  course_id integer [ref: > courses.id]

  indexes {
    (student_id, course_id) [pk]
  }
}
```

---

## Indexes

```dbml
Table bookings {
  id integer [pk]
  country varchar
  booking_date date
  created_at timestamp

  indexes {
    created_at [name: 'idx_created_at', note: 'Date index']
    booking_date [type: hash]
    (country, booking_date) [unique]
    (id, country) [pk]             // composite primary key
    (`lower(country)`)             // expression index
  }
}
```

### Index Settings

| Setting                       | Description                     |
| ----------------------------- | ------------------------------- |
| `name: 'index_name'`          | Custom index name               |
| `unique`                      | Unique index                    |
| `pk`                          | Primary key (for composite PKs) |
| `type: btree` or `type: hash` | Index type                      |
| `note: 'text'`                | Index description               |

---

## Enums

```dbml
Enum order_status {
  pending     [note: 'Order just placed']
  processing  [note: 'Payment confirmed']
  shipped
  delivered
  cancelled
}

Table orders {
  id integer [pk]
  status order_status [not null, default: 'pending']
}
```

For enums in a specific schema:

```dbml
Enum v2.order_status {
  pending
  shipped
}
```

If enum values contain spaces or special characters, use double quotes:

```dbml
Enum color {
  "bright red"
  "dark blue"
}
```

---

## Table Groups

Groups are purely visual — they cluster tables together in the ERD:

```dbml
TableGroup core_tables [color: #2196F3] {
  users
  profiles
  roles
}

TableGroup content_tables {
  posts
  comments
  tags
}
```

---

## TablePartial (Reusable Columns)

Define common columns once, inject into multiple tables:

```dbml
TablePartial timestamps {
  created_at timestamp [not null, default: `now()`]
  updated_at timestamp [not null, default: `now()`]
}

TablePartial soft_delete {
  deleted_at timestamp [null]
  is_deleted boolean [default: false]
}

Table users {
  id integer [pk, increment]
  name varchar [not null]

  ~timestamps      // injects created_at and updated_at
  ~soft_delete     // injects deleted_at and is_deleted
}

Table posts {
  id integer [pk, increment]
  title varchar [not null]

  ~timestamps
}
```

---

## Notes and Documentation

### Multi-line Notes (triple single quotes)

```dbml
Table users {
  id integer [pk]

  Note: '''
    # Users Table
    Contains all registered users.
    - Soft-deleted users have `deleted_at` set
    - Email must be verified before `is_active = true`
  '''
}
```

### Column Notes

```dbml
Table orders {
  total decimal(10,2) [note: 'Total in USD, before tax']
}
```

### Sticky Notes

Free-floating notes not attached to any table:

```dbml
Note single_line_note {
  'This is a single line note'
}

Note detailed_note {
  '''
    # Architecture Decision
    We use soft deletes across all tables.
  '''
}
```

---

## Comments

```dbml
// Single-line comment

Table users {  // inline comment
  id integer [pk]
}
```

**Note:** DBML does NOT support multi-line `/* */` comments.

---

## Common Patterns

### SaaS Multi-Tenant

```dbml
Table tenants {
  id uuid [pk, default: `gen_random_uuid()`]
  name varchar [not null]
  slug varchar [unique, not null]
  plan varchar [default: 'free']
  created_at timestamp [default: `now()`]
}

Table users {
  id uuid [pk, default: `gen_random_uuid()`]
  tenant_id uuid [not null, ref: > tenants.id]
  email varchar [unique, not null]
  role varchar [not null, default: 'member']
  created_at timestamp [default: `now()`]

  indexes {
    (tenant_id, email) [unique]
  }
}
```

### Auth / RBAC

```dbml
Table users {
  id integer [pk, increment]
  email varchar [unique, not null]
  password_hash varchar [not null]
}

Table roles {
  id integer [pk, increment]
  name varchar [unique, not null]
}

Table permissions {
  id integer [pk, increment]
  name varchar [unique, not null, note: 'e.g. users:read, posts:write']
}

Table user_roles {
  user_id integer [ref: > users.id]
  role_id integer [ref: > roles.id]

  indexes {
    (user_id, role_id) [pk]
  }
}

Table role_permissions {
  role_id integer [ref: > roles.id]
  permission_id integer [ref: > permissions.id]

  indexes {
    (role_id, permission_id) [pk]
  }
}
```

### E-Commerce

```dbml
Enum order_status {
  pending
  paid
  shipped
  delivered
  refunded
}

Table products {
  id integer [pk, increment]
  name varchar [not null]
  price decimal(10,2) [not null]
  stock integer [not null, default: 0]
}

Table orders {
  id integer [pk, increment]
  user_id integer [not null, ref: > users.id]
  status order_status [not null, default: 'pending']
  total decimal(10,2) [not null]
  created_at timestamp [default: `now()`]
}

Table order_items {
  id integer [pk, increment]
  order_id integer [not null, ref: > orders.id]
  product_id integer [not null, ref: > products.id]
  quantity integer [not null, default: 1]
  unit_price decimal(10,2) [not null]
}
```

---

## Best Practices

### Naming Conventions

- **Tables**: `snake_case`, plural (`users`, `order_items`)
- **Columns**: `snake_case`, singular (`user_id`, `created_at`)
- **Enums**: `snake_case` (`order_status`, `user_role`)
- **Foreign keys**: `referenced_table_singular_id` (`user_id`, `order_id`)
- **Junction tables**: `table1_table2` alphabetically (`courses_students`)

### Organization

- Define Enums before the tables that use them
- Group related tables together
- Use TableGroups for visual organization in large schemas
- Use TablePartials for common columns (`timestamps`, `soft_delete`, `audit_fields`)
- Add Notes on every table to explain its purpose
- Comment complex relationships

### Schema Quality

- Every table should have a primary key
- Add `not null` on columns that should never be empty
- Use `default` values where appropriate
- Define indexes for foreign keys and frequently queried columns
- Use enums instead of raw strings for status/type columns
- Add `[note: ...]` on non-obvious columns

---

## Validation Checklist

Before finalizing a `.dbml` file, verify:

- [ ] Every table has a primary key (`[pk]` or composite via indexes)
- [ ] All foreign key columns reference valid table.column pairs
- [ ] Enum types referenced in columns are defined
- [ ] Column types are single words (use double quotes for types with spaces)
- [ ] Notes use single quotes `'...'` or triple single quotes `'''...'''`
- [ ] No `/* */` multi-line comments (use `//` only)
- [ ] TablePartials are defined before the tables that use `~partial_name`
- [ ] String defaults use single quotes: `default: 'value'`
- [ ] Expression defaults use backticks: `default: \`now()\``
- [ ] Relationship symbols are correct (`>` many-to-one, `<` one-to-many, `-` one-to-one, `<>` many-to-many)

---

## Output

- Write the file with `.dbml` extension (e.g., `schema.dbml`)
- Use UTF-8 encoding
- The file will be rendered as an ERD in VS Code via the DBML Live Preview extension
