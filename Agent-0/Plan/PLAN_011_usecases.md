# Backend: Use Cases

**ID:** PLAN_011 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_004/01_business_logic.md](../Topic/TOPIC_004_backend_architecture/01_business_logic.md)

---

## 🎯 Tujuan

Implementasi application use cases: CreateProjectUseCase dan CheckEnvironmentUseCase.

---

## 📁 Working Directory

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_009 selesai (Domain Entities)
- [x] PLAN_010 selesai (Core Services)

---

## 🛠️ Strategi Implementasi

### Step 1: Create UseCases Directory

```bash
mkdir -p src/core/usecases
```

---

### Step 2: Create CheckEnvironmentUseCase

Create `src/core/usecases/check-environment.ts`:

- execute method
- getRequiredTools helper
- Return EnvironmentStatus

**Kode:** Section 4.2 dari 01_business_logic.md

---

### Step 3: Create CreateProjectUseCase

Create `src/core/usecases/create-project.ts`:

- execute method dengan steps:
  1. Validate input
  2. Check environment
  3. Install framework
  4. Apply stack config
  5. Init git
- Return ProjectResult

**Kode:** Section 4.1 dari 01_business_logic.md

---

### Step 4: Create UseCases Index

Create `src/core/usecases/index.ts`:

- Re-export all use cases

---

### Step 5: Build & Test

```bash
npm run build
```

---

## ✅ Kriteria Sukses

- [x] src/core/usecases/check-environment.ts created
- [x] src/core/usecases/create-project.ts created
- [x] src/core/usecases/index.ts created
- [x] Use cases menggunakan services dari PLAN_010 ✅
- [x] `npm run build` passes ✅

---

## 📝 Notes

1. Use cases adalah "orchestrators" — mereka mengkoordinasi services
2. Input dan output harus typed dengan domain entities
3. Error handling menggunakan custom error classes

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_012** (DI Container)

---

## 🔗 Terkait

Topic: [TOPIC_004](../Topic/TOPIC_004_backend_architecture/_main.md)
Ref: [01_business_logic.md](../Topic/TOPIC_004_backend_architecture/01_business_logic.md)
