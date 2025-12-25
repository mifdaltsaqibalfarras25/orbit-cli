# Frontend: Design System Components

**ID:** PLAN_006 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25
**Source:** [TOPIC_003/05_design_system.md](../Topic/TOPIC_003_frontend_design/05_design_system.md)

---

## 🎯 Tujuan

Meng-upgrade komponen UI yang sudah ada dengan fitur tambahan dari design system — termasuk theme, enhanced spinner, dan box utilities.

---

## 📁 Working Directory

```
/home/mifdlalalf2025/Documents/Proyek 2025/orbit/orbit-cli/
```

---

## 📋 Prerequisites

- [x] PLAN_001-005 selesai (Project setup complete)
- [x] src/ui/\*.ts files exist

---

## 🛠️ Strategi Implementasi

### Step 1: Create Theme Configuration

Create `src/ui/theme.ts`:

- Central theme config
- Design tokens

**Kode:** Section 3 - New file

---

### Step 2: Enhance colors.ts

Update `src/ui/colors.ts`:

- Add `c.step()` function untuk step counter
- Add modifiers (bold, underline)

**Kode:** Section 3 - colors.ts

---

### Step 3: Enhance gradients.ts

Update `src/ui/gradients.ts`:

- Add `supernova` (error gradient)
- Add `stellar` (info gradient)

**Kode:** Section 3 - gradients.ts

---

### Step 4: Enhance text.ts

Update `src/ui/text.ts`:

- Add `title()`, `body()`, `secondary()`
- Add `code()`, `path()`, `command()`, `link()`

**Kode:** Section 3 - text.ts

---

### Step 5: Enhance spinner.ts

Update `src/ui/spinner.ts`:

- Add `spinner.start()` convenience method
- Add `spinner.wrap()` for async operations

**Kode:** Section 3 - spinner.ts

---

### Step 6: Enhance box.ts

Update `src/ui/box.ts`:

- Add `spacer()` function
- Add `divider()` function
- Improve `box()` dengan title support

**Kode:** Section 3 - box.ts

---

### Step 7: Enhance symbols.ts

Update `src/ui/symbols.ts`:

- Add selection symbols (selected, unselected, pointer)
- Add progress symbols (active, pending)
- Add box drawing characters

**Kode:** Section 3 - symbols.ts

---

### Step 8: Update UI Index

Update `src/ui/index.ts`:

- Export new functions (spacer, divider, orbitTheme)
- Export new gradients

---

### Step 9: Build & Verify

```bash
npm run build
npm run typecheck
```

---

## ✅ Kriteria Sukses

- [x] src/ui/theme.ts created
- [x] src/ui/colors.ts enhanced with c.step()
- [x] src/ui/gradients.ts has supernova, stellar
- [x] src/ui/text.ts has title, code, command, link
- [x] src/ui/spinner.ts has wrap() method
- [x] src/ui/box.ts has spacer(), divider()
- [x] src/ui/symbols.ts complete with all symbols
- [x] src/ui/index.ts updated
- [x] `npm run build` passes ✅

---

## 📝 Notes

1. Banyak file sudah ada dari PLAN_004, plan ini hanya enhance
2. Fokus pada utility functions yang belum diimplementasi
3. Maintain backward compatibility

---

## ⏭️ Next Plan

Setelah plan ini selesai: **PLAN_007** (Prompt Flows)

---

## 🔗 Terkait

Topic: [TOPIC_003](../Topic/TOPIC_003_frontend_design/_main.md)
Ref: [05_design_system.md](../Topic/TOPIC_003_frontend_design/05_design_system.md)
