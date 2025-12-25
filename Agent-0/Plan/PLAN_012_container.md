# Backend: DI Container

**ID:** PLAN_012 | **Status:** ✅ Done | **Prioritas:** 🟡 Sedang
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_004/01_business_logic.md](../Topic/TOPIC_004_backend_architecture/01_business_logic.md)

---

## 🎯 Tujuan

Implementasi Dependency Injection Container untuk wire-up semua services dan use cases.

---

## 📁 Working Directory

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_009 selesai (Domain Entities)
- [x] PLAN_010 selesai (Core Services)
- [x] PLAN_011 selesai (Use Cases)

---

## 🛠️ Strategi Implementasi

### Step 1: Create Container File

Create `src/core/container.ts`:

- createContainer function
- Wire all services and use cases
- Return typed container object

**Kode:** Section 6 dari 01_business_logic.md

---

### Step 2: Create Core Index

Create `src/core/index.ts`:

- Re-export container
- Re-export domain types
- Re-export errors

---

### Step 3: Build & Test

```bash
npm run build
```

---

## ✅ Kriteria Sukses

- [x] src/core/container.ts created
- [x] src/core/index.ts created
- [x] Container dapat instantiate semua services dan use cases ✅
- [x] `npm run build` passes ✅

---

## 📝 Notes

1. Gunakan factory-based DI (simple, no heavy framework)
2. Container harus lazy-loadable
3. Export Container type untuk type safety

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_013** (Validation and Security)

---

## 🔗 Terkait

Topic: [TOPIC_004](../Topic/TOPIC_004_backend_architecture/_main.md)
Ref: [01_business_logic.md](../Topic/TOPIC_004_backend_architecture/01_business_logic.md)
