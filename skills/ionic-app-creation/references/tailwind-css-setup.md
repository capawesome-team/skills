# Tailwind CSS Setup for Ionic Apps

Set up Tailwind CSS v4 in an Ionic project. The installation steps differ based on the framework and build tool.

## Detect the Build Tool

Before proceeding, determine which build tool the project uses by checking the project root:

| File               | Build Tool | Frameworks              |
| ------------------ | ---------- | ----------------------- |
| `vite.config.ts`   | Vite       | React, Vue              |
| `angular.json`     | Angular CLI| Angular, Angular Standalone |

## Option A: Vite-Based Projects (React, Vue)

### A.1: Install Dependencies

```bash
npm install tailwindcss @tailwindcss/vite
```

### A.2: Configure the Vite Plugin

Open the `vite.config.ts` file in the project root and add the Tailwind CSS plugin:

```diff
+ import tailwindcss from '@tailwindcss/vite';
  import { defineConfig } from 'vite';
  // ... other imports

  export default defineConfig({
    plugins: [
+     tailwindcss(),
      // ... existing plugins
    ],
  });
```

Add the `tailwindcss()` call **before** other plugins in the `plugins` array.

### A.3: Import Tailwind CSS

Add the Tailwind CSS import to the **main CSS file** of the project. The location depends on the framework:

- **React**: `src/theme/variables.css` or `src/index.css`
- **Vue**: `src/theme/variables.css`

Add this line at the **top** of the file:

```diff
+ @import "tailwindcss";

  /* Existing CSS content below */
```

## Option B: Angular CLI Projects (Angular, Angular Standalone)

### B.1: Install Dependencies

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

### B.2: Create PostCSS Config

Create a `postcss.config.mjs` file in the project root:

```javascript
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

If a `postcss.config.js` or `.postcssrc.json` file already exists, update it instead of creating a new file.

### B.3: Import Tailwind CSS

Add the Tailwind CSS import to `src/global.scss` (or `src/styles.scss` if `global.scss` does not exist).

Add this line at the **top** of the file:

```diff
+ @import "tailwindcss";

  /* Existing CSS content below */
```

## Verify the Setup

After completing the setup, run the development server:

```bash
ionic serve
```

Open the app in the browser and add a Tailwind utility class to any element to verify it works:

```html
<p class="text-3xl font-bold text-blue-500">Tailwind CSS is working!</p>
```

If the text appears large, bold, and blue, Tailwind CSS is correctly configured. Remove the test markup after verification.
