# Secure Development — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Secure Coding Practices

### 1.1 Input Validation Pattern

```typescript
// src/core/validation/schemas.ts

import { z } from "zod";

// Project name - npm package name rules
export const ProjectNameSchema = z
  .string()
  .min(1, "Project name is required")
  .max(214, "Project name must be ≤ 214 characters")
  .regex(
    /^(?:@[a-z0-9-*~][a-z0-9-*._~]*\/)?[a-z0-9-~][a-z0-9-._~]*$/,
    "Invalid npm package name"
  )
  .refine(
    (name) => !name.startsWith(".") && !name.startsWith("_"),
    "Cannot start with . or _"
  );

// Safe path - no traversal
export const SafePathSchema = z
  .string()
  .refine((p) => !p.includes(".."), 'Path cannot contain ".."')
  .refine((p) => !p.includes("\0"), "Path cannot contain null bytes")
  .refine((p) => !/[<>:"|?*]/.test(p), "Path contains invalid characters");

// Framework ID - whitelist only
export const FrameworkIdSchema = z.enum([
  "nextjs",
  "nuxt",
  "astro",
  "sveltekit",
  "vue",
  "remix",
  "laravel",
]);
```

### 1.2 Validation Before Processing

```typescript
// ✅ CORRECT - Validate then process
import { validate } from "./validation/validate.js";
import { ProjectNameSchema } from "./validation/schemas.js";

function createProject(userInput: unknown) {
  // FIRST: Validate
  const name = validate(ProjectNameSchema, userInput.name);

  // THEN: Use validated data
  await install(name);
}

// ❌ WRONG - Process without validation
function createProject(name: string) {
  await install(name); // Direct use of unvalidated input!
}
```

---

## 2. Safe Command Execution

### 2.1 spawn Pattern (WAJIB)

```typescript
// src/utils/safe-executor.ts

import { spawn, type SpawnOptions } from "child_process";
import { ALLOWED_COMMANDS } from "./whitelist.js";
import { CommandExecutionError } from "../core/errors.js";

/**
 * Execute command securely
 * - shell: false (ENFORCED)
 * - Only whitelisted commands
 * - Sanitized environment
 */
export function safeExecute(
  command: string,
  args: readonly string[],
  options?: Partial<SpawnOptions>
): Promise<ExecResult> {
  // 1. Validate command is whitelisted
  const basename = getBasename(command);
  if (!ALLOWED_COMMANDS.has(basename)) {
    throw new CommandExecutionError(`Command not allowed: ${command}`, command);
  }

  // 2. Execute with shell: false
  return new Promise((resolve, reject) => {
    const child = spawn(command, [...args], {
      shell: false, // SECURITY: Never override!
      env: createSafeEnv(),
      ...options,
    });

    let stdout = "";
    let stderr = "";

    child.stdout?.on("data", (d) => {
      stdout += d;
    });
    child.stderr?.on("data", (d) => {
      stderr += d;
    });

    child.on("close", (code) => {
      resolve({ code: code ?? 1, stdout, stderr });
    });

    child.on("error", (err) => {
      reject(new CommandExecutionError(err.message, command, stderr));
    });
  });
}

function getBasename(cmd: string): string {
  const base = cmd.split("/").pop() || cmd;
  return base.replace(/\.(exe|cmd|bat)$/i, "").toLowerCase();
}
```

### 2.2 Command Whitelist

```typescript
// src/utils/whitelist.ts

export const ALLOWED_COMMANDS = new Set([
  // Package managers
  "npm",
  "npx",
  "yarn",
  "pnpm",
  "bun",

  // Version control
  "git",

  // Runtimes (version check only)
  "node",
  "php",
  "composer",

  // Detection
  "which",
  "where",
]);
```

---

## 3. Path Security

### 3.1 Safe Path Resolution

```typescript
// src/utils/safe-path.ts

import * as path from "path";
import { FileSystemError } from "../core/errors.js";

/**
 * Resolve path safely within base directory
 * Prevents escape via '../' traversal
 */
export function resolveSafe(basePath: string, userPath: string): string {
  // 1. Check for obvious attacks
  if (userPath.includes("..")) {
    throw new FileSystemError('Path cannot contain ".."');
  }

  if (userPath.includes("\0")) {
    throw new FileSystemError("Path cannot contain null bytes");
  }

  // 2. Resolve and normalize
  const resolved = path.resolve(basePath, userPath);
  const normalized = path.normalize(resolved);

  // 3. Verify still within base
  const normalizedBase = path.normalize(basePath);
  if (!normalized.startsWith(normalizedBase)) {
    throw new FileSystemError("Path traversal detected");
  }

  return normalized;
}
```

### 3.2 Sandboxed Operations

```typescript
// All file operations sandboxed to cwd
const projectPath = resolveSafe(process.cwd(), input.name);
await fs.mkdir(projectPath); // Safe - within cwd
```

---

## 4. Environment Variable Security

### 4.1 Sanitized Environment

```typescript
// src/utils/safe-env.ts

const SENSITIVE_PATTERNS = [
  /^(API|AUTH|SECRET|PRIVATE).*KEY$/i,
  /^.*TOKEN$/i,
  /^.*SECRET$/i,
  /^AWS_/i,
  /^AZURE_/i,
  /^GCP_/i,
  /^.*DATABASE.*URL$/i,
  /^.*PASSWORD$/i,
  /^NPM_TOKEN$/i,
  /^GITHUB_TOKEN$/i,
];

export function createSafeEnv(): NodeJS.ProcessEnv {
  const env = { ...process.env };

  for (const key of Object.keys(env)) {
    if (SENSITIVE_PATTERNS.some((p) => p.test(key))) {
      delete env[key];
    }
  }

  return env;
}
```

---

## 5. Code Review Checklist

### Pre-Commit Security Review

- [ ] No `exec()` calls (use `spawn()`)
- [ ] All input validated with Zod schemas
- [ ] Paths checked for traversal
- [ ] No hardcoded secrets
- [ ] Environment sanitized before spawn
- [ ] Error messages don't leak sensitive info
- [ ] No `eval()` or `new Function()`
- [ ] Dependencies minimized

### ESLint Security Rules

```javascript
// eslint.config.js additions
{
  rules: {
    // Disallow risky functions
    'no-eval': 'error',
    'no-implied-eval': 'error',
    'no-new-func': 'error',

    // TypeScript strict
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/strict-boolean-expressions': 'warn',
  },
}
```

---

## 6. Secure Error Handling

### 6.1 Error Response Pattern

```typescript
// ✅ SECURE - Generic public message
catch (error) {
  console.error(c.fail('Failed to create project'));
  console.error(c.dim('Run with --verbose for details'));
  process.exit(1);
}

// ❌ INSECURE - Leaks internal info
catch (error) {
  console.error(error.stack); // Leaks file paths!
  console.error(`DB connection: ${connectionString}`); // Leaks secrets!
}
```

### 6.2 Custom Error Classes (from TOPIC_004)

```typescript
export abstract class OrbitError extends Error {
  abstract readonly code: string;
  abstract readonly exitCode: number;
  readonly isOperational = true; // Expected errors

  // Never include sensitive data in message
}
```
