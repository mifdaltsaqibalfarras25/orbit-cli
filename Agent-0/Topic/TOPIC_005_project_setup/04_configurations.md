# Configurations — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. tsconfig.json

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    // Target
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022"],

    // Output
    "outDir": "dist",
    "rootDir": "src",
    "declaration": true,
    "declarationMap": true,

    // Strict Mode (WAJIB)
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noUncheckedIndexedAccess": true,
    "forceConsistentCasingInFileNames": true,

    // Interop
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "isolatedModules": true,

    // Path aliases (optional)
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

---

## 2. tsup.config.ts

```typescript
import { defineConfig } from "tsup";

export default defineConfig({
  // Entry point
  entry: ["src/index.ts"],

  // Output format (ESM only)
  format: ["esm"],

  // Target Node.js version
  target: "node18",

  // Output directory
  outDir: "dist",

  // Clean output before build
  clean: true,

  // Generate TypeScript declarations
  dts: true,

  // Generate source maps
  sourcemap: true,

  // Code splitting for lazy loading
  splitting: true,

  // Tree shaking
  treeshake: true,

  // Minification (enable for production)
  minify: false,

  // Add shebang for CLI
  banner: {
    js: "#!/usr/bin/env node",
  },

  // External packages (don't bundle)
  external: [
    // Heavy packages - lazy load instead
    "figlet",
  ],

  // Environment
  define: {
    "process.env.NODE_ENV": JSON.stringify("production"),
  },
});
```

---

## 3. eslint.config.js

```javascript
import eslint from "@eslint/js";
import tseslint from "typescript-eslint";

export default tseslint.config(
  // Base ESLint recommended
  eslint.configs.recommended,

  // TypeScript strict rules
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,

  // Custom rules
  {
    languageOptions: {
      parserOptions: {
        project: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      // TypeScript
      "@typescript-eslint/no-unused-vars": [
        "error",
        {
          argsIgnorePattern: "^_",
          varsIgnorePattern: "^_",
        },
      ],
      "@typescript-eslint/explicit-function-return-type": [
        "warn",
        {
          allowExpressions: true,
        },
      ],
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/prefer-readonly": "error",
      "@typescript-eslint/no-floating-promises": "error",
      "@typescript-eslint/await-thenable": "error",
      "@typescript-eslint/no-misused-promises": "error",

      // General
      "no-console": ["warn", { allow: ["warn", "error"] }],
      "prefer-const": "error",
      "no-var": "error",
    },
  },

  // Ignore patterns
  {
    ignores: ["dist/", "node_modules/", "*.config.*", "coverage/"],
  }
);
```

---

## 4. .prettierrc

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

---

## 5. .prettierignore

```
dist/
node_modules/
coverage/
*.md
```

---

## 6. vitest.config.ts

```typescript
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    // Test environment
    environment: "node",

    // Glob patterns
    include: ["src/**/*.test.ts"],
    exclude: ["node_modules", "dist"],

    // Coverage
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      include: ["src/**/*.ts"],
      exclude: ["src/**/*.test.ts", "src/index.ts"],
    },

    // Globals
    globals: true,

    // Timeout
    testTimeout: 10000,
  },

  // Path aliases (match tsconfig)
  resolve: {
    alias: {
      "@": "./src",
    },
  },
});
```

---

## 7. .gitignore

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
yarn-debug.log*
yarn-error.log*

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

---

## 8. .editorconfig

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

---

## 9. .nvmrc

```
20
```

---

## 10. Complete package.json

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
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "files": ["dist"],
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "scripts": {
    "dev": "tsup --watch",
    "build": "tsup",
    "build:prod": "tsup --minify",
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix",
    "format": "prettier --write src/",
    "format:check": "prettier --check src/",
    "typecheck": "tsc --noEmit",
    "prepublishOnly": "npm run build:prod",
    "prepare": "npm run build"
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
    "@vitest/coverage-v8": "^2.1.0",
    "eslint": "^9.16.0",
    "prettier": "^3.4.0",
    "typescript-eslint": "^8.18.0"
  },
  "keywords": [
    "cli",
    "project-generator",
    "scaffold",
    "boilerplate",
    "nextjs",
    "nuxt",
    "astro",
    "sveltekit"
  ],
  "author": "",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/your-username/orbit-cli"
  },
  "bugs": {
    "url": "https://github.com/your-username/orbit-cli/issues"
  },
  "homepage": "https://github.com/your-username/orbit-cli#readme"
}
```

---

## 11. Configuration Checklist

- [x] `tsconfig.json` — Strict TypeScript config
- [x] `tsup.config.ts` — ESM bundle config
- [x] `eslint.config.js` — Flat config ESLint 9
- [x] `.prettierrc` — Code formatting
- [x] `vitest.config.ts` — Testing config
- [x] `.gitignore` — Git ignore rules
- [x] `.editorconfig` — Editor settings
- [x] `.nvmrc` — Node version
- [x] `package.json` — Complete manifest
