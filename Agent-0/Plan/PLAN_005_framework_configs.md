# Project Setup: Framework Configs

**ID:** PLAN_005 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_005/06_framework_configs.md](../Topic/TOPIC_005_project_setup/06_framework_configs.md)

---

## 🎯 Tujuan

Membuat file konfigurasi untuk setiap framework yang didukung ORBIT CLI — Next.js, Nuxt, Astro, SvelteKit, Vue, Remix, Laravel.

---

## 📁 Working Directory

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_001 selesai (package.json, node_modules)
- [x] PLAN_002 selesai (src/ folder structure)
- [x] PLAN_003 selesai (configuration files)
- [x] PLAN_004 selesai (initial files)

---

## 🛠️ Strategi Implementasi

### Step 1: Update Framework Types

Update `src/frameworks/types.ts` dengan interface lengkap:

- `Framework` dengan `installCommand` dan `requiredTools`
- `InstallCommand` dengan flags per package manager
- `StackPreset` dengan `postInstallDeps`

**Kode:** TOPIC_005/06_framework_configs.md Section 1

---

### Step 2: Create Next.js Config

Create `src/frameworks/nextjs.ts`:

- Install commands untuk npm/yarn/pnpm/bun
- Flags: --ts, --eslint, --tailwind, --src-dir
- Stack presets: minimal, standard, full

**Kode:** TOPIC_005/06_framework_configs.md Section 2.1

---

### Step 3: Create Nuxt Config

Create `src/frameworks/nuxt.ts`:

- Install commands dengan nuxi
- Stack presets dengan @nuxtjs modules

**Kode:** TOPIC_005/06_framework_configs.md Section 2.2

---

### Step 4: Create Astro Config

Create `src/frameworks/astro.ts`:

- Template flags
- Stack presets dengan MDX, React integration

**Kode:** TOPIC_005/06_framework_configs.md Section 2.3

---

### Step 5: Create SvelteKit Config

Create `src/frameworks/sveltekit.ts`:

- sv create command
- Stack presets dengan Drizzle

**Kode:** TOPIC_005/06_framework_configs.md Section 2.4

---

### Step 6: Create Vue Config

Create `src/frameworks/vue.ts`:

- create-vue command
- Stack presets dengan Router, Pinia

**Kode:** TOPIC_005/06_framework_configs.md Section 2.5

---

### Step 7: Create Remix Config

Create `src/frameworks/remix.ts`:

- create-remix command
- Stack presets dengan Prisma

**Kode:** TOPIC_005/06_framework_configs.md Section 2.6

---

### Step 8: Create Laravel Config

Create `src/frameworks/laravel.ts`:

- composer command (PHP)
- requiredTools: php, composer

**Kode:** TOPIC_005/06_framework_configs.md Section 2.7

---

### Step 9: Create Command Builder Service

Create `src/core/services/command-builder.ts`:

- `buildInstallCommand()` function
- Handles flags dan post-install deps

**Kode:** TOPIC_005/06_framework_configs.md Section 3

---

### Step 10: Update Framework Registry

Update `src/frameworks/index.ts`:

- Import semua framework configs
- Use lazy loading

---

### Step 11: Build & Verify

```bash
npm run build
orbit list
```

---

## ✅ Kriteria Sukses

- [x] src/frameworks/types.ts updated
- [x] src/frameworks/nextjs.ts created
- [x] src/frameworks/nuxt.ts created
- [x] src/frameworks/astro.ts created
- [x] src/frameworks/sveltekit.ts created
- [x] src/frameworks/vue.ts created
- [x] src/frameworks/remix.ts created
- [x] src/frameworks/laravel.ts created
- [x] src/core/services/command-builder.ts created
- [x] src/frameworks/index.ts updated
- [x] `npm run build` passes ✅
- [x] `orbit list` shows all frameworks ✅

---

## ⏭️ Next Plan

Setelah plan ini selesai: Development phase selesai!
Next: TOPIC_003 & TOPIC_004 implementation (full functionality)

---

## 🔗 Terkait

Topic: [TOPIC_005](../Topic/TOPIC_005_project_setup/_main.md)
Ref: [06_framework_configs.md](../Topic/TOPIC_005_project_setup/06_framework_configs.md)
