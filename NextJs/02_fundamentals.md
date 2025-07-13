# 02\_fundamentals.md

## 1. Rendering Fundamentals

### Static Rendering (Build Time)

* **Definition:** Pages are rendered at build time and served as static files.
* **Pros:** Fast delivery, can be cached, good for SEO.
* **Cons:** Not suitable for frequently changing data.

### Dynamic Rendering (Request Time)

* **Definition:** Pages are rendered on each request (server-side or via API calls).
* **Pros:** Fresh data for every request, supports user-specific content.
* **Cons:** Slower, resource intensive, less SEO benefit unless using SSR with Next.js.

#### Common Problems (Especially with Dynamic Rendering)

* Large bundles
* Resource intensive
* No SEO (with client-side rendering only)
* Less secure

---

## 2. Caching Concepts

### Data Sources for Caching

* **Memory** (e.g., process memory)
* **File system** (local or edge disk cache)
* **Network** (CDN, upstream cache, etc.)

### Fetch API Caching in Next.js

#### **Dynamic Fetch (No Caching)**

```js
fetch('some-url', { cache: 'no-store' })
```

* Page becomes **dynamic** (built at request time).
* No cache is used.

#### **Static Fetch (Auto-Caching)**

```js
fetch('some-url')
```

* Page becomes **static** (built at build time).
* Data is **auto-cached**.

> **Note:**
>
> * Caching behavior described here is **only for the built-in `fetch`** in Next.js.
> * **Does not apply to Axios or other HTTP clients.**

---

## 3. Example Public API

Use for testing fetch and rendering concepts:

* [https://jsonplaceholder.typicode.com/](https://jsonplaceholder.typicode.com/)

---

## 4. CSS Modules

* Use CSS modules for component-scoped styles.
* Filename should match the component:
  Example:

  ```
  ProductCard.module.css  // for ProductCard.tsx
  ```

### Import Convention

```js
import styles from './ProductCard.module.css';
```

### Invoke Convention

```jsx
<div className={styles.card}>
  {/* Content */}
</div>
```

---

## 5. Naming Conventions

### CSS Classes

* **camelCase** is required for CSS module classes.
* **Correct:**

  ```css
  .cardContainer { /* ... */ }
  ```
* **Wrong:**

  ```css
  .card-container { /* ... */ }
  ```

---

## 6. Summary Table

| Concept        | Static | Dynamic       |
| -------------- | ------ | ------------- |
| Rendering Time | Build  | Request       |
| Caching        | Yes    | No            |
| SEO            | Good   | Depends (SSR) |
| Performance    | Fast   | Can be slow   |
