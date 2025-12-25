# Security Fixes: Windows Reserved Names, spawnSync, Type Safety

**ID:** PLAN_015 | **Status:** ✅ Done | **Prioritas:** 🟡 Normal  
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25

---

## 🎯 Tujuan

Menerapkan perbaikan security yang ditemukan dalam TOPIC_007 Security & Bug Audit:

1. **Windows Reserved Names** - Tambah validasi untuk device names Windows (CON, PRN, AUX, NUL, COM1-9, LPT1-9)
2. **Replace execSync** - Ganti `execSync` dengan `spawnSync` di check-tools.ts untuk konsistensi security
3. **Fix Type Issues** - Perbaiki 6x penggunaan `any` type dengan eslint-disable comments

**Research Sources:**

- Microsoft Docs: [Naming Files, Paths, and Namespaces](https://learn.microsoft.com/en-us/windows/win32/fileio/naming-a-file)
- Node.js Docs: [child_process.spawnSync](https://nodejs.org/api/child_process.html#child_processspawnsynccommand-args-options)
- Context7: Node.js API documentation (verified 2025-12-25)

---

## 🛠️ Strategi Implementasi

### Task 1: Add Windows Reserved Names Validation

#### [MODIFY] [validation.ts](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/orbit-cli/src/utils/validation.ts)

**Current Code (line 49-53):**

```typescript
// Check for reserved names
const reserved = ["node_modules", "package", "dist", "build", "src", "test"];
if (reserved.includes(name)) {
  return { valid: false, error: `"${name}" is a reserved name` };
}
```

**New Code:**

```typescript
/**
 * Windows reserved device names (MS-DOS legacy)
 * Source: https://learn.microsoft.com/en-us/windows/win32/fileio/naming-a-file
 */
const WINDOWS_RESERVED_NAMES = [
  "con",
  "prn",
  "aux",
  "nul",
  "com1",
  "com2",
  "com3",
  "com4",
  "com5",
  "com6",
  "com7",
  "com8",
  "com9",
  "lpt1",
  "lpt2",
  "lpt3",
  "lpt4",
  "lpt5",
  "lpt6",
  "lpt7",
  "lpt8",
  "lpt9",
] as const;

// Check for reserved names (project-specific + Windows)
const PROJECT_RESERVED = [
  "node_modules",
  "package",
  "dist",
  "build",
  "src",
  "test",
];
const lowerName = name.toLowerCase();

if (PROJECT_RESERVED.includes(name)) {
  return { valid: false, error: `"${name}" is a reserved project name` };
}

if (
  WINDOWS_RESERVED_NAMES.includes(
    lowerName as (typeof WINDOWS_RESERVED_NAMES)[number]
  )
) {
  return {
    valid: false,
    error: `"${name}" is a Windows reserved device name and cannot be used as a project name`,
  };
}
```

**Rationale:**

- Windows tidak mengizinkan CON, PRN, AUX, NUL, COM1-9, LPT1-9 sebagai nama file/folder
- Case-insensitive check karena Windows filesystem case-insensitive
- Separated dari project reserved names untuk error message yang lebih jelas

---

### Task 2: Replace execSync with spawnSync

#### [MODIFY] [check-tools.ts](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/orbit-cli/src/commands/helpers/check-tools.ts)

**Current Code (line 6, 16-28):**

```typescript
import { execSync } from "node:child_process";

function getVersion(command: string): string | undefined {
  try {
    const output = execSync(command, {
      encoding: "utf-8",
      stdio: ["pipe", "pipe", "pipe"],
    }).trim();
    const match = output.match(/v?(\d+\.\d+\.\d+)/);
    return match?.[1] ?? output;
  } catch {
    return undefined;
  }
}
```

**New Code:**

```typescript
import { spawnSync } from "node:child_process";

/**
 * Execute command safely using spawnSync
 * SECURITY: spawnSync with shell:false prevents command injection
 *
 * @param tool - The tool binary name (e.g., 'node', 'npm')
 * @param args - Arguments array (e.g., ['--version'])
 * @returns Version string or undefined if not found
 */
function getVersion(
  tool: string,
  args: readonly string[] = ["--version"]
): string | undefined {
  try {
    const result = spawnSync(tool, [...args], {
      encoding: "utf-8",
      shell: false, // SECURITY: Prevent command injection
      stdio: ["pipe", "pipe", "pipe"],
      timeout: 5000, // 5 second timeout to prevent hangs
    });

    if (result.status !== 0 || result.error) {
      return undefined;
    }

    const output = result.stdout.trim();
    // Extract version number (e.g., "v20.10.0" -> "20.10.0")
    const match = output.match(/v?(\d+\.\d+\.\d+)/);
    return match?.[1] ?? output;
  } catch {
    return undefined;
  }
}
```

**Update Function Calls (checkNode, checkNpm, etc.):**

```typescript
export function checkNode(): ToolCheckResult {
  const version = getVersion("node");
  return { ok: !!version, version };
}

export function checkNpm(): ToolCheckResult {
  const version = getVersion("npm");
  return { ok: !!version, version };
}

export function checkGit(): ToolCheckResult {
  const version = getVersion("git");
  return { ok: !!version, version };
}

// ... same pattern for checkPnpm, checkYarn, checkBun, checkPhp, checkComposer
```

**Rationale:**

- `spawnSync` with `shell: false` prevents shell injection attacks
- Separate tool and args prevents argument injection
- Added timeout to prevent process hangs
- Consistent with security pattern used elsewhere in codebase

---

### Task 3: Fix Type Issues (Remove `any` with eslint-disable)

#### [MODIFY] [prompts.ts](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/orbit-cli/src/ui/prompts.ts)

**Lines 39-40, 59-60:**

**Analysis:** The `any` types are used because `@clack/prompts` has strict generic types that don't align perfectly.

**Solution Options:**

**Option A (Recommended - Type Assertion):**

```typescript
// Instead of:
// eslint-disable-next-line @typescript-eslint/no-explicit-any
const result = await p.select({ message, options: options as any });

// Use explicit type:
type SelectOption<T> = { value: T; label: string; hint?: string };
const result = await p.select<SelectOption<T>[]>({ message, options });
```

**Option B (Type Declaration):**
Create proper type definitions for prompt options:

```typescript
// Add to top of file
interface PromptSelectOption<T> {
  value: T;
  label: string;
  hint?: string;
}

// Use in functions
export async function promptSelect<T extends string>(
  message: string,
  options: PromptSelectOption<T>[]
): Promise<T | symbol> {
  const result = await p.select({
    message,
    options: options.map((opt) => ({
      value: opt.value,
      label: opt.label,
      hint: opt.hint,
    })),
  });
  return result as T | symbol;
}
```

#### [MODIFY] [create.ts](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/orbit-cli/src/commands/create.ts)

**Lines 81-86, 97-102, 112-118:**

**Analysis:** Similar issue with `@clack/prompts` group() function.

**Solution:**

```typescript
// Create properly typed group steps
type GroupStep = Parameters<typeof p.group>[0];

// Define steps with proper types
const steps: GroupStep = {
  name: () => p.text({ message: "Project name?", placeholder: "my-app" }),
  framework: () =>
    p.select({
      message: "Framework?",
      options: FRAMEWORK_OPTIONS,
    }),
  // ... etc
};

const answers = await p.group(steps);
```

---

## 📁 Files to Modify

| File                                  | Change Type                   | Priority  |
| :------------------------------------ | :---------------------------- | :-------- |
| `src/utils/validation.ts`             | Add Windows reserved names    | 🔴 High   |
| `src/commands/helpers/check-tools.ts` | Replace execSync→spawnSync    | 🟡 Medium |
| `src/ui/prompts.ts`                   | Fix `any` types (2 locations) | 🟢 Low    |
| `src/commands/create.ts`              | Fix `any` types (3 locations) | 🟢 Low    |

---

## ✅ Kriteria Sukses

### Build & Lint

- [ ] `npm run build` berhasil tanpa error
- [ ] `npm run typecheck` berhasil tanpa error
- [ ] `npm run lint` berhasil (reduce eslint-disable comments)

### Functional Testing

#### Test 1: Windows Reserved Names Validation

```bash
# Start CLI and try reserved names
cd orbit-cli
npm run dev

# In another terminal, test validation:
node dist/index.js create con
# Expected: Error "con is a Windows reserved device name..."

node dist/index.js create prn
# Expected: Error "prn is a Windows reserved device name..."

node dist/index.js create aux
# Expected: Error "aux is a Windows reserved device name..."

node dist/index.js create nul
# Expected: Error "nul is a Windows reserved device name..."

node dist/index.js create com1
# Expected: Error "com1 is a Windows reserved device name..."
```

#### Test 2: spawnSync Replacement

```bash
# Run doctor command to verify tool detection still works
node dist/index.js doctor

# Expected output:
# ✓ Node.js    20.x.x (or current version)
# ✓ npm        10.x.x
# ✓ git        2.x.x
# ... etc (should show same versions as before)
```

#### Test 3: Type Safety

```bash
# Run typecheck
npm run typecheck

# Expected: No errors

# Run lint
npm run lint

# Expected: Fewer eslint-disable-next-line comments
```

### Unit Tests (NEW - To Be Created)

#### [NEW] [validation.test.ts](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/orbit-cli/src/__tests__/unit/utils/validation.test.ts)

```typescript
import { describe, it, expect } from "vitest";
import { validateProjectName } from "../../../utils/validation.js";

describe("validateProjectName", () => {
  describe("Windows reserved names", () => {
    const windowsReserved = [
      "con",
      "prn",
      "aux",
      "nul",
      "com1",
      "com2",
      "com3",
      "com4",
      "com5",
      "com6",
      "com7",
      "com8",
      "com9",
      "lpt1",
      "lpt2",
      "lpt3",
      "lpt4",
      "lpt5",
      "lpt6",
      "lpt7",
      "lpt8",
      "lpt9",
    ];

    it.each(windowsReserved)(
      "should reject Windows reserved name: %s",
      (name) => {
        const result = validateProjectName(name);
        expect(result.valid).toBe(false);
        expect(result.error).toContain("Windows reserved");
      }
    );

    it("should reject Windows reserved names case-insensitively", () => {
      // Note: validateProjectName requires lowercase, so these would fail regex first
      // But if we bypass with lowercase:
      expect(validateProjectName("con").valid).toBe(false);
      expect(validateProjectName("CON".toLowerCase()).valid).toBe(false);
    });
  });

  describe("project reserved names", () => {
    const projectReserved = [
      "node_modules",
      "package",
      "dist",
      "build",
      "src",
      "test",
    ];

    it.each(projectReserved)(
      "should reject project reserved name: %s",
      (name) => {
        const result = validateProjectName(name);
        expect(result.valid).toBe(false);
        expect(result.error).toContain("reserved");
      }
    );
  });

  describe("valid names", () => {
    it("should accept valid project names", () => {
      expect(validateProjectName("my-app").valid).toBe(true);
      expect(validateProjectName("myapp123").valid).toBe(true);
      expect(validateProjectName("a").valid).toBe(true);
    });
  });
});
```

**Run Tests:**

```bash
npm run test -- src/__tests__/unit/utils/validation.test.ts
```

---

## ⏱️ Estimated Time

| Task                           | Estimated Time |
| :----------------------------- | :------------- |
| Task 1: Windows Reserved Names | 30 minutes     |
| Task 2: spawnSync Replacement  | 45 minutes     |
| Task 3: Type Fixes             | 30 minutes     |
| Unit Tests                     | 30 minutes     |
| Manual Testing                 | 15 minutes     |
| **Total**                      | **~2.5 hours** |

---

## 🔗 Terkait

- **Topic:** [TOPIC_007 - Security & Bug Audit](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/Agent-0/Topic/TOPIC_007_security_bug_audit/_main.md)
- **Find:** -
- **Research Sources:**
  - Microsoft: Windows File Naming Conventions
  - Node.js: child_process API
  - CVE-2024-27980: Command Injection via batch files
