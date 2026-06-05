---
tags: [security, permissions, authentication]
---

- Database security answers two fundamental questions: **who are you?** (authentication) and **what are you allowed to do?** (authorization). Getting these wrong doesn't just risk data leaks — it can mean regulatory fines, legal liability, and loss of customer trust. Security is not optional, and it is not something you add at the end.
- **Prerequisite:** [[03 - Backup and Recovery]]. Backups are part of your security posture — encrypted backups, access control to backup files, and the ability to recover from ransomware all depend on concepts from the previous note.

---

## Authentication — Who Are You?

- **Authentication** is the process of verifying identity. Before you can do anything in a database, you must prove who you are.

### Authentication Methods (SQL Server)

| Method                        | How It Works                                                              | When to Use                                                                                         | Security Level                                                           |
| ----------------------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **Windows Authentication**    | Your Windows/Active Directory login is used. No separate password needed. | Internal applications where the app server and database are in the same domain.                     | Highest — leverages Kerberos, no passwords stored in connection strings. |
| **SQL Server Authentication** | A username and password stored in SQL Server.                             | External applications, cross-platform clients (Linux, macOS), when Windows domain is not available. | Lower — passwords must be managed, can appear in connection strings.     |
| **Azure Active Directory**    | Microsoft's cloud identity platform. Supports MFA, conditional access.    | Cloud applications using Azure SQL Database.                                                        | High — modern identity with multi-factor authentication.                 |
| **Mixed Mode**                | Both Windows and SQL Authentication are enabled.                          | Most production environments — allows flexibility for different connection types.                   | Depends on configuration.                                                |

```sql
-- Check the current authentication mode:
SELECT SERVERPROPERTY('IsIntegratedSecurityOnly') AS WindowsOnlyAuth;
-- 1 = Windows Authentication only
-- 0 = Mixed Mode (Windows + SQL Authentication)
```

### Connection String Examples

```
-- Windows Authentication (no password in connection string):
Server=dbserver;Database=MyDb;Integrated Security=true;

-- SQL Server Authentication:
Server=dbserver;Database=MyDb;User Id=AppUser;Password=StrongP@ss123!;

-- Azure AD (with managed identity):
Server=dbserver.database.windows.net;Database=MyDb;Authentication=Active Directory Managed Identity;
```

```ad-important
Prefer **Windows Authentication** whenever possible. It eliminates passwords from connection strings, leverages existing Active Directory security (password policies, account lockout, MFA), and supports Kerberos delegation. SQL Authentication should be a fallback, not the default.
```

---

## Authorization — What Can You Do?

- Once the database knows who you are (authentication), it must determine what you're allowed to do (authorization). SQL Server uses a layered security model:

### Principals — WHO Can Act

- A **principal** is any entity that can request access to a resource.

| Principal Type | Scope | Description |
| --- | --- | --- |
| **Login** | Server level | Authenticates a user to the SQL Server instance. Created in the `master` database. |
| **User** | Database level | Maps to a login and grants access to a specific database. Created inside each database. |
| **Role** | Server or database level | A named group of permissions. Add users/logins to roles instead of granting permissions individually. |
| **Application Role** | Database level | Activated by the application with a password. Overrides the user's individual permissions for the session. |

```ad-note
The separation between **logins** and **users** is a common source of confusion. Think of it this way: a **login** gets you through the front door (server). A **user** gets you into a specific room (database). You need both. A login without a matching user in a database cannot access that database.
```

### Securables — WHAT Can Be Acted Upon

- A **securable** is any resource that can be protected with permissions.

| Scope | Examples |
| --- | --- |
| **Server** | Databases, logins, endpoints, server roles |
| **Database** | Tables, views, stored procedures, functions, schemas, database roles |
| **Schema** | All objects within a schema (e.g., everything in `dbo`) |
| **Object** | Individual tables, views, stored procedures |

---

## Creating Logins and Users

### Step 1: Create a Login (Server Level)

```sql
-- SQL Authentication login:
CREATE LOGIN AppUser WITH PASSWORD = 'StrongP@ss123!';

-- Windows Authentication login:
CREATE LOGIN [DOMAIN\ServiceAccount] FROM WINDOWS;

-- With password policy enforcement:
CREATE LOGIN AppUser WITH PASSWORD = 'StrongP@ss123!',
    CHECK_POLICY = ON,           -- enforce Windows password policy
    CHECK_EXPIRATION = ON,       -- password expires per policy
    DEFAULT_DATABASE = MyDb;
```

### Step 2: Create a User in a Database

```sql
USE MyDb;

-- Map the login to a database user:
CREATE USER AppUser FOR LOGIN AppUser;

-- You can give the user a different name from the login (but don't — it's confusing):
CREATE USER AppDatabaseUser FOR LOGIN AppUser;

-- User with a default schema:
CREATE USER AppUser FOR LOGIN AppUser WITH DEFAULT_SCHEMA = dbo;
```

### Viewing Existing Logins and Users

```sql
-- All logins on the server:
SELECT name, type_desc, is_disabled
FROM sys.server_principals
WHERE type IN ('S', 'U')  -- S = SQL login, U = Windows login
ORDER BY name;

-- All users in the current database:
SELECT name, type_desc, default_schema_name
FROM sys.database_principals
WHERE type IN ('S', 'U')
ORDER BY name;
```

---

## Granting and Revoking Permissions

- Permissions control what a principal can do with a securable. There are three permission statements:

| Statement | Effect |
| --- | --- |
| `GRANT` | Gives a permission. The principal can now do the action. |
| `REVOKE` | Removes a previously granted (or denied) permission. Returns to the default state (no permission). |
| `DENY` | Explicitly blocks a permission. **Overrides GRANT** — even if the user is in a role that has the permission, DENY wins. |

```ad-important
The precedence order is: **DENY > GRANT > (no permission)**. A DENY on a user overrides a GRANT on any role the user belongs to. Use DENY sparingly and deliberately — it can create confusing permission situations when combined with role memberships.
```

### Granting Permissions on Objects

```sql
-- Grant specific permissions on a table:
GRANT SELECT, INSERT, UPDATE ON dbo.Users TO AppUser;

-- Grant EXECUTE on a stored procedure:
GRANT EXECUTE ON dbo.sp_GetUsers TO AppUser;

-- Grant SELECT on all tables in a schema:
GRANT SELECT ON SCHEMA::dbo TO AppUser;

-- Grant with the ability to grant to others:
GRANT SELECT ON dbo.Users TO TeamLead WITH GRANT OPTION;
```

### Revoking Permissions

```sql
-- Remove a previously granted permission:
REVOKE DELETE ON dbo.Users FROM AppUser;

-- Revoke cascades if the user had WITH GRANT OPTION and granted to others:
REVOKE SELECT ON dbo.Users FROM TeamLead CASCADE;
```

### Denying Permissions

```sql
-- Explicitly block a permission (overrides any GRANT):
DENY DROP TABLE TO AppUser;
DENY DELETE ON dbo.AuditLog TO AppUser;  -- no one should delete audit records
```

### Viewing Effective Permissions

```sql
-- What permissions does AppUser actually have on a table?
EXECUTE AS USER = 'AppUser';
SELECT * FROM fn_my_permissions('dbo.Users', 'OBJECT');
REVERT;

-- Or for the current user:
SELECT * FROM fn_my_permissions('dbo.Users', 'OBJECT');
```

---

## Database Roles — Permission Groups

- Instead of granting permissions to each user individually, assign users to **roles**. Roles bundle permissions and make administration manageable.

### Built-in Database Roles

| Role | Permissions | Use Case |
| --- | --- | --- |
| **db_datareader** | SELECT on all tables and views | Read-only reporting accounts |
| **db_datawriter** | INSERT, UPDATE, DELETE on all tables | Accounts that need to modify data |
| **db_ddladmin** | CREATE, ALTER, DROP tables and other objects | Schema management accounts |
| **db_securityadmin** | Manage role membership and permissions | Security management |
| **db_backupoperator** | BACKUP DATABASE and BACKUP LOG | Backup service accounts |
| **db_owner** | **Everything** — full control over the database | Only for administrators (dangerous for applications) |
| **public** | Default role — all users are members. Has minimal permissions by default. | Do not grant extra permissions to public — it affects everyone. |

```sql
-- Add a user to a role:
ALTER ROLE db_datareader ADD MEMBER AppUser;
ALTER ROLE db_datawriter ADD MEMBER AppUser;

-- Remove a user from a role:
ALTER ROLE db_datawriter DROP MEMBER AppUser;

-- Check role membership:
SELECT 
    dp.name AS user_name,
    r.name AS role_name
FROM sys.database_role_members rm
JOIN sys.database_principals dp ON rm.member_principal_id = dp.principal_id
JOIN sys.database_principals r ON rm.role_principal_id = r.principal_id
ORDER BY r.name, dp.name;
```

### Custom Roles

- For more granular control, create your own roles:

```sql
-- Create a role for the web application:
CREATE ROLE WebAppRole;

-- Grant specific permissions to the role:
GRANT SELECT, INSERT, UPDATE ON dbo.Orders TO WebAppRole;
GRANT SELECT ON dbo.Products TO WebAppRole;
GRANT EXECUTE ON dbo.sp_PlaceOrder TO WebAppRole;
-- Note: no DELETE permission — the web app can't delete orders

-- Add users to the role:
ALTER ROLE WebAppRole ADD MEMBER WebAppUser;
ALTER ROLE WebAppRole ADD MEMBER ApiServiceUser;
```

```ad-note
Custom roles are the preferred approach for production applications. Built-in roles like `db_datareader` grant access to **all** tables, including ones you might add in the future. Custom roles give you explicit control — a new table is inaccessible until you deliberately grant permissions.
```

### Server-Level Roles

| Role | Permissions |
| --- | --- |
| **sysadmin** | Unrestricted access to everything (equivalent to `sa`) |
| **serveradmin** | Server configuration settings |
| **securityadmin** | Manage logins and permissions |
| **dbcreator** | Create, alter, drop databases |
| **bulkadmin** | Run BULK INSERT |

```sql
-- Add a login to a server role:
ALTER SERVER ROLE dbcreator ADD MEMBER DevOpsLogin;
```

```ad-warning
The **sysadmin** role and the **sa** login have unrestricted access to every database on the server. They bypass all permission checks. Never use `sa` for application connections. Never add application accounts to `sysadmin`. These should be reserved for a small number of trusted database administrators.
```

---

## Principle of Least Privilege

- The single most important security concept: **give every account only the minimum permissions it needs to do its job**. Nothing more.

```
BAD:  Web app connects as sa
      → Can do ANYTHING: drop databases, read other databases,
        create logins, shut down the server

BAD:  Web app connects as db_owner
      → Can drop tables, alter schema, add users, change permissions

GOOD: Web app connects with a custom role
      → SELECT, INSERT, UPDATE on specific tables
      → EXECUTE on specific stored procedures
      → Cannot DROP anything, cannot access other tables
```

### Practical Implementation

```sql
-- 1. Create a login for the application:
CREATE LOGIN WebApp WITH PASSWORD = 'V3ry$tr0ngP@ssw0rd!';

-- 2. Create a user in the specific database:
USE MyDb;
CREATE USER WebApp FOR LOGIN WebApp;

-- 3. Create a custom role with minimal permissions:
CREATE ROLE WebAppRole;
GRANT SELECT ON dbo.Products TO WebAppRole;
GRANT SELECT, INSERT ON dbo.Orders TO WebAppRole;
GRANT SELECT, INSERT ON dbo.OrderItems TO WebAppRole;
GRANT SELECT ON dbo.Customers TO WebAppRole;
GRANT UPDATE (email, phone) ON dbo.Customers TO WebAppRole;  -- column-level!
GRANT EXECUTE ON dbo.sp_PlaceOrder TO WebAppRole;

-- 4. Add the user to the role:
ALTER ROLE WebAppRole ADD MEMBER WebApp;

-- 5. Explicitly deny dangerous permissions:
DENY ALTER ON SCHEMA::dbo TO WebAppRole;
DENY CREATE TABLE TO WebAppRole;
```

- Notice the column-level `GRANT UPDATE (email, phone)` — this allows the app to update only those two columns on the Customers table, not the `credit_limit` or `is_admin` columns. This is granular security.

---

## SQL Injection — The #1 Database Security Threat

- **SQL injection** occurs when an attacker inserts malicious SQL code into a query through unvalidated user input. It is consistently ranked as one of the top security vulnerabilities (OWASP Top 10).

### How the Attack Works

```csharp
// VULNERABLE CODE — string concatenation:
string query = "SELECT * FROM users WHERE username = '" + username + "'";
// If username = "admin' OR '1'='1" then the query becomes:
// SELECT * FROM users WHERE username = 'admin' OR '1'='1'
// This returns ALL users — authentication bypass!

// Even worse — if username = "'; DROP TABLE users; --"
// SELECT * FROM users WHERE username = ''; DROP TABLE users; --'
// This DELETES the entire users table!
```

### The Fix — Parameterized Queries

```csharp
// SAFE CODE — parameterized query:
string query = "SELECT * FROM users WHERE username = @Username";
cmd.CommandText = query;
cmd.Parameters.AddWithValue("@Username", username);
// The parameter value is NEVER interpreted as SQL — it's treated as pure data
// Even if username = "'; DROP TABLE users; --", it searches for a user
// literally named "'; DROP TABLE users; --" (and finds nothing)
```

- The key distinction: with string concatenation, user input becomes part of the SQL **code**. With parameters, user input is treated as **data** and never executed.

```ad-warning
**Every single SQL query** that includes user input must use parameters. No exceptions. It doesn't matter if you "validate" the input first — validation can be bypassed. Parameterized queries are the only reliable defense against SQL injection. See [[Parameterized Queries]] for comprehensive C# examples.
```

### Additional Defenses (Defense in Depth)

- Parameterized queries are the primary defense. These additional measures add layers:

| Defense | What It Does |
| --- | --- |
| **Least privilege** | Even if injection succeeds, the account can only do what it's permitted to. An account with only SELECT on two tables can't DROP anything. |
| **Stored procedures** | Application calls stored procedures instead of sending raw SQL. Grant EXECUTE only — no direct table access. |
| **Input validation** | Validate data types, lengths, and formats at the application layer. Not a substitute for parameterization, but reduces attack surface. |
| **Web Application Firewall (WAF)** | Detects and blocks common injection patterns at the network level. |
| **Error message suppression** | Don't expose SQL error messages to users. Detailed errors help attackers understand the database structure. Log errors server-side, show generic messages to users. |

---

## Encryption

- Encryption protects data from unauthorized access even if someone gains access to the physical files, network traffic, or backup media.

### Encryption in Transit — TLS/SSL

- Protects data as it travels between the application and the database server. Without it, someone sniffing the network can read every query and every result in plain text.

```
-- Connection string with encryption enabled:
Server=dbserver;Database=MyDb;User Id=AppUser;Password=P@ss;Encrypt=true;TrustServerCertificate=false;
```

| Setting | Effect |
| --- | --- |
| `Encrypt=true` | Forces TLS encryption on the connection |
| `TrustServerCertificate=false` | Validates the server's certificate against a trusted CA. Set to `true` only in development (self-signed certs). |

```ad-note
SQL Server 2022 and later enable connection encryption by default. Older versions require explicit configuration. Always use `Encrypt=true` in production connection strings — there is no valid reason to send database traffic unencrypted.
```

### Encryption at Rest — Transparent Data Encryption (TDE)

- **TDE** encrypts the physical database files (`.mdf`, `.ldf`) and backup files on disk. If someone steals the disk or backup file, they cannot read the data without the encryption key.

```sql
-- Enable TDE (one-time setup):

-- Step 1: Create a master key in the master database
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'MasterKeyP@ssw0rd!';

-- Step 2: Create a certificate to protect the database encryption key
CREATE CERTIFICATE TDECert WITH SUBJECT = 'TDE Certificate';

-- Step 3: Create the database encryption key in the target database
USE MyDb;
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE TDECert;

-- Step 4: Enable encryption
ALTER DATABASE MyDb SET ENCRYPTION ON;
```

- TDE is **transparent** — applications don't need any changes. The DBMS handles encryption and decryption automatically. There is a small CPU overhead (typically 2-5%).

```ad-important
If you enable TDE, you **must** back up the certificate and its private key immediately. Without the certificate, you cannot restore the database on a different server or after a server rebuild. Store the certificate backup securely, separate from the database backups.
```

```sql
-- CRITICAL: Back up the TDE certificate
BACKUP CERTIFICATE TDECert
TO FILE = 'C:\CertBackup\TDECert.cer'
WITH PRIVATE KEY (
    FILE = 'C:\CertBackup\TDECert_PrivateKey.pvk',
    ENCRYPTION BY PASSWORD = 'CertBackupP@ss!'
);
```

### Column-Level Encryption — Always Encrypted

- **Always Encrypted** protects sensitive columns (Social Security numbers, credit card numbers, medical records) so that even the database administrator cannot read the plaintext. The encryption keys are held by the application, not the database.

| Feature | TDE | Always Encrypted |
| --- | --- | --- |
| **What's encrypted** | Entire database files on disk | Specific columns |
| **Who holds the key** | The database server | The application / client |
| **DBA can see plaintext** | Yes (data is decrypted in memory for queries) | No (data is encrypted end-to-end) |
| **Application changes** | None | Requires driver and connection string changes |
| **Performance impact** | Low (2-5% CPU) | Higher (encryption per operation, limited query capability on encrypted columns) |
| **Use case** | General at-rest protection | Protecting specific sensitive data from all access, including DBAs |

```ad-note
Always Encrypted has significant limitations: you cannot perform comparisons, JOINs, LIKE, or range queries on encrypted columns (except with deterministic encryption, which allows equality comparisons only). Design your schema and queries with these limitations in mind before enabling it.
```

---

## Security Audit and Monitoring

- Security is not a one-time configuration. You must continuously monitor who is doing what.

### SQL Server Audit

```sql
-- Create a server-level audit (writes to a file):
CREATE SERVER AUDIT SecurityAudit
TO FILE (FILEPATH = 'C:\AuditLogs\', MAXSIZE = 100 MB);
ALTER SERVER AUDIT SecurityAudit WITH (STATE = ON);

-- Create a database audit specification (track specific events):
USE MyDb;
CREATE DATABASE AUDIT SPECIFICATION DataAccessAudit
FOR SERVER AUDIT SecurityAudit
ADD (SELECT, INSERT, UPDATE, DELETE ON dbo.Customers BY public)
WITH (STATE = ON);
```

### Monitor Failed Logins

```sql
-- Check for failed login attempts (stored in SQL Server error log):
EXEC xp_readerrorlog 0, 1, N'Login failed';
```

### Review Permissions Regularly

```sql
-- Find all permissions granted to a specific user:
SELECT 
    dp.name AS principal_name,
    dp.type_desc AS principal_type,
    o.name AS object_name,
    p.permission_name,
    p.state_desc AS permission_state  -- GRANT, DENY, REVOKE
FROM sys.database_permissions p
JOIN sys.database_principals dp ON p.grantee_principal_id = dp.principal_id
LEFT JOIN sys.objects o ON p.major_id = o.object_id
WHERE dp.name = 'AppUser'
ORDER BY o.name, p.permission_name;

-- Find users with db_owner or sysadmin (should be a short list):
SELECT 
    dp.name AS user_name,
    r.name AS role_name
FROM sys.database_role_members rm
JOIN sys.database_principals dp ON rm.member_principal_id = dp.principal_id
JOIN sys.database_principals r ON rm.role_principal_id = r.principal_id
WHERE r.name IN ('db_owner');
```

---

## Security Checklist

- A practical checklist for securing a SQL Server database:

1. **Use Windows Authentication** where possible. Fall back to SQL Authentication only when necessary.
2. **Disable the `sa` account** or rename it. Never use it for application connections.
3. **Apply the principle of least privilege** — create custom roles with minimum required permissions.
4. **Use parameterized queries** in all application code. No exceptions.
5. **Enable TLS encryption** on all connections (`Encrypt=true`).
6. **Enable TDE** on production databases to encrypt data at rest.
7. **Back up the TDE certificate** and store it securely, separate from database backups.
8. **Use Always Encrypted** for highly sensitive columns (SSN, credit cards).
9. **Enable login auditing** — track both successful and failed logins.
10. **Review permissions regularly** — remove accounts that are no longer needed, audit role memberships.
11. **Keep SQL Server patched** — security updates fix known vulnerabilities.
12. **Don't expose the database port (1433) to the internet** — use a firewall, VPN, or application tier in between.
13. **Don't store connection strings with passwords in source control** — use environment variables, Azure Key Vault, or Windows Credential Manager.

```ad-warning
Never use the `sa` account for application connections. Never grant `db_owner` to application accounts. Never concatenate user input into SQL strings. These three rules alone prevent the vast majority of database security incidents.
```

---

**Previous:** [[03 - Backup and Recovery]]
