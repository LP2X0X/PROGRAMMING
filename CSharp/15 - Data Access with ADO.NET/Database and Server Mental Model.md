---
tags:
 - csharp
 - database
 - ado-net
---

## The Big Picture

Everything in the database world has three layers: the **server** (the program), the **databases** (the data it manages), and the **clients** (your code and tools that talk to it).

```
┌─────────────────────────────────────────────────────────────────────┐
│                         YOUR MACHINE                                │
│                                                                     │
│  CLIENTS (how you talk to the server)                               │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                │
│  │ Your C# App  │ │    SSMS      │ │  HeidiSQL    │                │
│  │ (ADO.NET)    │ │ (SQL Server  │ │ (MariaDB     │                │
│  │              │ │  GUI only)   │ │  GUI)        │                │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘                │
│         │                │                │                         │
│         │ TCP connection │                │                         │
│         ▼                ▼                ▼                         │
│  SERVERS (programs running as services in the background)           │
│  ┌─────────────────────┐  ┌─────────────────────┐                  │
│  │ SQL Server          │  │ MariaDB              │                  │
│  │ (sqlservr.exe)      │  │ (mariadbd.exe)       │                  │
│  │ Port 1433           │  │ Port 3306            │                  │
│  │ TDS protocol        │  │ MySQL protocol       │                  │
│  │                     │  │                      │                  │
│  │ DATABASES:          │  │ DATABASES:           │                  │
│  │ ├── AppDb           │  │ ├── mydb             │                  │
│  │ ├── HR              │  │ ├── shop_db          │                  │
│  │ └── Testing         │  │ └── blog_db          │                  │
│  │                     │  │                      │                  │
│  │ Stored as:          │  │ Stored as:           │                  │
│  │ .mdf/.ldf files     │  │ .ibd files           │                  │
│  └─────────────────────┘  └─────────────────────┘                  │
│                                                                     │
│  EXCEPTION — no server needed:                                      │
│  ┌─────────────────────┐                                            │
│  │ SQLite              │                                            │
│  │ No service, no port │                                            │
│  │ Just a .db FILE     │                                            │
│  │ Your app reads it   │                                            │
│  │ directly            │                                            │
│  └─────────────────────┘                                            │
└─────────────────────────────────────────────────────────────────────┘
```


---

## What Is a Database Server?

A database server is a **program** — not a physical machine. It's software that:

1. **Installs** on your computer (or a remote server)
2. **Runs as a Windows Service** — starts at boot, runs 24/7 in the background
3. **Listens on a port** — waiting for connections
4. **Stores databases** — files on disk that only it can read/write
5. **Executes SQL** — your code sends SQL, the server processes it and returns results

```
You NEVER touch the database files directly.
You always go THROUGH the server.

Your Code → TCP Connection → Server Process → Disk Files
                                ↑
                         This is the gatekeeper.
                         Handles security, locking,
                         transactions, caching, etc.
```


---

## The Server Lifecycle

```
1. INSTALL         You download and install the software
                   (SQL Server Express, MariaDB, PostgreSQL)

2. SERVICE STARTS  Windows starts the service automatically at boot
                   sqlservr.exe / mariadbd.exe is now running
                   Listening on its port

3. IDLE            Server sits there waiting for connections
                   Uses some RAM and CPU, but mostly idle

4. CONNECTION      Your app calls conn.Open()
                   TCP connection established to the port
                   Server authenticates you

5. QUERY           Your app sends: "SELECT * FROM Users"
                   Server reads from disk/cache
                   Server sends results back over TCP

6. DISCONNECT      Your app calls conn.Close()
                   TCP connection returned to pool (or closed)
                   Server goes back to waiting

7. ALWAYS ON       Even when your app closes, the server keeps running
                   Other apps can connect at the same time
```


---

## One Server, Many Databases

A single server installation manages **multiple databases**. Each database is isolated — its own tables, users, permissions:

```
MariaDB Server (one installation on your machine)
│
├── Database: shop_db
│   ├── Table: products
│   ├── Table: orders
│   └── Table: customers
│
├── Database: blog_db
│   ├── Table: posts
│   ├── Table: comments
│   └── Table: authors
│
└── Database: test_db
    └── (whatever you're experimenting with)
```

You switch between them with `USE database_name;` in SQL or by specifying the database in your connection string.


---

## Server vs Client vs Protocol

Every database interaction has three parts:

| Part | What it is | Examples |
|---|---|---|
| **Server** | The program storing data | SQL Server, MariaDB, PostgreSQL |
| **Client** | The program connecting to the server | Your C# app, SSMS, HeidiSQL, DBeaver |
| **Protocol** | The language they speak over TCP | TDS (SQL Server), MySQL protocol (MariaDB/MySQL) |

A client can ONLY connect to a server that speaks the same protocol:

```
SSMS          → speaks TDS        → can connect to SQL Server ONLY
HeidiSQL      → speaks MySQL      → can connect to MariaDB/MySQL
                speaks TDS        → can also connect to SQL Server
DBeaver       → speaks everything → can connect to anything
Your C# app   → depends on which NuGet package (provider) you use
```


---

## Where Your C# Code Fits (ADO.NET)

ADO.NET is the **client library** in your C# code. The **provider** you install determines which server you can talk to:

```csharp
// MariaDB/MySQL — uses MySqlConnector provider
using var conn = new MySqlConnection(
    "Server=localhost;Port=3306;Database=mydb;User=root;Password=secret");

// SQL Server — uses Microsoft.Data.SqlClient provider
using var conn = new SqlConnection(
    "Server=localhost;Database=mydb;Integrated Security=true");

// PostgreSQL — uses Npgsql provider
using var conn = new NpgsqlConnection(
    "Host=localhost;Port=5432;Database=mydb;Username=postgres;Password=secret");

// SQLite — uses Microsoft.Data.Sqlite provider (no server!)
using var conn = new SqliteConnection("Data Source=mydb.db");
```

Same pattern every time: create connection → open → execute → close. The provider handles the protocol differences.

```
Your C# Code
    │
    │ uses ADO.NET API (DbConnection, DbCommand, DbDataReader)
    │
    ├── MySqlConnection ──── MySQL protocol ──── MariaDB/MySQL server
    ├── SqlConnection   ──── TDS protocol   ──── SQL Server
    ├── NpgsqlConnection ─── PG protocol    ──── PostgreSQL
    └── SqliteConnection ─── file I/O       ──── .db file (no server)
```


---

## SQL Server Installation Options

Since you have SSMS but no SQL Server:

| Option | What it is | Runs as service? | Memory usage | SSMS works? |
|---|---|---|---|---|
| **LocalDB** | Minimal SQL Server that starts on demand | No — starts when you connect, stops when idle | Very low | Yes — connect to `(localdb)\MSSQLLocalDB` |
| **Express** | Free full SQL Server with size limits | Yes — always running | ~200-500 MB | Yes — connect to `localhost` or `.\SQLEXPRESS` |
| **Developer** | Full SQL Server, all features | Yes — always running | ~500 MB+ | Yes |
| **Don't install** | Keep using MariaDB | N/A | N/A | No — use HeidiSQL instead |


---

## The "Local" Confusion

"Local database" is not a type of database. It just means the server runs on **your machine** instead of a remote one:

```
LOCAL                                    REMOTE

Server: localhost (your PC)              Server: db.mycompany.com (cloud/datacenter)
┌─────────────────────┐                 ┌─────────────────────┐
│ MariaDB on your PC  │                 │ MariaDB on AWS      │
│ Port 3306           │                 │ Port 3306           │
│ You manage it       │                 │ Someone else manages│
└─────────────────────┘                 └─────────────────────┘

Your code: "Server=localhost"            Your code: "Server=db.mycompany.com"

Same server software. Same protocol. Same SQL.
Only the address changes.
```

The **only exception** is SQLite — it's truly local with no server. Just a file your app reads directly.


---

## Summary

```
DATABASE SERVER = A program (service) that stores and manages databases
DATABASE        = A collection of tables managed by that server
CLIENT          = Any program that connects to the server (your app, SSMS, HeidiSQL)
PROTOCOL        = The language the client and server speak (TDS, MySQL protocol, etc.)
PROVIDER        = The NuGet package that teaches your C# app to speak a specific protocol

You MUST have the server running to use its databases.
SSMS ONLY talks to SQL Server.
Each server is its own world — can't see other servers' data.
"Local" just means the server is on your machine.
```


---

## See Also

- [[ADO.NET Overview]] — how your C# code connects to databases
- [[Data Providers]] — the NuGet packages for each database
- [[Connection Strings]] — how to specify which server and database to connect to
- [[Connection Pooling]] — how connections are reused for performance
