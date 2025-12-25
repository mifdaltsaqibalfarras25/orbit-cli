# Project Setup: Installation

**ID:** PLAN_001 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_005/02_installation.md](../Topic/TOPIC_005_project_setup/02_installation.md)

---

## 🎯 Tujuan

Inisialisasi project ORBIT CLI dari nol hingga build pertama berhasil — mencakup npm init, package.json, dan semua dependencies terinstall.

---

## 📋 Prerequisites

- [x] Node.js ≥18.20.0 terinstall
- [x] npm ≥10.0.0 terinstall
- [x] Working directory kosong / siap dipakai

---

## � Working Directory

> ⚠️ **PENTING:** Semua command harus dijalankan di folder berikut:

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

**Struktur akhir:**

```
orbit/
├── Agent-0/       # Dokumentasi (sudah ada)
├── .agent/        # Standards (sudah ada)
└── orbit-cli/     # Source code (dibuat oleh plan ini)
    ├── package.json
    ├── node_modules/
    └── ...
```

---

## �🛠️ Strategi Implementasi

### Step 1: Create Project Directory

```bash
# Dari folder orbit, buat subfolder orbit-cli
cd /home/mifdlalalf2025/Documents/Proyek\ 2025/orbit
mkdir orbit-cli
cd orbit-cli
```

**Validation:** Directory `/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/` berhasil dibuat

---

### Step 2: Initialize npm Project

```bash
npm init -y
```

**Validation:** `package.json` terbuat

---

### Step 3: Configure package.json

Edit `package.json` dengan content berikut:

```json
{
  "name": "orbit-cli",
  "version": "1.0.0",
  "description": "Universal Project Generator CLI",
  "type": "module",
  "engines": {
    "node": ">=18.20.0"
  },
  "bin": {
    "orbit": "./dist/index.js"
  },
  "files": ["dist"],
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "scripts": {
    "dev": "tsup --watch",
    "build": "tsup",
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "lint": "eslint src/",
    "format": "prettier --write src/",
    "format:check": "prettier --check src/",
    "typecheck": "tsc --noEmit",
    "prepublishOnly": "npm run build"
  },
  "keywords": [
    "cli",
    "project-generator",
    "scaffold",
    "nextjs",
    "nuxt",
    "astro",
    "sveltekit"
  ],
  "author": "",
  "license": "MIT"
}
```

**Validation:** `cat package.json` shows correct content

---

### Step 4: Install Production Dependencies

```bash
npm install commander@^13.0.0 @clack/prompts@^0.9.0 chalk@^5.3.0 ora@^8.1.0 gradient-string@^3.0.0 figlet@^1.8.0 zod@^3.24.0
```

**Expected:** 7 packages added

**Validation:**

```bash
npm list --depth=0
```

---

### Step 5: Install Development Dependencies

```bash
npm install -D typescript@^5.7.0 tsup@^8.3.0 @types/node@^22.0.0 @types/figlet@^1.5.8 vitest@^2.1.0 @vitest/coverage-v8@^2.1.0 eslint@^9.16.0 prettier@^3.4.0 typescript-eslint@^8.18.0
```

**Expected:** ~150 packages added

**Validation:**

```bash
npm list --depth=0 --dev
```

---

### Step 6: Verify Installation

```bash
# Check package.json has all deps
cat package.json | grep -A 20 '"dependencies"'

# Check node_modules exists
ls node_modules/ | head -10

# Quick sanity check
npx tsc --version
npx tsup --version
```

---

## ✅ Kriteria Sukses

- [x] `package.json` memiliki `"type": "module"`
- [x] `package.json` memiliki `"bin": { "orbit": "./dist/index.js" }`
- [x] 7 production dependencies terinstall
- [x] 9 development dependencies terinstall
- [x] `npm list` tidak menunjukkan missing deps
- [x] `npx tsc --version` menunjukkan TypeScript 5.7+ (actual: 5.9.3)
- [x] `npx tsup --version` menunjukkan tsup 8.3+ (actual: 8.5.1)

---

## 📝 Notes untuk AI Implementor

1. **Node version:** Pastikan ≥18.20.0 untuk CVE-2024-27980 mitigation
2. **ESM-only:** Chalk v5, Ora v8 adalah ESM-only — wajib `"type": "module"`
3. **Exact versions:** Gunakan versi spesifik seperti tercantum di atas
4. **No folder structure yet:** Plan ini HANYA install deps, folder dibuat di PLAN_002

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_002_folder_structure**

---

## 🔗 Terkait

Topic: [TOPIC_005](../Topic/TOPIC_005_project_setup/_main.md)
Ref: [01_tech_stack.md](../Topic/TOPIC_005_project_setup/01_tech_stack.md) | [02_installation.md](../Topic/TOPIC_005_project_setup/02_installation.md)
