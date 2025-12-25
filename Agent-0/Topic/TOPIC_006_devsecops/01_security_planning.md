# Security Planning — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Security by Design

### 1.1 Threat Model for CLI Applications

| Threat                   | Vector                     | Likelihood | Impact      | Mitigation                               |
| :----------------------- | :------------------------- | :--------- | :---------- | :--------------------------------------- |
| **Command Injection**    | User input in spawn        | 🔴 High    | 🔴 Critical | spawn with shell:false, input validation |
| **Path Traversal**       | User-provided paths        | 🟡 Medium  | 🔴 Critical | Path validation, sandbox to cwd          |
| **Dependency Confusion** | Typosquatting npm packages | 🟡 Medium  | 🔴 Critical | Lock files, npm audit                    |
| **Supply Chain Attack**  | Compromised dependencies   | 🟡 Medium  | 🔴 Critical | Minimal deps, regular audits             |
| **Credential Leakage**   | Env vars in child process  | 🟡 Medium  | 🟡 High     | Sanitized environment                    |
| **DoS**                  | Infinite loops, memory     | 🟢 Low     | 🟡 Medium   | Timeouts, resource limits                |

### 1.2 Security Requirements

| Req ID  | Requirement          | Priority | Implementation             |
| :------ | :------------------- | :------- | :------------------------- |
| SEC-001 | No shell execution   | MUST     | spawn with shell:false     |
| SEC-002 | Input validation     | MUST     | Zod schemas for all input  |
| SEC-003 | Path sanitization    | MUST     | No `..` traversal allowed  |
| SEC-004 | No hardcoded secrets | MUST     | Environment variables only |
| SEC-005 | Minimal dependencies | SHOULD   | < 10 production deps       |
| SEC-006 | Regular audits       | MUST     | npm audit in CI            |
| SEC-007 | Lockfile integrity   | MUST     | lockfile-lint in CI        |
| SEC-008 | CVE monitoring       | SHOULD   | Dependabot enabled         |

---

## 2. Security Architecture

### 2.1 Trust Boundaries

```
┌──────────────────────────────────────────────────────────────┐
│                     UNTRUSTED ZONE                           │
│  • User input (CLI arguments)                                │
│  • User-provided paths                                       │
│  • Environment variables                                     │
│  • Network responses (if any)                                │
└──────────────────────────┬───────────────────────────────────┘
                           │
                    ┌──────▼──────┐
                    │  VALIDATION │  ← Trust Boundary
                    │    LAYER    │
                    └──────┬──────┘
                           │
┌──────────────────────────▼───────────────────────────────────┐
│                      TRUSTED ZONE                            │
│  • Validated input                                           │
│  • Sanitized paths                                           │
│  • Hardcoded command lists                                   │
│  • Internal configuration                                    │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 Security Controls by Layer

| Layer                 | Controls                                              |
| :-------------------- | :---------------------------------------------------- |
| **Input (CLI args)**  | Zod validation, type coercion, length limits          |
| **File System**       | Path traversal check, cwd sandbox, permission check   |
| **Process Execution** | spawn only, whitelisted commands, sanitized env       |
| **Dependencies**      | Minimal footprint, regular audits, lockfile integrity |
| **Output**            | No sensitive data in logs, masked secrets             |

---

## 3. Security Policies

### 3.1 Secure Coding Policy

```markdown
## ORBIT CLI Secure Coding Policy

1. **Input Validation**

   - ALL user input MUST be validated using Zod schemas
   - Validation MUST occur before any processing
   - Invalid input MUST result in clear error messages

2. **Command Execution**

   - NEVER use child_process.exec()
   - ALWAYS use spawn() or execFile() with shell:false
   - ONLY execute whitelisted commands

3. **File System Access**

   - ALL paths MUST be validated for traversal attacks
   - Operations MUST be restricted to user's cwd
   - Sensitive files (.env, secrets) MUST NOT be read

4. **Secrets Management**

   - NEVER hardcode secrets in source code
   - Use environment variables for sensitive data
   - Sanitize env before passing to child processes

5. **Dependencies**
   - Keep production dependencies minimal (< 10)
   - Run npm audit before every release
   - Update dependencies regularly
   - Use exact versions in package.json
```

### 3.2 Approved Third-Party Commands

```typescript
// Whitelist of commands ORBIT CLI may execute
const ALLOWED_COMMANDS = new Set([
  // Package managers
  "npm",
  "npx",
  "yarn",
  "pnpm",
  "bun",

  // Version control
  "git",

  // Runtimes (for version check only)
  "node",
  "php",
  "composer",

  // Detection commands
  "which", // Unix
  "where", // Windows
]);
```

---

## 4. Compliance Checklist

### 4.1 OWASP Top 10 for CLI

| OWASP                              | Applicability      | Status                           |
| :--------------------------------- | :----------------- | :------------------------------- |
| A03:2021 Injection                 | ✅ Highly Relevant | Mitigated via spawn + validation |
| A05:2021 Security Misconfiguration | ✅ Relevant        | Config hardening documented      |
| A06:2021 Vulnerable Components     | ✅ Relevant        | npm audit + Dependabot           |
| A08:2021 Software Integrity        | ✅ Relevant        | Lockfile + provenance            |

### 4.2 Node.js Security Best Practices

| Practice               | Implemented | Reference                   |
| :--------------------- | :---------- | :-------------------------- |
| Use `spawn` not `exec` | ✅ Yes      | TOPIC_004/03_security       |
| Validate all input     | ✅ Yes      | TOPIC_004/03_security       |
| Use strict mode        | ✅ Yes      | TOPIC_005/04_configurations |
| Minimal dependencies   | ✅ Yes      | TOPIC_005/01_tech_stack     |
| Regular audits         | 🔲 CI/CD    | This topic                  |
| Lock dependencies      | ✅ Yes      | package-lock.json           |

---

## 5. Security Documentation Requirements

Before release, ensure these documents exist:

- [ ] `SECURITY.md` — Vulnerability disclosure policy
- [ ] `CONTRIBUTING.md` — Secure development guidelines
- [ ] `.github/CODEOWNERS` — Security review requirements
- [ ] `docs/security.md` — User security guide
