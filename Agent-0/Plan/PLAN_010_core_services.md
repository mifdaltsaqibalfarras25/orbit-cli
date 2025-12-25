# Backend: Core Services

**ID:** PLAN_010 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_004/02_core_modules.md](../Topic/TOPIC_004_backend_architecture/02_core_modules.md)

---

## 🎯 Tujuan

Implementasi core services untuk ORBIT CLI: ToolDetector, FrameworkInstaller, ConfigApplier, dan GitInitializer.

---

## 📁 Working Directory

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_009 selesai (Domain Entities)
- [x] Domain types tersedia

---

## 🛠️ Strategi Implementasi

### Step 1: Create Services Directory

```bash
mkdir -p src/core/services
```

---

### Step 2: Create ToolDetector Service

Create `src/core/services/tool-detector.ts`:

- detect method untuk check tool
- findPath method using which/where
- getVersion method
- HARUS gunakan spawn, BUKAN exec

**Kode:** Section 2.1 dari 02_core_modules.md

---

### Step 3: Create FrameworkInstaller Service

Create `src/core/services/framework-installer.ts`:

- install method
- executeCommand dengan spawn
- getSafeEnv untuk environment sanitization

**Kode:** Section 2.2 dari 02_core_modules.md

---

### Step 4: Create ConfigApplier Service

Create `src/core/services/config-applier.ts`:

- apply method untuk stack configs
- installDeps method
- updatePackageJson method

**Kode:** Section 2.3 dari 02_core_modules.md

---

### Step 5: Create GitInitializer Service

Create `src/core/services/git-initializer.ts`:

- init method
- runGit helper dengan spawn

**Kode:** Section 2.4 dari 02_core_modules.md

---

### Step 6: Create Services Index

Create `src/core/services/index.ts`:

- Re-export all services

---

### Step 7: Build & Test

```bash
npm run build
```

---

## ✅ Kriteria Sukses

- [x] src/core/services/tool-detector.ts created
- [x] src/core/services/framework-installer.ts created
- [x] src/core/services/config-applier.ts created
- [x] src/core/services/git-initializer.ts created
- [x] src/core/services/index.ts created
- [x] Semua services menggunakan spawn, BUKAN exec ✅
- [x] `npm run build` passes ✅

---

## 📝 Notes

1. CRITICAL: Gunakan `spawn` dengan `shell: false` untuk security
2. Environment harus di-sanitize sebelum pass ke child process
3. Semua async operations harus return Promise

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_011** (Use Cases)

---

## 🔗 Terkait

Topic: [TOPIC_004](../Topic/TOPIC_004_backend_architecture/_main.md)
Ref: [02_core_modules.md](../Topic/TOPIC_004_backend_architecture/02_core_modules.md)
Security: [03_security.md](../Topic/TOPIC_004_backend_architecture/03_security.md)
