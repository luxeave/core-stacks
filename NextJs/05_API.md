# 05\_API.md

## Building APIs in Next.js

APIs (Application Programming Interfaces) allow your frontend or external clients to communicate with your application’s backend. In Next.js, building APIs is straightforward and follows a file-based routing convention.

---

## 1. API Route Files

* To create an API endpoint, add a `route.ts` (or `route.tsx`) file inside a directory under the `/app` or `/api` folder.
* **Important:** Within a single directory, you can have either a `page` file or a `route` file, **not both**.

**Example Structure:**

```
/app/api/todos/route.ts
```

---

## 2. Route Handlers

A route handler is a function that responds to an HTTP request. You can define handlers for different HTTP methods:

* `GET`: Retrieve data
* `POST`: Create new data
* `PUT` / `PATCH`: Update data
* `DELETE`: Remove data

**Basic Example:**

```ts
// /app/api/todos/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  // handle GET requests
  return NextResponse.json({ message: 'List todos' });
}

export async function POST(request: NextRequest) {
  // handle POST requests
  return NextResponse.json({ message: 'Create todo' }, { status: 201 });
}
```

---

## 3. HTTP Status Codes

* `200`: Success
* `201`: Resource created
* `400`: Bad request (client error, e.g., invalid data)
* `404`: Not found
* `500`: Internal server error

**Tip:** Use appropriate status codes for clear API communication.

---

## 4. Data Validation

* **Always validate data sent by clients!**
* For simple cases, use `if` statements.
* For complex schemas, use libraries like [Zod](https://zod.dev).

**Example with Zod:**

```ts
import { z } from 'zod';

const TodoSchema = z.object({
  title: z.string().min(1),
  completed: z.boolean().optional(),
});

export async function POST(request: NextRequest) {
  const body = await request.json();
  const result = TodoSchema.safeParse(body);

  if (!result.success) {
    return NextResponse.json({ error: result.error.errors }, { status: 400 });
  }

  // proceed with valid data
}
```

---

## 5. Common HTTP Methods & Usage

* **POST**:

  * Create a new resource.
  * Send data in the request body.
* **PUT** / **PATCH**:

  * Update an existing resource.
  * PUT: Replace the whole object.
  * PATCH: Update one or more properties.
* **DELETE**:

  * Delete a resource.
  * Request body should usually be empty.

---

## 6. Testing APIs with Postman

* [Postman](https://www.postman.com/) is a popular tool for API testing.
* Allows you to:

  * Send various HTTP requests to your API endpoints
  * Inspect request/response headers, bodies, and status codes
  * Automate API tests

---

## 7. Key Terms

* **API Endpoint**: The URL path where the API is accessible, e.g., `/api/todos`
* **Route Handler**: The function responding to a specific HTTP method at an endpoint
* **HTTP Method**: Defines the intended action (GET, POST, PUT, PATCH, DELETE)
* **Status Code**: Number indicating the result of the request
* **Data Validation**: Ensuring client-sent data matches your expected schema

---

## 8. Summary

* API endpoints are created with `route.ts(x)` files.
* Only one `page` or `route` file per directory.
* Use route handlers for different HTTP methods.
* Return proper status codes.
* **Always** validate incoming data (prefer libraries like Zod).
* Use Postman for manual API testing and inspection.

---

**For more, see:**

* [Next.js API Routes Docs](https://nextjs.org/docs/app/building-your-application/routing/router-handlers)
* [Zod Validation Library](https://zod.dev)
* [Postman API Platform](https://www.postman.com/)
