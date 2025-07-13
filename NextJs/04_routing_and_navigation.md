# 04\_routing.md

## File System Based Routing in Next.js

Next.js uses a **file system-based router**, where the structure of your files and folders under the `app/` directory determines your app’s routes. Here are the key concepts and files:

---

## # Special Files

| File Name       | Purpose                                                         |
| --------------- | --------------------------------------------------------------- |
| `page.tsx`      | The page component rendered for a given route                   |
| `layout.tsx`    | Shared layout/scaffold for one or more pages, acts as a wrapper |
| `loading.tsx`   | Loading UI shown while page/route is being fetched              |
| `route.tsx`     | Route handlers for APIs (in App Router)                         |
| `not-found.tsx` | Custom 404 page for the segment                                 |
| `error.tsx`     | Custom error UI for unhandled errors in a route segment         |

---

## # Folder Structuring Your Components

* Components used **only in a specific route** can live inside that route’s folder (under `app/`).
* **Shared components** (used in multiple places) should be moved to a top-level `/components` directory.

---

## # Dynamic Routes

Dynamic routes are created using brackets:

* Example: `/users/[id]/page.tsx` matches `/users/123` or `/users/abc`

#### Passing Props as Path Params

* Props are passed as part of the path (similar to query params).
* **Next.js 14+**: If you use async functions, **params must be awaited**.

#### Example

```tsx
import React from 'react';

// For a segment like /users/[id], "params" is passed in
interface Props {
  params: { id: string }
}

const UserDetailPage = ({ params: { id } }: Props) => {
  return (
    <div>UserDetailPage {id}</div>
  )
}

export default UserDetailPage;
```

#### Gotcha

* **You cannot nest the same param name:**
  `/users/[id]/[id]/page.tsx` is **NOT allowed**.

---

## # Catch-All Segments

To match multiple segments, use double or triple brackets:

* `[...slug]` or `[[...slug]]` captures multiple path segments as an **array**.
* Useful for building things like `/docs/a/b/c` → params.slug = \[“a”, “b”, “c”]

---

## # Layouts (Shared Look)

* `layout.tsx` provides a scaffold for the children routes.
* `page.tsx` is **embedded within** its parent `layout.tsx`.

**Example:**

```tsx
// layout.tsx
export default function Layout({ children }) {
  return (
    <div>
      <Sidebar />
      <main>{children}</main>
    </div>
  )
}
```

---

## # Navigation

* Use **`<Link>`** from `next/link` for client-side navigation.

  * Downloads only the content of the target page.
  * Pre-fetches links visible in the viewport.
  * Caches pages on the client (cache cleared on full reload).

* **Strict mode** (React 18):

  * In development, components may render twice to help find bugs.

---

## # Programmatic Navigation

* Only available inside **client components** (`'use client';`).
* Use `useRouter()` from `next/navigation` for programmatic route changes.

---

## # Loading UI

* **Suspense** can wrap components to show fallback UIs during loading.
* At the route level, create a `loading.tsx` file for built-in loading UI.
* Loading UI can use DaisyUI or any style you want.

---

## # Global Suspense

* Add a `<Suspense>` boundary in your root `layout.tsx` to show loading state globally.

---

## # Handling Not Found

* Add a `not-found.tsx` file to customize the 404 experience for a segment.

---

## # Handling Unexpected Errors

* Add an `error.tsx` file to customize the error experience for a segment.

  * Can display a retry button or custom messaging.
  * Can be applied to any route/segment.

---

## # Chrome Extension

* Use **React Developer Tools** for inspecting component state/props, including Next.js routing context.

---

## Example Folder Structure

```
/app
  /users
    /[id]
      page.tsx         // User detail page
      loading.tsx      // Shows while fetching
      not-found.tsx    // Custom 404 for missing users
      error.tsx        // Custom error UI
  layout.tsx           // App-wide layout
  page.tsx             // Home page
/components            // Shared components
```

---

## # Summary

* Next.js routing is file-system based.
* Use special files for layouts, loading, error handling, and catch-all routes.
* Props for dynamic and catch-all routes are accessed via `params`.
* Use `<Link>`, Suspense, and client-side navigation for best UX.
* Structure components based on usage frequency (route-local vs shared).
