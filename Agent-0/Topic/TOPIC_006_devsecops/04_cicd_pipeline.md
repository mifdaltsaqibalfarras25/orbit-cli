# CI/CD Pipeline — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Pipeline Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    ORBIT CLI CI/CD Pipeline                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐         │
│  │  Lint   │──▶│  Test   │──▶│ Audit   │──▶│  Build  │         │
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘         │
│       │             │             │             │               │
│       ▼             ▼             ▼             ▼               │
│    ESLint       Vitest      npm audit       tsup              │
│    Prettier     Coverage    lockfile-lint   Type check         │
│                                                                 │
│  If main branch + release tag:                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                      PUBLISH                            │   │
│  │   npm publish (with OIDC Trusted Publishing)            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. GitHub Actions Workflows

### 2.1 CI Workflow (Every Push/PR)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Format check
        run: npm run format:check

      - name: Type check
        run: npm run typecheck

  test:
    name: Test
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm run test:coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true

  security:
    name: Security Audit
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: npm audit
        run: npm audit --audit-level=high

      - name: Lockfile lint
        run: npx lockfile-lint --path package-lock.json --type npm --validate-https --allowed-hosts npm

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [test, security]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
```

### 2.2 Release Workflow (Tags Only)

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - "v*"

permissions:
  contents: read
  id-token: write # Required for OIDC

jobs:
  build-and-publish:
    name: Build and Publish
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
          registry-url: "https://registry.npmjs.org"

      - name: Install dependencies
        run: npm ci

      - name: Run all checks
        run: |
          npm run lint
          npm run test:run
          npm audit --audit-level=high

      - name: Build
        run: npm run build

      - name: Publish to npm (Trusted Publishing)
        run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## 3. Security Best Practices for CI/CD

### 3.1 Secrets Management

```yaml
# ✅ CORRECT - Use GitHub Secrets
env:
  NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

# ❌ WRONG - Hardcoded token
env:
  NODE_AUTH_TOKEN: "npm_abc123..."  # NEVER DO THIS!
```

### 3.2 Minimal Permissions

```yaml
# Default to read-only
permissions:
  contents: read

# Only add write when needed
jobs:
  publish:
    permissions:
      contents: read
      id-token: write # For OIDC
```

### 3.3 Pin Action Versions

```yaml
# ✅ SECURE - Pinned to commit SHA
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1

# ⚠️ ACCEPTABLE - Pinned to major version
- uses: actions/checkout@v4

# ❌ INSECURE - Using latest
- uses: actions/checkout@latest
```

### 3.4 Prevent Fork Attacks

```yaml
on:
  pull_request:
    branches: [main]

# Only run on trusted branches, not forks with secrets
jobs:
  build:
    if: github.event.pull_request.head.repo.full_name == github.repository
```

---

## 4. Dependabot Configuration

### 4.1 Enable Dependabot

```yaml
# .github/dependabot.yml
version: 2
updates:
  # npm dependencies
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 5
    groups:
      dev-dependencies:
        dependency-type: "development"
        update-types: ["minor", "patch"]
      prod-dependencies:
        dependency-type: "production"
        update-types: ["minor", "patch"]
    ignore:
      - dependency-name: "*"
        update-types: ["version-update:semver-major"]
    labels:
      - "dependencies"
      - "security"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    labels:
      - "ci"
```

### 4.2 Auto-merge Safe Updates

```yaml
# .github/workflows/dependabot-automerge.yml
name: Dependabot Auto-merge

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  automerge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - uses: dependabot/fetch-metadata@v2
        id: metadata

      - name: Auto-merge patch updates
        if: steps.metadata.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 5. Branch Protection Rules

```
Repository Settings > Branches > Add rule

Branch name pattern: main

✅ Require a pull request before merging
  ✅ Require approvals: 1
  ✅ Dismiss stale PR approvals

✅ Require status checks to pass
  ✅ Require branches to be up to date
  Required checks:
    - Lint
    - Test
    - Security Audit
    - Build

✅ Require signed commits (optional but recommended)

✅ Do not allow bypassing the above settings
```

---

## 6. CODEOWNERS

```
# .github/CODEOWNERS

# Default owners
* @username

# Security-critical paths require security review
/src/core/validation/ @username @security-team
/src/utils/safe-*.ts @username @security-team
/.github/workflows/ @username @devops-team
```
