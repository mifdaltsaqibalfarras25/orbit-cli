# Project Setup: Folder Structure

**ID:** PLAN_002 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_005/03_folder_structure.md](../Topic/TOPIC_005_project_setup/03_folder_structure.md)

---

## 🎯 Tujuan

Membuat skeleton folder structure lengkap untuk ORBIT CLI — mencakup semua direktori dan file kosong sesuai arsitektur yang sudah didefinisikan.

---

## 📁 Working Directory

> ⚠️ **PENTING:** Semua command harus dijalankan di folder berikut:

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_001 selesai (package.json, node_modules exist)

---

## 🛠️ Strategi Implementasi

### Step 1: Create Directory Structure

```bash
cd /home/mifdlalalf2025/Documents/Proyek\ 2025/orbit/orbit-cli

# Create all directories in one command
mkdir -p src/{commands,core/{domain,usecases,services,validation},ui,frameworks,utils,__tests__/{unit/{core,utils},integration/commands}}
```

**Validation:**

```bash
find src -type d | sort
```

Expected output: 14 directories

---

### Step 2: Create Entry Point Files

```bash
touch src/index.ts
touch src/cli.ts
```

**Validation:** `ls src/*.ts`

---

### Step 3: Create Commands Files

```bash
touch src/commands/{index,create,list,doctor}.ts
```

**Validation:** `ls src/commands/`

Expected: 4 files

---

### Step 4: Create Core Files

```bash
# Core root files
touch src/core/{index,errors,container,types}.ts

# Domain entities
touch src/core/domain/{framework,project,environment}.ts

# Use cases
touch src/core/usecases/{create-project,check-environment}.ts

# Services
touch src/core/services/{tool-detector,framework-installer,config-applier,git-initializer}.ts

# Validation
touch src/core/validation/{schemas,validate}.ts
```

**Validation:** `find src/core -type f -name "*.ts" | wc -l`

Expected: 15 files

---

### Step 5: Create UI Files

```bash
touch src/ui/{index,banner,colors,gradients,text,spinner,prompts,symbols,box}.ts
```

**Validation:** `ls src/ui/`

Expected: 9 files

---

### Step 6: Create Frameworks Files

```bash
touch src/frameworks/{index,types,nextjs,nuxt,astro,sveltekit,vue,remix,laravel}.ts
```

**Validation:** `ls src/frameworks/`

Expected: 9 files

---

### Step 7: Create Utils Files

```bash
touch src/utils/{index,executor,filesystem,safe-path,safe-env,logger}.ts
```

**Validation:** `ls src/utils/`

Expected: 6 files

---

### Step 8: Verify Complete Structure

```bash
# Count all TypeScript files
find src -type f -name "*.ts" | wc -l

# Visual tree
find src -type f -name "*.ts" | head -20
```

Expected: **45 TypeScript files** total

---

## ✅ Kriteria Sukses

- [x] 14 directories created under `src/` (actual: 16)
- [x] 2 entry point files (`index.ts`, `cli.ts`)
- [x] 4 command files
- [x] 15 core files (including domain, usecases, services, validation)
- [x] 9 UI files
- [x] 9 framework files
- [x] 6 utils files
- [x] Total: 45 TypeScript files ✅

---

## 📝 Notes untuk AI Implementor

1. **Empty files only:** Semua file kosong, kode diisi di PLAN_004
2. **Naming convention:** Gunakan kebab-case untuk filenames
3. **No content yet:** Jangan tulis kode apapun, hanya buat file kosong

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_003_configurations**

---

## 🔗 Terkait

Topic: [TOPIC_005](../Topic/TOPIC_005_project_setup/_main.md)
Ref: [03_folder_structure.md](../Topic/TOPIC_005_project_setup/03_folder_structure.md)
