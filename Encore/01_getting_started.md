# 01\_getting\_started.md

## Getting Started with Encore

Encore is a backend development platform that streamlines building, running, and scaling cloud APIs. This guide helps you get started quickly with Encore on macOS.

---

### 1. Install Encore

Follow the official guide: [Encore Quick Start Docs](https://encore.dev/docs/ts/quick-start)

#### On macOS

Use Homebrew to install Encore:

```bash
brew install encoredev/tap/encore
```

---

### 2. Create a New Project

Initialize a new Encore project:

```bash
encore app create
```

Follow the prompts to set up your project.

---

### 3. Run the App

To run your Encore app locally:

```bash
encore run
```

---

### 4. Create an API

You can define your APIs following Encore’s conventions. (Refer to the official documentation for more details.)

---

### 5. Create a Service

Add a new file, for example: `encore.service.ts`

```typescript
import { Service } from "encore.dev/service";

export default new Service("foo");
```

This creates a simple Encore service named `"foo"`.

---

## Next Steps

* Explore defining API endpoints in your services.
* Check out [Encore documentation](https://encore.dev/docs/ts) for advanced usage.

---

**Tip:** Use `encore run` each time you make changes to see them reflected locally.
