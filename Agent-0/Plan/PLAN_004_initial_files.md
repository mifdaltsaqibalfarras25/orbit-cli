# Project Setup: Initial Files

**ID:** PLAN_004 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_005/05_initial_files.md](../Topic/TOPIC_005_project_setup/05_initial_files.md)

---

## 🎯 Tujuan

Mengisi file TypeScript kosong dengan kode implementasi awal — mencakup entry point, commands, core, UI, frameworks, dan utils.

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

---

## 🛠️ Strategi Implementasi

> **Note:** Konten file lengkap tersedia di source file. Plan ini hanya mencatat file mana yang harus diisi.

### Step 1: Entry Point & CLI

| File           | Purpose                         |
| :------------- | :------------------------------ |
| `src/index.ts` | Entry point dengan lazy loading |

**Kode tersedia di:** TOPIC_005/05_initial_files.md Section 1

---

### Step 2: Commands Layer

| File                     | Purpose            |
| :----------------------- | :----------------- |
| `src/commands/index.ts`  | Re-export commands |
| `src/commands/create.ts` | Create command     |
| `src/commands/list.ts`   | List command       |
| `src/commands/doctor.ts` | Doctor command     |

**Kode tersedia di:** TOPIC_005/05_initial_files.md Section 2

---

### Step 3: Core Layer - Base

| File                    | Purpose              |
| :---------------------- | :------------------- |
| `src/core/index.ts`     | Public exports       |
| `src/core/errors.ts`    | Custom error classes |
| `src/core/container.ts` | DI container         |
| `src/core/types.ts`     | Shared types         |

**Kode tersedia di:** TOPIC_005/05_initial_files.md Section 3

---

### Step 4: Core Layer - Domain

| File                             | Purpose            |
| :------------------------------- | :----------------- |
| `src/core/domain/framework.ts`   | Framework entity   |
| `src/core/domain/project.ts`     | Project entity     |
| `src/core/domain/environment.ts` | Environment entity |

**Kode tersedia di:** TOPIC_005/05_initial_files.md Section 7

---

### Step 5: UI Layer - Base

| File                  | Purpose           |
| :-------------------- | :---------------- |
| `src/ui/index.ts`     | Public exports    |
| `src/ui/colors.ts`    | Color definitions |
| `src/ui/gradients.ts` | Gradient effects  |
| `src/ui/text.ts`      | Text styling      |

**Kode tersedia di:** TOPIC_005/05_initial_files.md Section 4

---

### Step 6: UI Layer - Components

| File                | Purpose         |
| :------------------ | :-------------- |
| `src/ui/banner.ts`  | ASCII banner    |
| `src/ui/spinner.ts` | Loading spinner |
| `src/ui/prompts.ts` | Prompt wrappers |
| `src/ui/symbols.ts` | Unicode symbols |
| `src/ui/box.ts`     | Box drawing     |

**Kode tersedia di:** TOPIC_005/05_initial_files.md Section 4

---

### Step 7: Frameworks Layer

| File                      | Purpose            |
| :------------------------ | :----------------- |
| `src/frameworks/index.ts` | Framework registry |
| `src/frameworks/types.ts` | Framework types    |

**Kode tersedia di:** TOPIC_005/05_initial_files.md Section 5

---

### Step 8: Utils Layer

| File                      | Purpose                  |
| :------------------------ | :----------------------- |
| `src/utils/index.ts`      | Public exports           |
| `src/utils/executor.ts`   | Safe command execution   |
| `src/utils/filesystem.ts` | File operations          |
| `src/utils/safe-path.ts`  | Path validation          |
| `src/utils/safe-env.ts`   | Environment sanitization |
| `src/utils/logger.ts`     | Logging utilities        |

**Kode tersedia di:** TOPIC_005/05_initial_files.md Section 6

---

### Step 9: Build & Verify

```bash
# Run build
npm run build

# Verify dist created
ls -la dist/

# Test CLI works
node dist/index.js --version
node dist/index.js --help
```

---

## ✅ Kriteria Sukses

- [x] src/index.ts implemented
- [x] src/commands/\*.ts (4 files) implemented
- [x] src/core/errors.ts implemented
- [x] src/core/container.ts implemented
- [x] src/core/domain/framework.ts implemented
- [x] src/ui/colors.ts implemented
- [x] src/ui/banner.ts implemented
- [x] src/ui/spinner.ts implemented
- [x] src/frameworks/index.ts implemented
- [x] src/utils/executor.ts implemented
- [x] `npm run build` passes ✅
- [x] `node dist/index.js --version` outputs "1.0.0" ✅

---

## 📝 Notes untuk AI Implementor

1. **Copy dari source:** Semua kode tersedia di 05_initial_files.md
2. **Lazy loading:** Entry point minimal, lazy load heavy deps
3. **Security:** Utils/executor.ts uses spawn with shell:false
4. **Type safety:** All files use TypeScript strict mode

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_005_framework_configs**

---

## 🔗 Terkait

Topic: [TOPIC_005](../Topic/TOPIC_005_project_setup/_main.md)
Ref: [05_initial_files.md](../Topic/TOPIC_005_project_setup/05_initial_files.md)
