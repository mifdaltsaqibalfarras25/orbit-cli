# Deployment & Publishing — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. npm Publishing Overview

### 1.1 Publishing Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                   ORBIT CLI Publishing Flow                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Developer                     GitHub Actions                   │
│      │                              │                           │
│      │ 1. git tag v1.0.0            │                           │
│      │ 2. git push --tags           │                           │
│      └─────────────────────────────▶│                           │
│                                     │                           │
│                                     ▼                           │
│                              ┌─────────────┐                    │
│                              │    CI/CD    │                    │
│                              │  Pipeline   │                    │
│                              └──────┬──────┘                    │
│                                     │                           │
│                              Pass all checks                    │
│                                     │                           │
│                                     ▼                           │
│                              ┌─────────────┐                    │
│                              │   npm       │  ← OIDC Token      │
│                              │  publish    │                    │
│                              └──────┬──────┘                    │
│                                     │                           │
│                              With provenance                    │
│                                     │                           │
│                                     ▼                           │
│                              ┌─────────────┐                    │
│                              │  npmjs.org  │                    │
│                              │  registry   │                    │
│                              └─────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Pre-Publish Checklist

```markdown
## ORBIT CLI Release Checklist

### Code Quality

- [ ] All tests passing
- [ ] Coverage thresholds met
- [ ] ESLint 0 errors
- [ ] TypeScript compiles with 0 errors

### Security

- [ ] `npm audit` shows 0 vulnerabilities
- [ ] No hardcoded secrets in code
- [ ] `lockfile-lint` passes

### Documentation

- [ ] README.md up to date
- [ ] CHANGELOG.md updated
- [ ] Version bumped in package.json

### Files

- [ ] `npm publish --dry-run` reviewed
- [ ] No sensitive files in package
- [ ] `.npmignore` or `files` field configured
```

---

## 3. Package Content Configuration

### 3.1 Using `files` in package.json (Recommended)

```json
{
  "name": "orbit-cli",
  "files": ["dist", "README.md", "LICENSE"]
}
```

### 3.2 .npmignore

```
# Source (not needed in published package)
src/
__tests__/

# Config files
tsconfig.json
tsup.config.ts
eslint.config.js
vitest.config.ts
.prettierrc

# Git
.git/
.github/
.gitignore

# IDE
.vscode/
.idea/

# Development
node_modules/
coverage/
*.log

# Sensitive
.env*
.npmrc
```

### 3.3 Verify Package Contents

```bash
# Dry run - see what will be published
npm publish --dry-run

# Or use npm pack to create tarball
npm pack
tar -tzf orbit-cli-1.0.0.tgz
```

---

## 4. Trusted Publishing with OIDC

### 4.1 Why Trusted Publishing?

| Aspect         | Classic Token  | Trusted Publishing    |
| :------------- | :------------- | :-------------------- |
| Token lifetime | Long-lived     | Short-lived (minutes) |
| Rotation       | Manual         | Automatic             |
| Scope          | Static         | Workflow-specific     |
| Provenance     | Optional       | Automatic             |
| Setup          | Manual secrets | OIDC config           |

### 4.2 Configure npm for OIDC

```bash
# On npmjs.com:
# 1. Go to package settings
# 2. Enable "Trusted Publishing"
# 3. Add GitHub repository

# Trust configuration:
# - Owner: your-github-username
# - Repository: orbit-cli
# - Workflow: release.yml
# - Environment: (optional)
```

### 4.3 GitHub Actions with OIDC

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
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          registry-url: "https://registry.npmjs.org"

      - run: npm ci
      - run: npm run build

      # Publish with provenance
      - run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## 5. Version Management

### 5.1 Semantic Versioning

```
MAJOR.MINOR.PATCH

MAJOR: Breaking changes
MINOR: New features (backward compatible)
PATCH: Bug fixes (backward compatible)
```

### 5.2 Version Bump Commands

```bash
# Patch: 1.0.0 → 1.0.1
npm version patch

# Minor: 1.0.0 → 1.1.0
npm version minor

# Major: 1.0.0 → 2.0.0
npm version major

# Pre-release: 1.0.0 → 1.0.1-beta.0
npm version prerelease --preid=beta
```

### 5.3 Release Script

```bash
#!/bin/bash
# scripts/release.sh

set -e

# Ensure clean working directory
if [[ -n $(git status -s) ]]; then
  echo "Error: Working directory not clean"
  exit 1
fi

# Get version type
VERSION_TYPE=${1:-patch}

# Run all checks
npm run lint
npm run test:run
npm audit --audit-level=high

# Build
npm run build

# Bump version
npm version $VERSION_TYPE -m "chore: release v%s"

# Push with tags
git push --follow-tags

echo "✅ Release triggered! GitHub Actions will publish to npm."
```

---

## 6. CHANGELOG Management

### 6.1 CHANGELOG.md Format

```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Added

- New features

### Changed

- Updates to existing features

### Fixed

- Bug fixes

### Security

- Security patches

## [1.0.0] - 2024-12-25

### Added

- Initial release
- Framework support: Next.js, Nuxt, Astro, SvelteKit, Vue, Remix, Laravel
- Interactive CLI with @clack/prompts
- Stack presets (minimal, standard, full)

[Unreleased]: https://github.com/username/orbit-cli/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/username/orbit-cli/releases/tag/v1.0.0
```

---

## 7. npm Account Security

### 7.1 Enable 2FA

```
npmjs.com → Account Settings → Two-Factor Authentication

✅ Enable 2FA for authentication
✅ Enable 2FA for publishing (REQUIRED for security)

Recommended: Use FIDO/WebAuthn over TOTP
```

### 7.2 Granular Access Tokens

```
npmjs.com → Access Tokens → Generate New Token

Type: Granular Access Token
Name: github-actions-orbit-cli
Expiration: 365 days
Packages: Select packages → orbit-cli
Permissions: Read and write
IP Allowlist: (optional) GitHub Actions IPs
```

---

## 8. Post-Publish Verification

```bash
# Verify package was published
npm view orbit-cli

# Check provenance
npm audit signatures

# Test installation
npx orbit-cli --version

# Check published files
npm pack orbit-cli
tar -tzf orbit-cli-*.tgz
```
