# Bug Fix: Create Command Placeholder Implementation

**ID:** PLAN_016 | **Status:** ✅ Done | **Prioritas:** 🔴 Tinggi  
**Dibuat:** 2025-12-25 | **Update:** 2025-12-25

---

## 🐛 Bug Report

### Problem Statement

File `src/commands/create.ts` berisi **placeholder implementation** dengan simulated delays, BUKAN implementasi nyata. Sementara itu, implementasi yang benar sudah ada di `src/flows/create-flow.ts` tetapi **TIDAK TERHUBUNG** ke entry point.

### Root Cause Analysis

**Bagaimana bug ini terjadi:**

1. Pada PLAN_007 (Frontend: Prompt Flows), `flows/create-flow.ts` dibuat dengan implementasi lengkap yang memanggil `container.usecases.createProject.execute()`
2. Pada PLAN_008 atau setelahnya, `commands/create.ts` dibuat sebagai alternatif UI dengan `p.tasks()` pattern
3. **Developer (AI) lupa menghubungkan flow yang benar** ke entry point
4. `index.ts` tetap memanggil `runCreate` (placeholder) bukan `runCreateFlow` (real implementation)

### Evidence

**File: `commands/create.ts` (Lines 152-185)**

```typescript
await p.tasks([
  {
    title: "Creating project directory",
    task: async () => {
      // TODO: mkdir logic                    ← PLACEHOLDER!
      await setTimeout(500); // Simulated delay ← FAKE!
      return "Directory created";
    },
  },
  // ... 3 more TODOs with simulated delays
]);
```

**File: `flows/create-flow.ts` (Lines 142-143)**

```typescript
// REAL implementation that should be used:
const result = await container.usecases.createProject.execute(input, reporter);
```

**File: `index.ts` (Lines 26-28)**

```typescript
// Currently calls the WRONG function:
const { runCreate } = await import("./commands/create.js");
await runCreate(name, options);
```

### Impact

- **Severity:** 🔴 Critical (Core functionality broken)
- `orbit create` command **TIDAK BENAR-BENAR MEMBUAT PROJECT**
- Users melihat progress bar palsu yang menampilkan "success" tanpa hasil nyata
- Kepercayaan user terhadap CLI rusak

---

## 🎯 Tujuan

1. Menghubungkan entry point ke implementasi yang benar (`flows/create-flow.ts`)
2. Mengikuti Clean Architecture: Commands → Flows → UseCases
3. Menghapus atau memperbaiki dead code placeholder
4. Mendokumentasikan pelajaran untuk AI di masa mendatang

---

## 🛠️ Strategi Implementasi

### Approach: Hybrid A + Refactor

Menggunakan `runCreateFlow` yang sudah ada, bukan mengimplementasi ulang TODOs.

### Step 1: Refactor `commands/create.ts`

#### [MODIFY] [create.ts](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/orbit-cli/src/commands/create.ts)

**Current (Broken):**

```typescript
export async function runCreate(projectName, options) {
  // ... prompts ...
  await p.tasks([
    // TODO: ... simulated delays
  ]);
}
```

**New (Fixed):**

```typescript
import { collectCreateInput, runCreateFlow } from "../flows/create-flow.js";

export async function runCreate(
  projectName: string | undefined,
  options: CreateOptions
): Promise<void> {
  try {
    if (!options.yes) {
      await showBanner();
    }
    p.intro(colors.primary("Create a new project"));

    // Collect input using flow (with validation)
    const input = await collectCreateInput();
    if (!input) {
      p.cancel("Operation cancelled.");
      process.exit(0);
    }

    // Execute using the real implementation
    const success = await runCreateFlow(input);

    if (!success) {
      process.exit(1);
    }
  } catch (error) {
    // ... error handling (keep existing) ...
  }
}
```

**Changes:**

- Remove `p.tasks()` with placeholder TODOs
- Import and use `collectCreateInput` and `runCreateFlow` from flows
- Keep error handling structure
- Keep banner and intro

### Step 2: Update Imports

#### [MODIFY] [create.ts](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/orbit-cli/src/commands/create.ts)

**Add import:**

```typescript
import { collectCreateInput, runCreateFlow } from "../flows/create-flow.js";
```

**Remove unused:**

```typescript
// Remove: import { setTimeout } from 'node:timers/promises';
// (No longer needed since we're not simulating delays)
```

### Step 3: Clean Up Dead Code

Remove lines 44-185 (the entire prompt group and tasks section that has TODO placeholders).

---

## 📁 Files to Modify

| File                     | Change                | Priority    |
| :----------------------- | :-------------------- | :---------- |
| `src/commands/create.ts` | Refactor to use flows | 🔴 Critical |

**No changes needed to:**

- `src/index.ts` (already imports from commands/create.ts)
- `src/flows/create-flow.ts` (already working correctly)
- `src/core/usecases/create-project.ts` (already implemented)

---

## ✅ Kriteria Sukses

### Build & Type Check

- [ ] `npm run build` berhasil tanpa error
- [ ] `npm run typecheck` berhasil tanpa error

### Functional Test (Manual)

**Test 1: Create Next.js Project**

```bash
cd /tmp
orbit create test-nextjs-project
# Expected:
# - Prompts appear for framework, package manager, etc.
# - REAL installation happens (npx create-next-app)
# - Project directory created with actual files
# - Success message with next steps
```

**Test 2: Verify Project Created**

```bash
ls -la /tmp/test-nextjs-project
# Expected: Real Next.js project files (package.json, app/, etc.)
```

**Test 3: Cancel Flow**

```bash
orbit create
# Press Ctrl+C during prompts
# Expected: Clean exit with "Operation cancelled"
```

### Code Quality

- [ ] No more `// TODO:` comments in create.ts
- [ ] No more `await setTimeout()` simulated delays
- [ ] grep "TODO" returns 0 results from create.ts

---

## 📚 Pelajaran untuk AI

> **PENTING:** Dokumentasi ini dibuat untuk mengingatkan AI di masa depan.

### ⚠️ Anti-Pattern yang Ditemukan

1. **Placeholder Code Left Behind**

   - Jangan pernah meninggalkan `// TODO` dengan simulated delays di production code
   - Jika membuat placeholder, HARUS ada tracking untuk mengimplementasi nanti

2. **Disconnect Between Components**

   - Selalu verifikasi bahwa komponen yang benar terhubung ke entry point
   - Test end-to-end, bukan hanya unit test

3. **Fake Progress Indicators**
   - `await setTimeout(500)` untuk simulasi progress adalah **ANTI-PATTERN**
   - Progress harus mencerminkan pekerjaan nyata

### ✅ Best Practices

1. **Follow Clean Architecture Flow**

   ```
   index.ts → commands/* → flows/* → usecases/* → services/*
   ```

2. **Verify Integration**

   - Setelah membuat flow baru, SELALU verifikasi terhubung ke entry point
   - Run manual test dari awal sampai akhir

3. **No Dead Code**
   - Jika ada dua implementasi untuk fungsi yang sama, pilih satu dan hapus yang lain
   - Jangan biarkan kode duplikat

---

## ⏱️ Estimated Time

| Task               | Time          |
| :----------------- | :------------ |
| Refactor create.ts | 30 menit      |
| Manual testing     | 15 menit      |
| Clean up           | 5 menit       |
| **Total**          | **~50 menit** |

---

## 🔗 Terkait

- **Topic:** [TOPIC_007 - Security & Bug Audit](file:///home/mifdlalalf2025/Documents/Proyek%202025/orbit/Agent-0/Topic/TOPIC_007_security_bug_audit/_main.md)
- **Finding:** Post-fix audit discovered placeholder TODOs
- **Related PLANs:** PLAN_007 (Frontend: Prompt Flows), PLAN_014 (Backend: Flow Integration)
