# Quick API Tutorial – URL Shortener

This guide helps you scaffold a basic URL shortener service using Encore, including service, API, database integration, and testing.

---

## 1. Create Service Directory and Files

### 1.1 Create the `url` Directory

```sh
mkdir url
```

### 1.2 Add Service Definition

Create `url/encore.service.ts`:

```ts
import { Service } from "encore.dev/service";

export default new Service("url");
```

This defines a new **service** called `url`.

---

## 2. Add API Endpoint

Create `url/url.ts` and add:

```ts
import { api } from "encore.dev/api";
import { randomBytes } from "node:crypto";

interface URL {
  id: string; // short-form URL id
  url: string; // complete URL, in long form
}

interface ShortenParams {
  url: string; // the URL to shorten
}

// Shortens a URL.
export const shorten = api(
  { method: "POST", path: "/url", expose: true },
  async ({ url }: ShortenParams): Promise<URL> => {
    const id = randomBytes(6).toString("base64url");
    return { id, url };
  },
);
```

---

## 3. Persist Data with SQL Database

### 3.1 Create Migration File

```sh
mkdir url/migrations
touch url/migrations/001_create_tables.up.sql
```

Add migration SQL to create the table (in `url/migrations/001_create_tables.up.sql`):

```sql
CREATE TABLE url (
  id TEXT PRIMARY KEY,
  original_url TEXT NOT NULL
);
```

### 3.2 Import Migration in Service

In `url/url.ts` (add at the top):

```ts
import { SQLDatabase } from "encore.dev/storage/sqldb";

// 'url' database is used to store the URLs that are being shortened.
const db = new SQLDatabase("url", { migrations: "./migrations" });
```

### 3.3 Update API to Insert into DB

Update the `shorten` function in `url/url.ts`:

```ts
export const shorten = api(
  { method: "POST", path: "/url", expose: true },
  async ({ url }: ShortenParams): Promise<URL> => {
    const id = randomBytes(6).toString("base64url");
    await db.exec`
      INSERT INTO url (id, original_url)
      VALUES (${id}, ${url})
    `;
    return { id, url };
  },
);
```

### 3.4 Verify Table in Infra Dashboard

* Check the **Infra** tab in Encore dashboard to confirm migration.

---

## 4. Retrieve Data

### 4.1 Access DB Shell

```sh
encore db shell url

select * from url;
```

---

## 5. Add Retrieve API Endpoint

### 5.1 Import Encore API Error

In `url/url.ts`:

```ts
import { api, APIError } from "encore.dev/api";
```

### 5.2 Add Retrieval Endpoint

Append to `url/url.ts`:

```ts
// Get retrieves the original URL for the id.
export const get = api(
  { expose: true, auth: false, method: "GET", path: "/url/:id" },
  async ({ id }: { id: string }): Promise<URL> => {
    const row = await db.queryRow`
      SELECT original_url FROM url WHERE id = ${id}
    `;
    if (!row) throw APIError.notFound("url not found");
    return { id, url: row.original_url };
  }
);
```

---

## 6. Add and Run Tests

### 6.1 Install Vitest

```sh
pnpm i --save-dev vitest
```

### 6.2 Add Test Script in `package.json`

```json
"scripts": {
  "test": "vitest"
}
```

### 6.3 Create Test File

`url/url.test.ts`:

```ts
import { describe, expect, test } from "vitest";
import { get, shorten } from "./url";

describe("shorten", () => {
  test("getting a shortened url should give back the original", async () => {
    const resp = await shorten({ url: "https://example.com" });
    const url = await get({ id: resp.id });
    expect(url.url).toBe("https://example.com");
  });
});
```

### 6.4 Run the Test

```sh
encore test
```

---

## 7. Configure for Deployment

### 7.1 Add `infra-config.json` to Project Root

```json
{
  "$schema": "https://encore.dev/schemas/infra.schema.json",
  "sql_servers": [
    {
      "host": "my-db-host:5432",
      "databases": {
        "url": {
          "username": "my-db-owner",
          "password": {"$env": "DB_PASSWORD"}
        }
      }
    }
  ]
}
```

---

# Summary of Key Steps

1. **Scaffold service and API** in `url/` folder
2. **Set up migrations** for SQL database
3. **Implement endpoints** for shortening and retrieving URLs
4. **Add automated tests** using Vitest
5. **Configure deployment** with `infra-config.json`
