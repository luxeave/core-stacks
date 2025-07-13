# Database Integration with Prisma

## Overview

This guide explains how to integrate a database into your Next.js (or Node.js) application using **Prisma**, a popular Object-Relational Mapper (ORM). It covers basic concepts, setup, and essential commands for day-to-day development.

---

## Key Concepts & Terms

- **Databases**: Permanent storage for application data.
- **Database Engines**: Examples include MySQL, PostgreSQL, MongoDB, etc.
- **Object-Relational Mapper (ORM)**: Maps database records to objects in code. Prisma is a widely-used ORM for Next.js/Node.js.
- **Model**: Represents an entity (e.g. User, Order) in your domain.
- **Migration**: A set of instructions (usually SQL) that updates the database schema to match your models.

---

## Why Use Prisma?

- Simplifies database operations with type-safe queries.
- Auto-generates types and client code.
- Handles migrations and model changes efficiently.

---

## Prisma Setup Guide

### 1. Install the Prisma VSCode Extension

For syntax highlighting and helpful autocompletion.

### 2. Install Prisma CLI and Package

```bash
pnpm add prisma
````

### 3. Initialize Prisma

```bash
npx prisma init
```

This creates:

* `prisma/` folder (contains `schema.prisma`)
* `.env` file for your database connection string

---

## Defining Your Data Models

In `prisma/schema.prisma`, define your models. Example:

```prisma
model User {
  id   Int    @id @default(autoincrement())
  name String
  // add more fields as needed
}
```

* Each model is an entity; each field is a property.

---

## Setting Up Database Connection

The database connection string is stored in `.env`:

```
DATABASE_URL="file:./dev.db"
```

* For production, replace with the actual database connection string (e.g. PostgreSQL, MySQL, etc).

### Adapter Versioning (For SQLite)

If using SQLite, make sure your adapter version matches the expected version:

* Example: `better-sqlite3: 11.10.0`

---

## Prisma Client Generation

Prisma generates a client that allows you to interact with your models in code.

**Recommended Pattern for Next.js:**

Create a file at `lib/prisma.ts`:

```ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

* This ensures a single Prisma client instance during development, avoiding connection issues due to hot reloads.

---

## Formatting Your Prisma Schema

For formatting and linting:

```bash
npx prisma format
```

---

## Running Migrations

Migrations update your database schema to match your models.

```bash
npx prisma migrate dev
```

* Creates/updates database tables as per your schema.
* Generates a new Prisma client.

---

## Further Help & Troubleshooting

* Official docs: [Prisma + Next.js](https://www.prisma.io/docs/orm/more/help-and-troubleshooting/nextjs-help)

---

## Quick Reference

| Task                       | Command / Location       |
| -------------------------- | ------------------------ |
| Install Prisma             | `pnpm add prisma`        |
| Initialize Prisma          | `npx prisma init`        |
| Format schema              | `npx prisma format`      |
| Run migrations             | `npx prisma migrate dev` |
| Prisma schema location     | `prisma/schema.prisma`   |
| Prisma client usage        | `lib/prisma.ts`          |
| Database connection string | `.env`                   |

---

## Summary

* Use Prisma as an ORM to map your database tables to TypeScript/JavaScript objects.
* Define models in `schema.prisma`.
* Use Prisma CLI to handle migrations and generate the client.
* Store your database connection string in `.env`.
* Use the generated Prisma client in your application code to perform database operations safely and efficiently.
