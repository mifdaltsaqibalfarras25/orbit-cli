# Backend: Flow Integration

**ID:** PLAN_014 | **Status:** ✅ Done | **Prioritas:** 🟡 Sedang
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_004/05_frontend_integration.md](../Topic/TOPIC_004_backend_architecture/05_frontend_integration.md)

---

## 🎯 Tujuan

Integrasi backend dengan frontend via flow files, menjembatani commands dengan usecases.

---

## 📁 Working Directory

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_009-013 selesai
- [x] Container tersedia
- [x] Utils tersedia

---

## 🛠️ Strategi Implementasi

### Step 1: Create Flows Directory

```bash
mkdir -p src/flows
```

---

### Step 2: Create Create Flow

Create `src/flows/create-flow.ts`:

- runCreateFlow function
- Integrates prompts with CreateProjectUseCase
- ProgressReporter untuk UI feedback

**Kode:** Section 2.1 dari 05_frontend_integration.md

---

### Step 3: Create Doctor Flow

Create `src/flows/doctor-flow.ts`:

- runDoctorFlow function
- Integrates with CheckEnvironmentUseCase
- Display tool status

**Kode:** Section 2.2 dari 05_frontend_integration.md

---

### Step 4: Create Flows Index

Create `src/flows/index.ts`:

- Re-export all flows

---

### Step 5: Build & Test

```bash
npm run build
```

---

## ✅ Kriteria Sukses

- [x] src/flows/create-flow.ts created
- [x] src/flows/doctor-flow.ts created
- [x] src/flows/index.ts created
- [x] Flows menggunakan Container untuk akses usecases ✅
- [x] `npm run build` passes ✅

---

## 📝 Notes

1. Flows adalah "glue" antara UI dan backend
2. ProgressReporter digunakan untuk update spinner
3. Error handling harus consistent dengan error-display

---

## ⏭️ Next Steps

Setelah **PLAN_014** selesai, **TOPIC_004 Backend Architecture COMPLETE!** 🎉

---

## 🔗 Terkait

Topic: [TOPIC_004](../Topic/TOPIC_004_backend_architecture/_main.md)
Ref: [05_frontend_integration.md](../Topic/TOPIC_004_backend_architecture/05_frontend_integration.md)
