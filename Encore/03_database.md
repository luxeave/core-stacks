Certainly! Here’s a structured **03\_Database.md** guide for handling database setup, migration files, and extracting credentials with Encore. The steps are ordered for quick access and daily developer use.

---

# 03\_Database.md

## Database Setup with Encore

### 1. Creating Migration Files

Organize your migration scripts in a dedicated directory. Each migration is a `.sql` file describing schema changes.

**Recommended naming:**
`1_first_migration.up.sql`
Increment numbers and use underscores for readability.

#### **Example Structure**

```
url/
  migrations/
    1_first_migration.up.sql
    2_add_user_table.up.sql
```

### 2. Sample Migration File (`migrations.sql`)

A migration file contains raw SQL statements:

```sql
-- 1_first_migration.up.sql
CREATE TABLE urls (
  id TEXT PRIMARY KEY,
  url TEXT NOT NULL
);
```

### 3. Creating a Migration File

```bash
mkdir -p url/migrations
touch url/migrations/1_first_migration.up.sql
```

Edit the file as needed.

---

## Running Migrations

Encore can automatically apply your migrations when you run or deploy your app. If you want to run migrations manually (for local testing or troubleshooting), use:

```bash
encore db migrate
```

Or, for a specific service:

```bash
encore db migrate --service url
```

---

## Extracting Database Credentials

Encore provisions and manages the database for your service. To connect directly (e.g., with a client or migration tool), extract the **connection URI**:

```bash
encore db conn-uri <db-name>
# Example:
encore db conn-uri todo
```

This command outputs the connection string you can use with external tools.

---

## Calling Migrations from Code

Encore auto-applies migrations on deploy/run. No explicit code invocation is needed, but ensure your migration files are in the right place (e.g., `url/migrations`).

If you wish to call or check the status programmatically, refer to the [Encore documentation](https://encore.dev/docs/ts/database-migrations) or use Encore CLI as shown above.

---

## Migration File Format

* **Name:** `{number}_{description}.up.sql`
  (E.g., `1_first_migration.up.sql`, `2_add_users_table.up.sql`)
* **Content:** Raw SQL to migrate the schema upwards.

---

## Quick Reference

* **Create migration directory:**
  `mkdir -p url/migrations`
* **Create migration file:**
  `touch url/migrations/1_first_migration.up.sql`
* **Get connection URI:**
  `encore db conn-uri <db-name>`
* **Apply migrations:**
  `encore db migrate`

---

**Useful links:**

* [Encore: Database Migrations](https://encore.dev/docs/ts/database-migrations)

---

Let me know if you want a sample SQL for a different schema or more examples for migration files!
