# Input Validation & Sanitization

**Parent:** [← Kembali ke Main](_main.md)  
**Status:** ✅ Selesai

---

## 🚨 Current State: NO VALIDATION

### Critical Finding

**Files Status:**

- `src/core/validation/schemas.ts` → **0 bytes (EMPTY)**
- `src/core/validation/validate.ts` → **0 bytes (EMPTY)**

**Impact:** Seluruh user input tidak ter-validasi sebelum digunakan!

---

## 🎯 Attack Vectors Saat Ini

### 1. Project Name Injection

#### Path Traversal

```bash
# Attack 1: Path traversal
$ orbit create "../../etc/passwd"
# Creates project at: /etc/passwd (if permissions allow)

# Attack 2: Hidden directory
$ orbit create ".hidden-malware"
# Creates hidden dir that might bypass security scans

# Attack 3: Windows reserved names
$ orbit create "CON"  # Windows device name
$ orbit create "PRN"  # Windows device name
```

#### Command Flag Injection

```bash
# Attack: Inject additional flags
$ orbit create "myapp --template malicious"

# Spawned as:
npx create-next-app@latest myapp --template malicious --typescript --eslint
#                                 ^^^^^^^^^^^^^^^^^^^ Unintended flag!
```

#### Special Character Exploitation

````bash
# Attack: Newline injection (jika tidak handle dengan benar)
$ orbit create "app\n\nmalicious command"

# Attack: Unicode confusion
$ orbit create "app\u202E.js"  # Right-to-left override  ```

---

### 2. Framework Selection - LESS CRITICAL

**Current:** Framework dipilih dari menu (tidak direct input)

**Tetapi:** Jika ada refactor untuk accept CLI arg:
```bash
# Possible attack jika refactor tanpa validation
$ orbit create myapp --framework "nextjs; rm -rf /"
````

**Risk:** 🟢 LOW (currently menu-based)

---

### 3. Package Manager Selection - LESS CRITICAL

**Current:** PM dipilih dari menu

**Risk:** 🟢 LOW (currently menu-based)

---

## ✅ Required Validation Schemas (Zod)

### Implementation Plan

**File:** `src/core/validation/schemas.ts`

```typescript
import { z } from "zod";

/**
 * Project Name Validation
 *
 * RULES:
 * - Length: 1-100 characters
 * - Characters: alphanumeric, dash, underscore only
 * - Cannot start with dash (避免 flag injection)
 * - Cannot contain ".." (避免 path traversal)
 * - Cannot be Windows reserved names
 */
const WINDOWS_RESERVED_NAMES = [
  "CON",
  "PRN",
  "AUX",
  "NUL",
  "COM1",
  "COM2",
  "COM3",
  "COM4",
  "COM5",
  "COM6",
  "COM7",
  "COM8",
  "COM9",
  "LPT1",
  "LPT2",
  "LPT3",
  "LPT4",
  "LPT5",
  "LPT6",
  "LPT7",
  "LPT8",
  "LPT9",
] as const;

export const ProjectNameSchema = z
  .string({
    required_error: "Project name is required",
    invalid_type_error: "Project name must be a string",
  })
  .trim()
  .min(1, "Project name cannot be empty")
  .max(100, "Project name must be 100 characters or less")
  // Only alphanumeric, dash, underscore
  .regex(
    /^[a-z0-9-_]+$/i,
    "Project name can only contain letters, numbers, dashes, and underscores"
  )
  // Cannot start with dash (prevents flag injection)
  .refine(
    (val) => !val.startsWith("-"),
    "Project name cannot start with a dash"
  )
  // Cannot contain .. (prevents path traversal)
  .refine((val) => !val.includes(".."), 'Project name cannot contain ".."')
  // Cannot be Windows reserved name
  .refine(
    (val) => !WINDOWS_RESERVED_NAMES.includes(val.toUpperCase()),
    "Project name cannot be a Windows reserved name (CON, PRN, AUX, etc.)"
  )
  // Cannot be only dots
  .refine((val) => !/^\.+$/.test(val), "Project name cannot be only dots");

/**
 * Framework ID Validation
 */
export const FrameworkIdSchema = z.enum(
  ["nextjs", "nuxt", "astro", "sveltekit", "vue", "remix", "laravel"],
  {
    errorMap: () => ({ message: "Invalid framework" }),
  }
);

/**
 * Package Manager Validation
 */
export const PackageManagerSchema = z.enum(["npm", "yarn", "pnpm", "bun"], {
  errorMap: () => ({ message: "Invalid package manager" }),
});

/**
 * Stack Preset Validation
 */
export const StackPresetSchema = z
  .enum(["minimal", "standard", "full"], {
    errorMap: () => ({ message: "Invalid stack preset" }),
  })
  .optional();

/**
 * Full Create Project Input Schema
 */
export const CreateProjectInputSchema = z.object({
  name: ProjectNameSchema,
  framework: FrameworkIdSchema,
  packageManager: PackageManagerSchema,
  stack: StackPresetSchema,
  skipPrompts: z.boolean().optional(),
});

export type CreateProjectInput = z.infer<typeof CreateProjectInputSchema>;
```

---

### Validation Helper Functions

**File:** `src/core/validation/validate.ts`

```typescript
import { type ZodSchema, ZodError } from "zod";
import { ValidationError } from "../errors/classes.js";

/**
 * Validate input using Zod schema
 *
 * @throws {ValidationError} if validation fails
 */
export function validate<T>(
  schema: ZodSchema<T>,
  data: unknown,
  fieldName: string = "Input"
): T {
  try {
    return schema.parse(data);
  } catch (error) {
    if (error instanceof ZodError) {
      const firstError = error.errors[0];
      throw new ValidationError(
        `${fieldName} validation failed: ${firstError.message}`
      );
    }
    throw error;
  }
}

/**
 * Safe validate - returns Result type instead of throwing
 */
export function safeValidate<T>(
  schema: ZodSchema<T>,
  data: unknown
): { success: true; data: T } | { success: false; error: string } {
  const result = schema.safeParse(data);

  if (result.success) {
    return { success: true, data: result.data };
  }

  const firstError = result.error.errors[0];
  return {
    success: false,
    error: firstError.message,
  };
}

/**
 * Sanitize path to prevent traversal
 */
export function sanitizePath(input: string): string {
  // Remove any ../ or ..\\ sequences
  let sanitized = input.replace(/\.\.[/\\]/g, "");

  // Remove leading slashes/backslashes
  sanitized = sanitized.replace(/^[/\\]+/, "");

  // Normalize to forward slashes
  sanitized = sanitized.replace(/\\/g, "/");

  return sanitized;
}

/**
 * Check if path is within allowed directory
 */
export function isPathSafe(basePath: string, targetPath: string): boolean {
  const path = require("node:path");
  const resolvedBase = path.resolve(basePath);
  const resolvedTarget = path.resolve(basePath, targetPath);

  // Target must be within base directory
  return resolvedTarget.startsWith(resolvedBase);
}
```

---

## 🔧 Integration Points

### 1. Command Handler (`commands/create.ts`)

```typescript
// BEFORE (vulnerable)
export async function runCreate(name?: string, options?: Options) {
  // name langsung digunakan tanpa validasi
}

// AFTER (secure)
import { validate } from "../core/validation/validate.js";
import { ProjectNameSchema } from "../core/validation/schemas.js";

export async function runCreate(name?: string, options?: Options) {
  // Validate name if provided
  if (name) {
    name = validate(ProjectNameSchema, name, "Project name");
  }

  // ... rest of logic
}
```

---

### 2. Flow Handler (`flows/create-flow.ts`)

```typescript
// Add validation before prompt atau after prompt

// Option A: Validate after prompt
const input = await getProjectInput(name, options);

// Validate all inputs
const validatedInput = validate(
  CreateProjectInputSchema,
  input,
  "Project configuration"
);

// Use validatedInput instead of input
```

---

### 3. Use Case (`core/usecases/create-project.ts`)

```typescript
import { validate } from "../validation/validate.js";
import { CreateProjectInputSchema } from "../validation/schemas.js";

class CreateProject {
  async execute(input: ProjectInput, reporter: Reporter): Promise<Result> {
    // Validate at use case level (defense in depth)
    const validatedInput = validate(
      CreateProjectInputSchema,
      input,
      "Project input"
    );

    // ... rest of use case logic
  }
}
```

---

## 📊 Test Cases Required

### Unit Tests for Validation Schemas

**File:** `src/core/validation/__tests__/schemas.test.ts`

```typescript
import { describe, it, expect } from "vitest";
import { ProjectNameSchema } from "../schemas.js";

describe("ProjectNameSchema", () => {
  describe("valid names", () => {
    it("accepts lowercase alphanumeric", () => {
      expect(ProjectNameSchema.parse("myapp")).toBe("myapp");
    });

    it("accepts uppercase alphanumeric", () => {
      expect(ProjectNameSchema.parse("MyApp")).toBe("MyApp");
    });

    it("accepts dash and underscore", () => {
      expect(ProjectNameSchema.parse("my-awesome_app")).toBe("my-awesome_app");
    });

    it("trims whitespace", () => {
      expect(ProjectNameSchema.parse("  myapp  ")).toBe("myapp");
    });
  });

  describe("invalid names", () => {
    it("rejects empty string", () => {
      expect(() => ProjectNameSchema.parse("")).toThrow("cannot be empty");
    });

    it("rejects names starting with dash", () => {
      expect(() => ProjectNameSchema.parse("-myapp")).toThrow(
        "cannot start with a dash"
      );
    });

    it("rejects path traversal", () => {
      expect(() => ProjectNameSchema.parse("../etc/passwd")).toThrow(
        "cannot contain"
      );
    });

    it("rejects Windows reserved names", () => {
      expect(() => ProjectNameSchema.parse("CON")).toThrow(
        "Windows reserved name"
      );
      expect(() => ProjectNameSchema.parse("PRN")).toThrow(
        "Windows reserved name"
      );
    });

    it("rejects special characters", () => {
      expect(() => ProjectNameSchema.parse("my@app")).toThrow(
        "can only contain"
      );
      expect(() => ProjectNameSchema.parse("my app")).toThrow(
        "can only contain"
      );
      expect(() => ProjectNameSchema.parse("my/app")).toThrow(
        "can only contain"
      );
    });

    it("rejects names > 100 chars", () => {
      const longName = "a".repeat(101);
      expect(() => ProjectNameSchema.parse(longName)).toThrow(
        "100 characters or less"
      );
    });

    it("rejects only dots", () => {
      expect(() => ProjectNameSchema.parse(".")).toThrow("only dots");
      expect(() => ProjectNameSchema.parse("..")).toThrow("only dots");
    });
  });
});
```

---

## 🎯 Compliance with STANDARDS.md

### Section 9.A - Input Validation

```typescript
// ✅ SESUAI STANDARDS.md:279-298
// "Validate all external input"
// "Sanitize before use"

// ❌ CURRENT STATE: Tidak ada validation
// ⚠️ HARUS IMPLEMENT SEGERA
```

### Section 14.A - Anti-Hallucination

```markdown
// ✅ Best practice dari Context7 Zod docs
// ✅ Real CVE research (2024)
// ✅ Berdasarkan Node.js security guidelines
```

---

## 🔗 Terkait

- [02_security_analysis.md](02_security_analysis.md) - Command injection vectors
- [05_type_safety.md](05_type_safety.md) - TypeScript runtime validation
- [08_findings_recommendations.md](08_findings_recommendations.md) - Implementation PLAN
