# Installation Steps — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## Step 1: Create Project Directory

```bash
# Create project folder
mkdir orbit-cli
cd orbit-cli
```

---

## Step 2: Initialize npm Project

```bash
# Initialize with default values
npm init -y
```

---

## Step 3: Configure package.json

Edit `package.json` untuk set ESM dan metadata:

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
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
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

## Step 4: Install Production Dependencies

```bash
npm install commander@^13.0.0 @clack/prompts@^0.9.0 chalk@^5.3.0 ora@^8.1.0 gradient-string@^3.0.0 figlet@^1.8.0 zod@^3.24.0
```

Expected output:

```
added 7 packages in 5s
```

---

## Step 5: Install Development Dependencies

```bash
npm install -D typescript@^5.7.0 tsup@^8.3.0 @types/node@^22.0.0 @types/figlet@^1.5.8 vitest@^2.1.0 eslint@^9.16.0 prettier@^3.4.0 @typescript-eslint/parser@^8.18.0 @typescript-eslint/eslint-plugin@^8.18.0
```

Expected output:

```
added 150 packages in 15s
```

---

## Step 6: Create tsconfig.json

```bash
# Create TypeScript config
cat > tsconfig.json << 'EOF'
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022"],
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noUncheckedIndexedAccess": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
EOF
```

---

## Step 7: Create tsup.config.ts

```bash
cat > tsup.config.ts << 'EOF'
import { defineConfig } from 'tsup';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['esm'],
  target: 'node18',
  outDir: 'dist',
  clean: true,
  dts: true,
  sourcemap: true,
  splitting: true,
  treeshake: true,
  minify: false, // Set true for production
  banner: {
    js: '#!/usr/bin/env node',
  },
});
EOF
```

---

## Step 8: Create Folder Structure

```bash
# Create source directories
mkdir -p src/{commands,core/{domain,usecases,services,validation},ui,frameworks,utils,__tests__/{unit,integration}}

# Create empty index files
touch src/index.ts
touch src/cli.ts
touch src/core/index.ts
touch src/core/errors.ts
touch src/core/container.ts
touch src/ui/index.ts
touch src/frameworks/index.ts
touch src/utils/index.ts
```

---

## Step 9: Create ESLint Config

```bash
cat > eslint.config.js << 'EOF'
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        project: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'warn',
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/prefer-readonly': 'error',
    },
  },
  {
    ignores: ['dist/', 'node_modules/', '*.config.*'],
  }
);
EOF
```

---

## Step 10: Create Prettier Config

```bash
cat > .prettierrc << 'EOF'
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100,
  "arrowParens": "always"
}
EOF
```

---

## Step 11: Create .gitignore

```bash
cat > .gitignore << 'EOF'
# Dependencies
node_modules/

# Build output
dist/

# IDE
.idea/
.vscode/
*.swp
*.swo

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

# Lock files (keep one)
# yarn.lock
# pnpm-lock.yaml
EOF
```

---

## Step 12: Initialize Git

```bash
git init
git add .
git commit -m "chore: initial project setup"
```

---

## Step 13: Verify Setup

```bash
# Check TypeScript compilation
npm run typecheck

# Try build
npm run build

# Verify output
ls -la dist/
```

Expected output:

```
dist/
├── index.js      # Main entry point
├── index.js.map  # Source map
└── index.d.ts    # Type declarations
```

---

## Quick Start (All-in-One)

Untuk AI implementor, jalankan semua dalam satu script:

```bash
#!/bin/bash
set -e

# Step 1-2: Create and init
mkdir -p orbit-cli && cd orbit-cli
npm init -y

# Step 4-5: Install deps
npm install commander@^13.0.0 @clack/prompts@^0.9.0 chalk@^5.3.0 ora@^8.1.0 gradient-string@^3.0.0 figlet@^1.8.0 zod@^3.24.0
npm install -D typescript@^5.7.0 tsup@^8.3.0 @types/node@^22.0.0 @types/figlet@^1.5.8 vitest@^2.1.0 eslint@^9.16.0 prettier@^3.4.0 @typescript-eslint/parser@^8.18.0 @typescript-eslint/eslint-plugin@^8.18.0

# Step 8: Create structure
mkdir -p src/{commands,core/{domain,usecases,services,validation},ui,frameworks,utils,__tests__/{unit,integration}}

echo "✅ ORBIT CLI project initialized!"
```

---

## Troubleshooting

### Error: Cannot find module

```bash
# Pastikan type: module di package.json
# Pastikan import menggunakan .js extension
```

### Error: ERR_REQUIRE_ESM

```bash
# Chalk v5, Ora v8 adalah ESM-only
# Pastikan package.json memiliki "type": "module"
```

### Error: Node version too old

```bash
# Check version
node --version

# Update Node.js ke v18.20.0 atau lebih tinggi
nvm install 20
nvm use 20
```
