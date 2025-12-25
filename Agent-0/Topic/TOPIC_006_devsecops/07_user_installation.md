# User Installation Guide — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## Cara Public Mengakses ORBIT CLI

ORBIT CLI adalah **standalone CLI tool** yang didistribusikan melalui **npm registry** — bukan plugin AI atau extension.

---

## 1. Prerequisites

Pastikan user sudah memiliki:

| Tool    | Version  | Check Command    |
| :------ | :------- | :--------------- |
| Node.js | ≥18.20.0 | `node --version` |
| npm     | ≥10.0.0  | `npm --version`  |

---

## 2. Installation Methods

### Metode A: Global Install (Recommended)

```bash
# Install sekali, gunakan di mana saja
npm install -g orbit-cli

# Verifikasi instalasi
orbit --version

# Gunakan
orbit create my-app
```

**Keuntungan:**

- ✅ Perintah `orbit` tersedia di mana saja
- ✅ Tidak perlu download ulang
- ✅ Update mudah: `npm update -g orbit-cli`

---

### Metode B: npx (Tanpa Install)

```bash
# Langsung jalankan tanpa install global
npx orbit-cli create my-app

# Atau spesifik versi
npx orbit-cli@1.0.0 create my-app
```

**Keuntungan:**

- ✅ Tidak perlu install
- ✅ Selalu versi terbaru (atau bisa pilih versi)
- ✅ Cocok untuk one-time use

---

### Metode C: Project Dependency

```bash
# Tambahkan ke project sebagai devDependency
npm install -D orbit-cli

# Jalankan via npx atau npm scripts
npx orbit create component

# Atau tambahkan script di package.json
{
  "scripts": {
    "scaffold": "orbit create"
  }
}
```

---

## 3. Basic Usage

```bash
# Create new project (interactive mode)
orbit create my-app

# Create with specific framework
orbit create my-app --template nextjs

# Create with package manager preference
orbit create my-app --pm pnpm

# Create with stack preset
orbit create my-app --stack full

# Skip prompts, use defaults
orbit create my-app -y

# List available frameworks
orbit list

# Check system requirements
orbit doctor
```

---

## 4. Quick Start Examples

### Example 1: Create Next.js Project

```bash
# Interactive
orbit create my-nextjs-app

# Select "Next.js" when prompted
# Select "TypeScript" when prompted
# Select "Full Stack" preset
```

### Example 2: Create Nuxt Project (Non-interactive)

```bash
orbit create my-nuxt-app --template nuxt --pm pnpm --stack standard -y
```

### Example 3: Check Environment First

```bash
# Verify all tools are installed
orbit doctor

# Expected output:
# ✓ Node.js v20.10.0
# ✓ npm v10.2.3
# ✓ git v2.40.0
# ✓ pnpm v8.10.0 (optional)
```

---

## 5. Update ORBIT CLI

```bash
# Check current version
orbit --version

# Update to latest
npm update -g orbit-cli

# Or reinstall
npm install -g orbit-cli@latest
```

---

## 6. Uninstall

```bash
# Remove global installation
npm uninstall -g orbit-cli
```

---

## 7. Troubleshooting

### Error: Command not found

```bash
# Check if npm bin is in PATH
npm bin -g

# Add to PATH if needed (Linux/Mac)
export PATH="$(npm bin -g):$PATH"

# Or use npx instead
npx orbit-cli --version
```

### Error: Permission denied

```bash
# Linux/Mac: Use sudo (not recommended)
sudo npm install -g orbit-cli

# Better: Fix npm permissions
# https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally
```

### Error: Node version too old

```bash
# Check version
node --version

# Update Node.js to v18.20.0 or later
# Use nvm (recommended):
nvm install 20
nvm use 20
```

---

## 8. Verification Checklist

```bash
# After installation, verify:
orbit --version      # Should show version
orbit --help         # Should show help
orbit doctor         # Should show system status
orbit list           # Should list frameworks
```

---

## 9. Getting Help

```bash
# Show help
orbit --help

# Command-specific help
orbit create --help
orbit list --help
orbit doctor --help
```

---

## 10. Links

| Resource      | URL                                          |
| :------------ | :------------------------------------------- |
| npm Package   | https://www.npmjs.com/package/orbit-cli      |
| GitHub Repo   | https://github.com/username/orbit-cli        |
| Documentation | https://github.com/username/orbit-cli#readme |
| Issues        | https://github.com/username/orbit-cli/issues |
