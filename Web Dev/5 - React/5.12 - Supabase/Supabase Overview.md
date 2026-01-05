---
tags: 
 - supabase
 - overview
---

Supabase is an open-source **Backend-as-a-Service (BaaS)** platform designed to help developers build secure, scalable, and real-time applications quickly.

It is widely known as the **"Open Source Firebase Alternative"**, but with a critical difference: **it is built on top of PostgreSQL**, a powerful relational database, rather than a NoSQL document store. This gives you the speed of a BaaS with the complex querying power of a standard SQL database.

![[Pasted image 20260104092129.png|center|700]]

---

### 1. The Core Philosophy: "It’s just Postgres"

Unlike other platforms that abstract the database away, Supabase gives you full access to a standard PostgreSQL database. Every feature in Supabase (Auth, Realtime, Storage) is built to utilize Postgres' native capabilities.

- **No Vendor Lock-in:** Since it is standard Postgres, you can dump your data and move to AWS RDS, Heroku, or any other provider at any time.
    
- **Ecosystem Compatible:** You can use standard Postgres tools (Prisma, Drizzle, pgAdmin) directly with Supabase.
    

---

### 2. Key Features Breakdown

#### A. Database & Auto-generated APIs

When you create a table in your Supabase dashboard, the platform instantly exposes it via API. You don't need to write a backend server (like Express or NestJS) to create CRUD endpoints.

- **REST API (PostgREST):** Supabase automatically introspects your database schema and provides a RESTful API that reflects your tables, columns, and relationships.
    
- **GraphQL:** Through an extension called `pg_graphql`, you also get a full GraphQL endpoint instantly.
    

#### B. Authentication & Authorization (RLS)

This is Supabase's strongest security feature.

- **Auth:** Handles sign-ups via Email/Password, Magic Links, OTPs, and Social Logins (Google, GitHub, Apple, etc.).
    
- **Row Level Security (RLS):** instead of writing API middleware to check if `user_id === post_owner_id`, you write **SQL policies** directly on the database.
    
    - _Example Policy:_ `CREATE POLICY "User can update own post" ON posts FOR UPDATE USING (auth.uid() = user_id);`
        
    - The database _itself_ rejects unauthorized access, even if the API request tries to bypass it.
        

#### C. Realtime Subscriptions

Supabase allows you to listen to changes in your database via WebSockets.

- **Postgres Changes:** Subscribe to `INSERT`, `UPDATE`, or `DELETE` events on specific tables. (e.g., "Alert me when a new row is added to the `messages` table").
    
- **Broadcast:** Send ephemeral messages between users (like mouse cursors or "User is typing..." indicators) without storing them in the database.
    
- **Presence:** Track who is currently online (e.g., "3 users viewing this page").
    

#### D. Storage

A file storage system for images, videos, and docs, similar to AWS S3.

- It organizes files into "Buckets."
    
- It integrates with the database Auth/RLS, so you can write policies like "Only paying users can download from the `premium-content` bucket."
    
- Includes a built-in image optimizer (resize/compress on the fly).
    

#### E. Edge Functions

For complex business logic that cannot be handled by simple database operations (e.g., processing a Stripe payment, sanitizing data, or calling an OpenAI API), Supabase provides serverless **Edge Functions**.

- Built on **Deno** (allows TypeScript natively).
    
- Runs globally at the "edge" (closer to the user) for lower latency.
    

---

### 3. Supabase vs. Firebase

The choice often comes down to Relational (SQL) vs. Non-Relational (NoSQL).

| **Feature**    | **Supabase (Postgres)**                                                                        | **Firebase (NoSQL)**                                                                    |
| -------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Data Model** | **Relational (Tables)**. Great for structured data, complex relationships, and strict schemas. | **Document (JSON)**. Great for unstructured data and rapid prototyping without schemas. |
| **Querying**   | **Powerful SQL**. Supports complex `JOINS`, aggregations, and filtering.                       | **Limited**. Complex queries often require data duplication or client-side filtering.   |
| **Realtime**   | Opt-in (you enable it per table).                                                              | On by default (Firestore is realtime-native).                                           |
| **Pricing**    | Predictable (based on storage/compute).                                                        | "Pay as you go" (can be expensive if you have high read/write volume).                  |
| **Lock-in**    | Low. It's just Postgres; you can export and leave.                                             | High. Migration out of Firestore is difficult due to proprietary structure.             |

---

### 4. When to Use Supabase

**✅ Use Supabase if:**

- You need **complex relationships** in your data (e.g., a SaaS app with Organizations, Teams, Users, and Projects).
    
- You prefer **SQL** and structured schemas over NoSQL.
    
- You want to avoid vendor lock-in and prefer open-source standards.
    
- You are building an AI wrapper (Supabase has excellent **Vector/Embedding** support via the `pgvector` extension).
    

**❌ Consider alternatives if:**

- You need "Offline Mode" capabilities out of the box (Firebase has better native offline support for mobile).
    
- You are building a legacy enterprise app that requires a specific non-Postgres database (like Oracle or MS SQL).
    

### 5. Next Steps

To get started, the standard workflow is:

1. **Create a Project** on Supabase.com.
    
2. **Define your Schema** (Create tables in the Table Editor).
    
3. **Enable RLS** and write a policy (e.g., "Enable read access for everyone").
    
4. **Connect your frontend** using the Supabase JavaScript client:
    

```js
import { createClient } from '@supabase/supabase-js'

const supabase = createClient('https://xyz.supabase.co', 'public-anon-key')

// Fetch data (Auto-generated API)
const { data, error } = await supabase
  .from('todos')
  .select('*')
```
