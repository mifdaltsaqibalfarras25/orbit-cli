# Tech Stack — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Runtime & Environment

| Tool           | Version                   | Purpose         | Notes                     |
| :------------- | :------------------------ | :-------------- | :------------------------ |
| **Node.js**    | `^18.20.0` atau `^20.0.0` | Runtime         | LTS versions, ESM support |
| **npm**        | `^10.0.0`                 | Package manager | Atau pnpm/bun             |
| **TypeScript** | `^5.7.0`                  | Type checking   | Strict mode               |

### Node.js Version Requirements

```json
{
  "engines": {
    "node": ">=18.20.0"
  }
}
```

> ⚠️ **Minimum Node 18.20.0** — Required untuk CVE-2024-27980 patch dan fitur ESM stabil

---

## 2. Production Dependencies

| Package           | Version   | Size   | Purpose               |
| :---------------- | :-------- | :----- | :-------------------- |
| `commander`       | `^13.0.0` | ~34KB  | CLI command parsing   |
| `@clack/prompts`  | `^0.9.0`  | ~50KB  | Interactive prompts   |
| `chalk`           | `^5.3.0`  | ~31KB  | Terminal colors (ESM) |
| `ora`             | `^8.1.0`  | ~12KB  | Loading spinners      |
| `gradient-string` | `^3.0.0`  | ~8KB   | Gradient text         |
| `figlet`          | `^1.8.0`  | ~700KB | ASCII art banner      |
| `zod`             | `^3.24.0` | ~60KB  | Input validation      |

### Install Command

```bash
npm install commander@^13.0.0 @clack/prompts@^0.9.0 chalk@^5.3.0 ora@^8.1.0 gradient-string@^3.0.0 figlet@^1.8.0 zod@^3.24.0
```

### Dependency Notes

| Package           | Note                                             |
| :---------------- | :----------------------------------------------- |
| `chalk`           | v5 is **ESM-only** — CJS projects harus pakai v4 |
| `ora`             | v8 is **ESM-only**                               |
| `gradient-string` | v3 is **ESM-only**, requires Node 14+            |
| `figlet`          | Large (~700KB) — **lazy load only**              |
| `@clack/prompts`  | Modern alternative ke inquirer                   |

---

## 3. Development Dependencies

| Package                            | Version   | Purpose                  |
| :--------------------------------- | :-------- | :----------------------- |
| `typescript`                       | `^5.7.0`  | TypeScript compiler      |
| `tsup`                             | `^8.3.0`  | Bundler (esbuild-based)  |
| `@types/node`                      | `^22.0.0` | Node.js type definitions |
| `@types/figlet`                    | `^1.5.8`  | Figlet type definitions  |
| `vitest`                           | `^2.1.0`  | Unit testing             |
| `eslint`                           | `^9.16.0` | Linting (flat config)    |
| `prettier`                         | `^3.4.0`  | Code formatting          |
| `@typescript-eslint/parser`        | `^8.18.0` | TypeScript ESLint        |
| `@typescript-eslint/eslint-plugin` | `^8.18.0` | TypeScript rules         |

### Install Command

```bash
npm install -D typescript@^5.7.0 tsup@^8.3.0 @types/node@^22.0.0 @types/figlet@^1.5.8 vitest@^2.1.0 eslint@^9.16.0 prettier@^3.4.0 @typescript-eslint/parser@^8.18.0 @typescript-eslint/eslint-plugin@^8.18.0
```

---

## 4. Optional Dependencies

| Package       | Version   | When to Use                           |
| :------------ | :-------- | :------------------------------------ |
| `husky`       | `^9.1.0`  | Git hooks (pre-commit)                |
| `lint-staged` | `^15.2.0` | Lint staged files only                |
| `cross-spawn` | `^7.0.5`  | Cross-platform spawn (Windows compat) |
| `execa`       | `^9.5.0`  | Better child_process wrapper          |

> ⚠️ **cross-spawn** harus `^7.0.5` untuk fix CVE-2024-21538 (ReDoS)

---

## 5. Complete package.json Dependencies

```json
{
  "name": "orbit-cli",
  "version": "1.0.0",
  "description": "Universal Project Generator CLI",
  "type": "module",
  "engines": {
    "node": ">=18.20.0"
  },
  "bin": {
    "orbit": "./dist/index.js"
  },
  "files": ["dist"],
  "scripts": {
    "dev": "tsup --watch",
    "build": "tsup",
    "test": "vitest",
    "test:run": "vitest run",
    "lint": "eslint src/",
    "format": "prettier --write src/",
    "typecheck": "tsc --noEmit",
    "prepublishOnly": "npm run build"
  },
  "dependencies": {
    "commander": "^13.0.0",
    "@clack/prompts": "^0.9.0",
    "chalk": "^5.3.0",
    "ora": "^8.1.0",
    "gradient-string": "^3.0.0",
    "figlet": "^1.8.0",
    "zod": "^3.24.0"
  },
  "devDependencies": {
    "typescript": "^5.7.0",
    "tsup": "^8.3.0",
    "@types/node": "^22.0.0",
    "@types/figlet": "^1.5.8",
    "vitest": "^2.1.0",
    "eslint": "^9.16.0",
    "prettier": "^3.4.0",
    "@typescript-eslint/parser": "^8.18.0",
    "@typescript-eslint/eslint-plugin": "^8.18.0"
  },
  "keywords": [
    "cli",
    "project-generator",
    "scaffold",
    "nextjs",
    "nuxt",
    "astro"
  ],
  "author": "",
  "license": "MIT"
}
```

---

## 6. Version Verification Commands

```bash
# Check installed versions
npm list commander chalk ora @clack/prompts zod

# Check for updates
npm outdated

# Check for vulnerabilities
npm audit
```

---

## 7. Compatibility Matrix

| Node Version    | Status             | Notes                      |
| :-------------- | :----------------- | :------------------------- |
| v22.x           | ✅ Recommended     | Latest features            |
| v20.x           | ✅ Supported       | LTS, production ready      |
| v18.20.0+       | ✅ Supported       | Minimum (security patches) |
| v18.0.0-18.19.x | ⚠️ Not Recommended | CVE-2024-27980 vulnerable  |
| v16.x           | ❌ Unsupported     | EOL                        |

---

## 8. Size Budget

| Category              | Budget  | Actual (Est.) |
| :-------------------- | :------ | :------------ |
| **Bundle (minified)** | < 500KB | ~400KB        |
| **node_modules**      | < 50MB  | ~35MB         |
| **Install time**      | < 30s   | ~15s          |
| **Cold start**        | < 200ms | ~150ms        |
