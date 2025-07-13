# Quick Scaffold: Next.js + Tauri Desktop App

## 1. Prerequisites

* [Node.js (LTS)](https://nodejs.org/)
* [pnpm](https://pnpm.io/installation)
* [Rust](https://rustup.rs/)
* [Tauri Requirements](https://tauri.app/v1/guides/getting-started/prerequisites/) (includes Rust, Cargo, and OS-specific dependencies)

---

## 2. Create Next.js App

```sh
pnpm create next-app super-simple-note --ts --tailwind --eslint --app
cd super-simple-note
```

---

## 3. Add Tauri CLI

```sh
pnpm add -D @tauri-apps/cli
```

---

## 4. Initialize Tauri

```sh
pnpm tauri init
```

* Accept defaults or adjust as needed.
* This will add a `src-tauri` folder.

---

## 5. Adjust `tsconfig.json`

> Prevents TS from compiling the Rust code directory.

```jsonc
// tsconfig.json
{
  // ... rest of config ...
  "exclude": ["node_modules", "src-tauri"]
}
```

---

## 6. Adjust `next.config.ts` for Static Export

> Tauri works best with statically built assets.

```ts
// next.config.ts
const nextConfig = {
  output: 'export'
};

export default nextConfig;
```

---

## 7. Example Tauri Rust Command

Edit `src-tauri/src/lib.rs` to define a simple Tauri command:

```rust
#[tauri::command]
fn greet(name: &str, email: &str) -> String {
  format!("Hello, {}! Your email is {}", name, email)
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
  tauri::Builder::default()
    .setup(|app| {
      if cfg!(debug_assertions) {
        app.handle().plugin(
          tauri_plugin_log::Builder::default()
            .level(log::LevelFilter::Info)
            .build(),
        )?;
      }
      Ok(())
    })
    .invoke_handler(tauri::generate_handler![greet])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```

---

## 8. Build and Run

* **Development (hot-reload for Next.js):**

  ```sh
  pnpm dev
  pnpm tauri dev
  ```

  (Run these in separate terminals.)

* **Production Build:**

  ```sh
  pnpm build
  pnpm export
  pnpm tauri build
  ```

---

## 9. Common Tips

* Place your Next.js build output in the location expected by Tauri (`.output/public` or `out/` by default).
* All your static frontend files should be in the directory Tauri serves (`distDir` in `tauri.conf.json` if you customize).
* Use [Tauri API docs](https://tauri.app/v1/api/js/) to call Rust commands from your frontend.

---

## 10. Reference Links

* [Tauri + Next.js Docs](https://tauri.app/v1/guides/getting-started/setup-nextjs/)
* [Next.js Static Export](https://nextjs.org/docs/pages/building-your-application/deploying/static-exports)
* [Tauri Commands](https://tauri.app/v1/guides/features/command/)

---

**Ready!**
You now have a minimal, working Next.js app, bundled as a desktop app with Tauri.
Iterate as you build – adjust commands, add more features, and ship fast!
