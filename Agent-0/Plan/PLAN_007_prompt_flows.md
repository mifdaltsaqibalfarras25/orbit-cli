# Frontend: Prompt Flows

**ID:** PLAN_007 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_003/06_prompt_flows.md](../Topic/TOPIC_003_frontend_design/06_prompt_flows.md)

---

## 🎯 Tujuan

Implementasi lengkap prompt flows menggunakan @clack/prompts untuk ketiga command utama: `create`, `doctor`, dan `list`.

---

## 📁 Working Directory

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_001-006 selesai
- [x] @clack/prompts installed
- [x] UI components ready (colors, text, banner)

---

## 🛠️ Strategi Implementasi

### Step 1: Update prompts.ts

Update `src/ui/prompts.ts`:

- Add setTimeout import dari node:timers/promises
- Re-export p untuk akses langsung

**Kode:** Section 1

---

### Step 2: Create Helper Functions

Create `src/commands/helpers/check-tools.ts`:

- checkNode(), checkNpm(), checkGit()
- checkPnpm(), checkYarn(), checkBun()

**Kode:** Section 3 (for doctor command)

---

### Step 3: Implement Full Create Command

Rewrite `src/commands/create.ts`:

- p.group() untuk collect all inputs
- p.tasks() untuk installation steps
- p.outro() untuk success message
- Cancel handling

**Kode:** Section 2.2

---

### Step 4: Implement Full Doctor Command

Rewrite `src/commands/doctor.ts`:

- p.spinner() untuk loading
- Loop checks dengan colors
- Display pass/fail dengan c.ok/c.fail

**Kode:** Section 3

---

### Step 5: Implement Full List Command

Rewrite `src/commands/list.ts`:

- p.intro() dan p.outro()
- Group by category (nodejs/php)
- Show framework details jika ID provided

**Kode:** Section 4

---

### Step 6: Build & Test

```bash
npm run build
orbit list
orbit doctor
orbit create test-app
```

---

## ✅ Kriteria Sukses

- [x] src/ui/prompts.ts updated
- [x] src/commands/helpers/check-tools.ts created
- [x] src/commands/create.ts rewritten (full flow)
- [x] src/commands/doctor.ts rewritten (full flow)
- [x] src/commands/list.ts rewritten (full flow)
- [x] `npm run build` passes ✅
- [x] `orbit list` shows categorized frameworks ✅
- [x] `orbit doctor` shows tool checks with status ✅
- [x] `orbit create test` shows interactive prompts ✅

---

## 📝 Notes

1. Commands sudah ada dari PLAN_004, plan ini REPLACE dengan full implementation
2. Gunakan p.group() untuk collect inputs sekaligus
3. Handle cancel dengan p.isCancel()
4. Wireframes di 04_wireframes.md sebagai referensi visual

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_008** (Error Messages)

---

## 🔗 Terkait

Topic: [TOPIC_003](../Topic/TOPIC_003_frontend_design/_main.md)
Ref: [06_prompt_flows.md](../Topic/TOPIC_003_frontend_design/06_prompt_flows.md)
Wireframe: [04_wireframes.md](../Topic/TOPIC_003_frontend_design/04_wireframes.md)
