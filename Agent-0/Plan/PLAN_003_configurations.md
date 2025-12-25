# Project Setup: Configurations

**ID:** PLAN_003 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_005/04_configurations.md](../Topic/TOPIC_005_project_setup/04_configurations.md)

---

## 🎯 Tujuan

Membuat semua configuration files untuk ORBIT CLI — mencakup TypeScript, bundler, linter, formatter, testing, dan git.

---

## 📁 Working Directory

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_001 selesai (package.json, node_modules)
- [x] PLAN_002 selesai (src/ folder structure)

---

## 🛠️ Strategi Implementasi

### Step 1: Create tsconfig.json

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/tsconfig.json`:

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022"],
    "outDir": "dist",
    "rootDir": "src",
    "declaration": true,
    "declarationMap": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noUncheckedIndexedAccess": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "isolatedModules": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

**Validation:** `cat tsconfig.json | head -10`

---

### Step 2: Create tsup.config.ts

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/tsup.config.ts`:

```typescript
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"],
  target: "node18",
  outDir: "dist",
  clean: true,
  dts: true,
  sourcemap: true,
  splitting: true,
  treeshake: true,
  minify: false,
  banner: {
    js: "#!/usr/bin/env node",
  },
  external: ["figlet"],
  define: {
    "process.env.NODE_ENV": JSON.stringify("production"),
  },
});
```

**Validation:** `cat tsup.config.ts | head -5`

---

### Step 3: Create eslint.config.js

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/eslint.config.js`:

```javascript
import eslint from "@eslint/js";
import tseslint from "typescript-eslint";

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        project: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      "@typescript-eslint/no-unused-vars": [
        "error",
        { argsIgnorePattern: "^_", varsIgnorePattern: "^_" },
      ],
      "@typescript-eslint/explicit-function-return-type": [
        "warn",
        { allowExpressions: true },
      ],
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/prefer-readonly": "error",
      "@typescript-eslint/no-floating-promises": "error",
      "@typescript-eslint/await-thenable": "error",
      "@typescript-eslint/no-misused-promises": "error",
      "no-console": ["warn", { allow: ["warn", "error"] }],
      "prefer-const": "error",
      "no-var": "error",
    },
  },
  {
    ignores: ["dist/", "node_modules/", "*.config.*", "coverage/"],
  }
);
```

**Validation:** `cat eslint.config.js | head -5`

---

### Step 4: Create .prettierrc

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/.prettierrc`:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100,
  "arrowParens": "always",
  "endOfLine": "lf",
  "bracketSpacing": true,
  "useTabs": false
}
```

**Validation:** `cat .prettierrc`

---

### Step 5: Create .prettierignore

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/.prettierignore`:

```
dist/
node_modules/
coverage/
*.md
```

**Validation:** `cat .prettierignore`

---

### Step 6: Create vitest.config.ts

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/vitest.config.ts`:

```typescript
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "node",
    include: ["src/**/*.test.ts"],
    exclude: ["node_modules", "dist"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      include: ["src/**/*.ts"],
      exclude: ["src/**/*.test.ts", "src/index.ts"],
    },
    globals: true,
    testTimeout: 10000,
  },
  resolve: {
    alias: {
      "@": "./src",
    },
  },
});
```

**Validation:** `cat vitest.config.ts | head -5`

---

### Step 7: Create .gitignore

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/.gitignore`:

```gitignore
# Dependencies
node_modules/

# Build output
dist/

# IDE
.idea/
.vscode/
*.swp
*.swo
.cursorrules

# OS
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*

# Test coverage
coverage/

# Environment
.env
.env.local
.env.*.local

# Cache
.eslintcache
.prettiercache
*.tsbuildinfo
```

**Validation:** `cat .gitignore | head -10`

---

### Step 8: Create .editorconfig

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/.editorconfig`:

```editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
```

**Validation:** `cat .editorconfig`

---

### Step 9: Create .nvmrc

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/.nvmrc`:

```
20
```

**Validation:** `cat .nvmrc`

---

### Step 10: Create README.md

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/README.md`:

````markdown
# ORBIT CLI

> Universal Project Generator CLI

## Installation

```bash
npm install -g orbit-cli
```
````

## Usage

```bash
orbit create my-app
orbit list
orbit doctor
```

## License

MIT

````

**Validation:** `cat README.md`

---

### Step 11: Create LICENSE

Create `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/LICENSE`:

MIT License file with current year.

---

### Step 12: Verify All Config Files

```bash
ls -la *.json *.ts *.js .* 2>/dev/null | grep -v node_modules
````

Expected: 10+ config files

---

## ✅ Kriteria Sukses

- [x] tsconfig.json created
- [x] tsup.config.ts created
- [x] eslint.config.js created
- [x] .prettierrc created
- [x] .prettierignore created
- [x] vitest.config.ts created
- [x] .gitignore created
- [x] .editorconfig created
- [x] .nvmrc created
- [x] README.md created
- [x] LICENSE created
- [x] `npm run typecheck` passes ✅

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_004_initial_files**

---

## 🔗 Terkait

Topic: [TOPIC_005](../Topic/TOPIC_005_project_setup/_main.md)
Ref: [04_configurations.md](../Topic/TOPIC_005_project_setup/04_configurations.md)
