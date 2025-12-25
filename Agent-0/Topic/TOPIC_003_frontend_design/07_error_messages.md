# Error Messages — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Error Message Design Principles

1. **Clear** — Jelaskan apa yang salah
2. **Actionable** — Berikan langkah perbaikan
3. **Consistent** — Format seragam
4. **Non-technical** — Hindari jargon untuk user-facing errors

---

## 2. Error Code System

### Error Code Format

```
ORBIT-{CATEGORY}{NUMBER}

Categories:
- V = Validation (input errors)
- E = Environment (missing tools)
- F = Filesystem (file/dir errors)
- N = Network (connection errors)
- C = Command (execution errors)
- I = Internal (unexpected errors)
```

### Error Codes Table

| Code       | Type        | Message                          |
| :--------- | :---------- | :------------------------------- |
| ORBIT-V001 | Validation  | Project name is required         |
| ORBIT-V002 | Validation  | Invalid project name format      |
| ORBIT-V003 | Validation  | Project name too long            |
| ORBIT-V004 | Validation  | Unknown framework                |
| ORBIT-V005 | Validation  | Unknown stack preset             |
| ORBIT-V006 | Validation  | Invalid package manager          |
| ORBIT-E001 | Environment | Node.js not found                |
| ORBIT-E002 | Environment | Node.js version too old          |
| ORBIT-E003 | Environment | npm not found                    |
| ORBIT-E004 | Environment | git not found                    |
| ORBIT-E005 | Environment | PHP not found (for Laravel)      |
| ORBIT-E006 | Environment | Composer not found (for Laravel) |
| ORBIT-F001 | Filesystem  | Directory already exists         |
| ORBIT-F002 | Filesystem  | Permission denied                |
| ORBIT-F003 | Filesystem  | Cannot write to directory        |
| ORBIT-F004 | Filesystem  | Path traversal detected          |
| ORBIT-C001 | Command     | Command not allowed              |
| ORBIT-C002 | Command     | Command failed                   |
| ORBIT-C003 | Command     | Command timeout                  |
| ORBIT-I001 | Internal    | Unknown error                    |

---

## 3. Error Message Templates

### 3.1 Validation Errors

```typescript
// src/core/errors/messages.ts

export const MESSAGES = {
  // Project Name
  V001: {
    code: "ORBIT-V001",
    title: "Project name is required",
    message: "Please provide a project name.",
    hint: "Usage: orbit create <project-name>",
  },

  V002: {
    code: "ORBIT-V002",
    title: "Invalid project name",
    message: (name: string) => `"${name}" is not a valid project name.`,
    hint: "Use lowercase letters, numbers, and dashes only.",
  },

  V003: {
    code: "ORBIT-V003",
    title: "Project name too long",
    message: (length: number) => `Name is ${length} characters (max 50).`,
    hint: "Choose a shorter name.",
  },

  // Framework
  V004: {
    code: "ORBIT-V004",
    title: "Unknown framework",
    message: (fw: string) => `Framework "${fw}" is not supported.`,
    hint: 'Run "orbit list" to see available frameworks.',
  },

  V005: {
    code: "ORBIT-V005",
    title: "Unknown stack preset",
    message: (stack: string) => `Stack "${stack}" is not available.`,
    hint: "Available: minimal, standard, full",
  },

  V006: {
    code: "ORBIT-V006",
    title: "Invalid package manager",
    message: (pm: string) => `"${pm}" is not a valid package manager.`,
    hint: "Available: npm, yarn, pnpm, bun",
  },
};
```

### 3.2 Environment Errors

```typescript
export const ENV_MESSAGES = {
  E001: {
    code: "ORBIT-E001",
    title: "Node.js not found",
    message: "Node.js is required but was not found on your system.",
    hint: "Install Node.js from https://nodejs.org",
  },

  E002: {
    code: "ORBIT-E002",
    title: "Node.js version too old",
    message: (current: string, required: string) =>
      `Found Node.js ${current}, but ${required} or higher is required.`,
    hint: "Update Node.js or use nvm: nvm install 20",
  },

  E003: {
    code: "ORBIT-E003",
    title: "npm not found",
    message: "npm is required but was not found.",
    hint: "npm usually comes with Node.js. Reinstall Node.js.",
  },

  E004: {
    code: "ORBIT-E004",
    title: "git not found",
    message: "git is required for version control.",
    hint: "Install git from https://git-scm.com",
  },

  E005: {
    code: "ORBIT-E005",
    title: "PHP not found",
    message: "PHP is required for Laravel projects.",
    hint: "Install PHP 8.1+ from https://php.net",
  },

  E006: {
    code: "ORBIT-E006",
    title: "Composer not found",
    message: "Composer is required for Laravel projects.",
    hint: "Install Composer from https://getcomposer.org",
  },
};
```

### 3.3 Filesystem Errors

```typescript
export const FS_MESSAGES = {
  F001: {
    code: "ORBIT-F001",
    title: "Directory already exists",
    message: (path: string) => `Directory "${path}" already exists.`,
    hint: "Choose a different name or delete the existing directory.",
  },

  F002: {
    code: "ORBIT-F002",
    title: "Permission denied",
    message: (path: string) => `Cannot access "${path}".`,
    hint: "Check file permissions or try a different location.",
  },

  F003: {
    code: "ORBIT-F003",
    title: "Cannot write to directory",
    message: (path: string) => `Cannot create files in "${path}".`,
    hint: "Check write permissions or try a different location.",
  },

  F004: {
    code: "ORBIT-F004",
    title: "Invalid path",
    message: "Path traversal is not allowed.",
    hint: 'Use a simple directory name without ".." or absolute paths.',
  },
};
```

### 3.4 Command Errors

```typescript
export const CMD_MESSAGES = {
  C001: {
    code: "ORBIT-C001",
    title: "Command not allowed",
    message: (cmd: string) => `Command "${cmd}" is not permitted.`,
    hint: "This is a security restriction.",
  },

  C002: {
    code: "ORBIT-C002",
    title: "Command failed",
    message: (cmd: string, code: number) =>
      `"${cmd}" exited with code ${code}.`,
    hint: "Check the output above for details.",
  },

  C003: {
    code: "ORBIT-C003",
    title: "Command timeout",
    message: "The operation took too long and was cancelled.",
    hint: "Check your network connection and try again.",
  },
};
```

---

## 4. Error Display Functions

```typescript
// src/ui/error-display.ts

import * as p from "@clack/prompts";
import { colors, c } from "./colors.js";
import type { OrbitError } from "../core/errors.js";

/**
 * Display user-friendly error message
 */
export function displayError(error: OrbitError): void {
  console.log();
  console.log(c.fail(error.title || "Error"));

  if (error.message) {
    console.log(colors.dim(error.message));
  }

  if (error.hint) {
    console.log();
    console.log(`${colors.info("💡 Hint:")} ${error.hint}`);
  }

  if (process.env.DEBUG) {
    console.log();
    console.log(colors.dim(`Code: ${error.code}`));
    if (error.stack) {
      console.log(colors.dim(error.stack));
    }
  }

  console.log();
}

/**
 * Display validation error inline
 */
export function displayValidationError(message: string): string {
  return colors.error(message);
}

/**
 * Cancel operation display
 */
export function displayCancel(message = "Operation cancelled."): void {
  p.cancel(message);
}

/**
 * Display warning (non-fatal)
 */
export function displayWarning(message: string, hint?: string): void {
  console.log(c.warn(message));
  if (hint) {
    console.log(colors.dim(`  ${hint}`));
  }
}
```

---

## 5. Error Handling Pattern

```typescript
// src/commands/create.ts

import { displayError, displayCancel } from "../ui/error-display.js";
import { ValidationError, EnvironmentError } from "../core/errors.js";

export async function runCreate(name: string, options: Options): Promise<void> {
  try {
    // ... main logic
  } catch (error) {
    if (error instanceof ValidationError) {
      displayError({
        code: error.code,
        title: error.title,
        message: error.message,
        hint: error.hint,
      });
      process.exit(1);
    }

    if (error instanceof EnvironmentError) {
      displayError({
        code: error.code,
        title: error.title,
        message: error.message,
        hint: error.hint,
      });
      process.exit(2);
    }

    // Unknown error
    displayError({
      code: "ORBIT-I001",
      title: "Unexpected error",
      message: error instanceof Error ? error.message : String(error),
      hint: "Please report this issue at https://github.com/username/orbit-cli/issues",
    });

    if (process.env.DEBUG) {
      console.error(error);
    }

    process.exit(1);
  }
}
```

---

## 6. Example Error Outputs

### Validation Error

```
✗ Invalid project name

  "my project" is not a valid project name.

💡 Hint: Use lowercase letters, numbers, and dashes only.
```

### Environment Error

```
✗ Node.js version too old

  Found Node.js v16.14.0, but v18.20.0 or higher is required.

💡 Hint: Update Node.js or use nvm: nvm install 20
```

### Filesystem Error

```
✗ Directory already exists

  Directory "my-app" already exists.

💡 Hint: Choose a different name or delete the existing directory.
```

### Command Error (with DEBUG)

```
✗ Command failed

  "npm create next-app" exited with code 1.

💡 Hint: Check the output above for details.

Code: ORBIT-C002
```

---

## 7. Localization Ready

```typescript
// Future: i18n support
const messages = {
  en: {
    "ORBIT-V001": "Project name is required",
    "ORBIT-V002": "Invalid project name",
  },
  id: {
    "ORBIT-V001": "Nama proyek wajib diisi",
    "ORBIT-V002": "Nama proyek tidak valid",
  },
};
```
