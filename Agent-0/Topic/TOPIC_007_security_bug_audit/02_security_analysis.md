# Security Vulnerabilities Analysis

**Parent:** [← Kembali ke Main](_main.md)  
**Status:** ✅ Selesai

---

## 🔐 Command Injection Analysis

### Research Summary: 2024 CVEs

Dari deep research, ditemukan vulnerability kritis di Node.js 2024:

#### CVE-2024-27980 (HIGH SEVERITY)

**Affected:** Node.js 18.x, 20.x, 21.x on Windows  
**Issue:** Command injection via `child_process.spawn` on Windows dengan batch files

**Attack Vector:**

```javascript
// ❌ VULNERABLE (Windows)
spawn("git", ["--version"], { shell: false });
// Batch files can inject commands even without shell!
```

**Mitigation:** Update Node.js >= 18.20.1, 20.12.1, 21.7.2

---

### ORBIT CLI Command Execution Audit

#### ✅ **GOOD PRACTICES FOUND**

1. **Consistent Use of `spawn()` with `shell: false`**

**Locations:**

- `src/core/services/git-initializer.ts`
- `src/core/services/framework-installer.ts`
- `src/core/services/config-applier.ts`
- `src/core/services/tool-detector.ts`
- `src/utils/safe-executor.ts`
- `src/utils/executor.ts`

**Example (git-initializer.ts:31):**

```typescript
const child = spawn("git", args, {
  cwd: projectPath,
  shell: false, // SECURITY: Never use shell!
  stdio: ["inherit", "inherit", "pipe"],
});
```

**Example (safe-executor.ts:84-86):**

```typescript
const spawnOptions: SpawnOptions = {
  cwd: options?.cwd ?? process.cwd(),
  shell: false, // CRITICAL: Never use shell!
  stdio: ["pipe", "pipe", "pipe"],
  env: sanitizeEnv(options?.env),
};
```

✅ **VERDICT:** Mengikuti best practice untuk mencegah command injection

---

2. **Environment Variable Sanitization**

**Location:** `src/utils/safe-executor.ts:28-72`

```typescript
const SENSITIVE_ENV_KEYS = [
  "AWS_SECRET_ACCESS_KEY",
  "AWS_ACCESS_KEY_ID",
  "GITHUB_TOKEN",
  "GH_TOKEN",
  "NPM_TOKEN",
  "DATABASE_URL",
  "DB_PASSWORD",
  "API_KEY",
  "SECRET_KEY",
  "PRIVATE_KEY",
  "JWT_SECRET",
  "SESSION_SECRET",
  "ENCRYPTION_KEY",
  "SUPABASE_SERVICE_KEY",
  "STRIPE_SECRET_KEY",
] as const;

export function sanitizeEnv(env?: NodeJS.ProcessEnv): NodeJS.ProcessEnv {
  const baseEnv = env ?? process.env;
  const sanitized = { ...baseEnv };

  for (const key of SENSITIVE_ENV_KEYS) {
    delete sanitized[key];
  }

  // Pattern matching for additional security
  for (const key of Object.keys(sanitized)) {
    const upperKey = key.toUpperCase();
    if (
      upperKey.includes("SECRET") ||
      upperKey.includes("PASSWORD") ||
      upperKey.includes("_TOKEN") ||
      upperKey.includes("PRIVATE_KEY")
    ) {
      delete sanitized[key];
    }
  }

  return sanitized;
}
```

✅ **VERDICT:** Excellent practice - prevents accidental secret leakage

---

3. **Security Documentation**

Every file using `spawn` includes security comments:

```typescript
/**
 * SECURITY: Uses spawn with shell:false - NEVER use exec!
 */
```

✅ **VERDICT:** Good security awareness

---

#### ⚠️ **POTENTIAL RISKS**

### 1. Use of `execSync` in check-tools.ts

**Location:** `src/commands/helpers/check-tools.ts:18-21`

```typescript
function getVersion(command: string): string | undefined {
  try {
    const output = execSync(command, {
      encoding: 'utf-8',
      stdio: ['pipe', 'pipe', 'pipe'],
    }).trim();
```

**Used For:**

- `node --version`
- `npm --version`
- `git --version`
- `pnpm --version`
- `yarn --version`
- `bun --version`
- `php --version`
- `composer --version`

**Risk Level:** 🟡 **MEDIUM**

**Why It's Vulnerable:**

- `execSync` passes command to shell
- Shell interprets special characters
- If `command` variable contains user input → Command injection

**Current State:**

```typescript
// Currently HARDCODED - NO user input
const version = getVersion("node --version"); // ✅ SAFE
```

**Potential Attack (if refactored badly):**

```typescript
// ❌ DANGEROUS if someone does this:
const tool = userInput; // e.g., "node; rm -rf /"
const version = getVersion(`${tool} --version`); // INJECTION!
```

**Recommendation:** Refactor ke `spawn` untuk konsistensi

```typescript
// ✅ SAFER ALTERNATIVEimport { spawn } from 'node:child_process';

function getVersion(tool: string): string | undefined {
  try {
    const result = spawnSync(tool, ["--version"], {
      encoding: "utf-8",
      shell: false, // SECURE!
    });
    if (result.status === 0) {
      const match = result.stdout.match(/v?(\d+\.\d+\.\d+)/);
      return match?.[1] ?? result.stdout.trim();
    }
  } catch {
    return undefined;
  }
}
```

---

### 2. No Input Validation Before Command Execution

**Risk:** User-controlled values digunakan langsung untuk:

#### A. Project Name → Directory Name

**Location:** `src/core/services/framework-installer.ts:29`

```typescript
return path.join(process.cwd(), input.name);
```

**Attack Vector (Path Traversal):**

```bash
# User input:
orbit create "../../etc/passwd"

# Results in:
path.join(process.cwd(), "../../etc/passwd")
# Could create files outside intended directory!
```

**Risk Level:** 🔴 **HIGH**

---

#### B. Project Name → Command Arguments

**Location:** `src/core/services/framework-installer.ts:35-71`

```typescript
// User input langsung ke args
const commands: Record<FrameworkId, { command: string; args: string[] }> = {
  nextjs: {
    command: "npx",
    args: ["create-next-app@latest", name, "--typescript", "--eslint"],
    //                                ^^^^ NO VALIDATION!
  },
};
```

**Attack Vector:**

```bash
# User input:
orbit create "test --template malicious"

# Spawned command:
npx create-next-app@latest test --template malicious --typescript --eslint
#                                ^^^^^^^^^^^^^^^^^^^^ Unintended flag injection!
```

**Risk Level:** 🟡 **MEDIUM** (mitigated by `shell: false`, tapi masih bisa inject flags)

---

## 🎯 CVE-2024-27980 Impact on ORBIT

### Windows Batch File Attack

**Scenario:** Windows user runs ORBIT

**Exploitation:**

1. User creates malicious batch file: `npm.bat`

```batch
@echo off
echo Malicious code
del /F /Q /S important-files
npm %*
```

2. User runs ORBIT with PATH pointing to malicious batch:

```bash
orbit create my-app
```

3. ORBIT executes:

```typescript
spawn("npm", ["create", "astro@latest", "my-app"], { shell: false });
```

4. On Windows, `npm` resolves to `npm.bat` → Malicious code runs

**Mitigation:**

- Require Node.js >= 18.20.1
- Validate `PATH` environment variable
- Use full executable paths on Windows

---

## 📋 Recommendations

### 🔴 CRITICAL

1. **Add Input Validation (Zod)**

   ```typescript
   const ProjectNameSchema = z
     .string()
     .min(1)
     .max(100)
     .regex(/^[a-z0-9-_]+$/i, "Only alphanumeric, dash, underscore allowed")
     .refine((val) => !val.includes(".."), "Path traversal not allowed")
     .refine((val) => !val.startsWith("-"), "Cannot start with dash");
   ```

2. **Validate `package.json` engines**
   ```json
   {
     "engines": {
       "node": ">=18.20.1 || >=20.12.1 || >=21.7.2"
     }
   }
   ```

### 🟡 MEDIUM

1. **Replace `execSync` with `spawnSync`** in `check-tools.ts`

2. **Add Absolute Path Resolution for Commands on Windows**

   ```typescript
   if (process.platform === "win32") {
     // Resolve to full path to avoid batch file injection
     command = await which(command);
   }
   ```

3. **Add Runtime Node.js Version Check**
   ```typescript
   const nodeVersion = process.version;
   if (isVulnerableVersion(nodeVersion)) {
     console.error("Node.js version vulnerable to CVE-2024-27980");
     process.exit(1);
   }
   ```

---

## ✅ Security Compliance Matrix

| Check                                       | Status | Notes                               |
| :------------------------------------------ | :----- | :---------------------------------- |
| No `exec()` or `execSync()` with user input | ⚠️     | `execSync` used but hardcoded       |
| All `spawn()` use `shell: false`            | ✅     | Consistent across codebase          |
| Environment sanitization                    | ✅     | Comprehensive sensitive var removal |
| Input validation before commands            | ❌     | **MISSING - CRITICAL**              |
| Path traversal prevention                   | ❌     | **MISSING - CRITICAL**              |
| Node.js version requirements                | ⚠️     | Should specify >= 18.20.1           |

---

## 🔗 Terkait

- [03_input_validation.md](03_input_validation.md) - Implementasi Zod validation
- [07_best_practices.md](07_best_practices.md) - Node.js security best practices
- [08_findings_recommendations.md](08_findings_recommendations.md) - Consolidated recommendations
