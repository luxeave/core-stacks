# 03_styling.md

## Global Styling

### globals.css

- Used to define global styles for your application.
- **Purpose:**  
  - Detect preferred theme (e.g., dark mode).
  - Set base styles for `body`, `#root`, etc.
- **Important:**  
  - Must import TailwindCSS directives at the top for Tailwind to be active.
  - Reserve `globals.css` for truly global styles only.

#### Example: Importing Tailwind in globals.css (for Tailwind v4)
```css
@import "tailwindcss";
````

---

## Utility Classes Reference

### Padding

* `p-[number]` : Padding on all sides.
* `px-[number]` : Padding left & right.
* `py-[number]` : Padding top & bottom.
* `pt-[number]` : Padding top only.

### Margin

* `m-[number]` : Margin on all sides.

### Text Size

* `text-xs` : Extra small text.
* `text-xl` : Extra large text.
* `text-2xl` : 2x extra large text.

### Colors

* `text-[color]` : Text color.
* `bg-[color]` : Background color.

### Font Thickness

* `font-thin` : Thin font.
* `font-light` : Light font.
* `font-bold` : Bold font.

---

### Example Component

```jsx
<div className='p-5 my-5 bg-sky-400 text-white text-xl hover:bg-sky-500'>
  Styled Component Example
</div>
```

---

## DaisyUI Integration

### Install DaisyUI

```bash
pnpm i -D daisyui@latest
```

### Tailwind Config

* For v4: No need to manually adjust `tailwind.config.ts`
* For previous versions or advanced use, see:
  [DaisyUI Config Docs](https://daisyui.com/docs/config/)

### globals.css (Tailwind v4+)

* Add DaisyUI as a plugin:

```css
@plugin "daisyui";
```

#### Example: Customizing Themes

```css
@plugin "daisyui" {
  themes: light --default, luxury --prefersdark;
};
```

* More info: [DaisyUI Themes](https://daisyui.com/docs/themes/)

---

## Productivity Shortcuts

* **Multiple select:**

  * `cmd + d` (VS Code) — select next matching word or selection.
* **HTML Table Emmet Shortcut:**

  * `thead>tr>th*2` (in editor) — expands to table header with two columns.

---

## Summary

* Use `globals.css` for essential global styles and theme detection.
* Import TailwindCSS directives at the top of your global stylesheet.
* Use utility classes for rapid styling: padding, margin, text, color, and font weight.
* Integrate DaisyUI for easy theming and component styling.
* Use keyboard/editor shortcuts to boost productivity.
