# Failure Log

Log kegagalan tool/command untuk pembelajaran Agent.

---

## 📊 Statistik

| Tool            | Total Failures | Last Failure |
| --------------- | -------------- | ------------ |
| run_command     | 2              | 2025-12-25   |
| write_to_file   | 9              | 2025-12-25   |
| Code Generation | 4              | 2025-12-25   |
| Config          | 1              | 2025-12-25   |

---

## 📝 Log Entries

### F-001 | write_to_file | 2025-12-25

- **Command/Params:** src/ui/gradients.ts - Export gradient variables
- **Error:** `Exported variable 'gradients' has or is using name 'GradientFunction' from external module but cannot be named.`
- **Context:** Creating UI gradients dengan `gradient-string` library
- **Workaround:** Definisikan explicit type alias `type GradientFn = { (text: string): string; multiline: (text: string) => string }` dan gunakan `Record<string, GradientFn>` untuk export object
- **Pattern ID:** P-001

---

### F-002 | write_to_file | 2025-12-25

- **Command/Params:** src/ui/prompts.ts - selectOption function
- **Error:** `Type '{ value: T; label: string; hint?: string | undefined; }[]' is not assignable to type 'Option<T>[]' with 'exactOptionalPropertyTypes: true'.`
- **Context:** Wrapping @clack/prompts select function
- **Workaround:** Gunakan `as any` cast dengan eslint-disable comment: `options: options as any`
- **Pattern ID:** P-001

---

### F-003 | run_command | 2025-12-25

- **Command/Params:** npm run build - tsup bundle with DTS
- **Error:** `Error parsing dist/index.js - Expected ident (double shebang)`
- **Context:** Shebang di source file + shebang di tsup banner = double shebang
- **Workaround:** Hapus shebang dari source file (`src/index.ts`), biarkan tsup yang menambahkan via `banner: { js: '#!/usr/bin/env node' }`
- **Pattern ID:** P-002

---

### F-004 | write_to_file | 2025-12-25

- **Command/Params:** src/core/services/command-builder.ts - BuildResult interface
- **Error:** `Type '{ postInstall: ... | undefined }' is not assignable to type 'BuildResult' with 'exactOptionalPropertyTypes: true'.`
- **Context:** Optional property dengan conditional value
- **Workaround:** Tambahkan `| undefined` pada interface: `postInstall?: { ... }[] | undefined`
- **Pattern ID:** P-001

---

### F-005 | write_to_file | 2025-12-25

- **Command/Params:** src/core/services/tool-detector.ts - version property
- **Error:** `Type 'string | undefined' is not assignable to type 'string' with exactOptionalPropertyTypes.`
- **Context:** Interface property dengan optional value
- **Workaround:** Ubah interface dari `version?: string` menjadi `version: string | undefined`
- **Pattern ID:** P-001

---

### F-006 | write_to_file | 2025-12-25

- **Command/Params:** src/ui/symbols.ts - symbols object
- **Error:** `An object literal cannot have multiple properties with the same name.`
- **Context:** Membuat object `symbols` dengan property `cross` (emoji ✖) dan `cross` (box drawing ┼)
- **Workaround:** Gunakan nama unik untuk setiap property. Rename `cross` (box) menjadi `crossBox`
- **Pattern ID:** P-003

---

### F-007 | write_to_file | 2025-12-25

- **Command/Params:** src/commands/helpers/check-tools.ts - ToolCheckResult interface
- **Error:** `Type '{ ok: boolean; version: string | undefined; }' is not assignable to type 'ToolCheckResult' with 'exactOptionalPropertyTypes: true'.`
- **Context:** Interface dengan `version?: string` tidak kompatibel saat return value `string | undefined`
- **Workaround:** Ubah interface ke `version?: string | undefined`
- **Pattern ID:** P-001

---

### F-008 | write_to_file | 2025-12-25

- **Command/Params:** src/commands/create.ts - validate function in p.text
- **Error:** `Not all code paths return a value.`
- **Context:** validate function yang menggunakan early return tanpa explicit return di akhir
- **Workaround:** Tambahkan `return undefined;` di akhir function body
- **Pattern ID:** P-004

---

### F-009 | write_to_file | 2025-12-25

- **Command/Params:** src/ui/prompts.ts - textInput optional params
- **Error:** `Argument of type '{ placeholder: string | undefined; ... }' is not assignable to parameter of type 'TextOptions' with 'exactOptionalPropertyTypes: true'.`
- **Context:** Passing optional params langsung ke library function
- **Workaround:** Build object secara dinamis, hanya assign property jika value !== undefined
- **Pattern ID:** P-001

---

### F-010 | run_command | 2025-12-25

- **Command/Params:** git commit -m "... TOPIC_003 (Frontend Design) COMPLETE!"
- **Error:** Shell menunggu closing quote karena parentheses `()` di dalam double quotes menyebabkan shell parsing issue
- **Context:** Commit message dengan parentheses dan tanda seru
- **Workaround:** Hindari `()` dan `!` dalam commit message, atau gunakan single quotes
- **Pattern ID:** P-005

---

### F-011 | write_to_file | 2025-12-25

- **Command/Params:** src/flows/create-flow.ts - displayError call
- **Error:** `Argument of type 'Error' is not assignable to parameter of type 'OrbitError'. Type 'Error' is missing the following properties from type 'OrbitError': code, title`
- **Context:** Passing generic Error ke displayError yang expects OrbitError interface
- **Workaround:** Gunakan p.log.error untuk generic errors, displayError hanya untuk OrbitError
- **Pattern ID:** P-006

---

### F-012 | Code Generation | 2025-12-25

- **Command/Params:** src/commands/create.ts - p.tasks() with placeholder TODOs
- **Error:** `await setTimeout(500); // Simulated delay` - Placeholder code shipped instead of real implementation
- **Context:** PLAN_016 - Commands layer had fake progress with simulated delays, while real implementation was in flows layer but NOT CONNECTED to entry point
- **Workaround:** Replace placeholder with actual flow calls: `import { runCreateFlow } from '../flows/create-flow.js'`
- **Pattern ID:** P-007
- **Severity:** 🔴 CRITICAL - Core functionality broken

---

### F-013 | Code Generation | 2025-12-25

- **Command/Params:** src/core/validation/schemas.ts, validate.ts
- **Error:** Empty files (0 bytes) left in codebase
- **Context:** Zod installed as dependency but these placeholder files were never implemented. Validation was done in utils/validation.ts instead
- **Workaround:** Delete empty files or implement Zod schemas. Deleted folder entirely.
- **Pattern ID:** P-008

---

### F-014 | Code Generation | 2025-12-25

- **Command/Params:** src/commands/doctor.ts - function signature
- **Error:** `@typescript-eslint/require-await: Async function 'runDoctor' has no 'await' expression`
- **Context:** Function declared as `async` but doesn't use await - spinner operations are synchronous
- **Workaround:** Remove `async` keyword: `export function runDoctor(): void`
- **Pattern ID:** P-009

---

### F-015 | Code Generation | 2025-12-25

- **Command/Params:** src/commands/helpers/check-tools.ts - regex usage
- **Error:** `@typescript-eslint/prefer-regexp-exec: Use the RegExp#exec() method instead`
- **Context:** Using `string.match(regex)` instead of `regex.exec(string)`
- **Workaround:** Change from `output.match(/v?(\d+\.\d+\.\d+)/)` to `const regex = /v?(\d+\.\d+\.\d+)/; regex.exec(output)`
- **Pattern ID:** P-010

---

### F-016 | Config | 2025-12-25

- **Command/Params:** tsconfig.json - exclude array
- **Error:** ESLint could not find test files in project: `The file was not found in any of the provided project(s)`
- **Context:** `"exclude": ["**/*.test.ts"]` prevented ESLint from parsing test files
- **Workaround:** Remove `**/*.test.ts` from exclude array
- **Pattern ID:** P-011

---

## 🔍 Identified Patterns

### P-001: exactOptionalPropertyTypes TypeScript Error

- **Description:** Dengan `exactOptionalPropertyTypes: true` di tsconfig, TypeScript sangat ketat tentang optional properties. `prop?: T` tidak sama dengan `prop?: T | undefined`.
- **Root Cause:** tsconfig.json menggunakan strict mode dengan `exactOptionalPropertyTypes: true`
- **Solution:**
  1. Untuk interface: gunakan `prop?: T | undefined` bukan `prop?: T`
  2. Untuk external library types yang tidak kompatibel: gunakan `as any` cast dengan eslint-disable
  3. Untuk export variable dari external module: definisikan explicit type alias
- **Affected Files:** Semua file yang menggunakan optional properties atau external library types
- **Created:** 2025-12-25

---

### P-004: Implicit Void Return in Conditional Functions

- **Description:** Function yang menggunakan early return dengan kondisi if-else tanpa explicit return di akhir akan dianggap tidak mengembalikan value di semua path.
- **Root Cause:** TypeScript strict mode memerlukan explicit return di semua code paths
- **Solution:**
  1. Selalu tambahkan `return undefined;` di akhir function yang butuh return value
  2. Untuk validate functions: pattern `if (error) return 'error msg'; return undefined;`
- **Affected Files:** Semua function dengan conditional returns
- **Created:** 2025-12-25

---

### P-002: Double Shebang Error

- **Description:** Jika source file (`.ts`) memiliki shebang DAN tsup config juga menambahkan shebang via `banner`, hasil build akan memiliki double shebang yang menyebabkan parse error.
- **Root Cause:** tsup `banner.js` option menambahkan shebang, source file juga sudah punya shebang
- **Solution:** Hapus shebang dari source file, biarkan bundler yang menambahkan shebang
- **Affected Files:** Entry point CLI (`src/index.ts`)
- **Created:** 2025-12-25

---

### P-003: Duplicate Object Property Name

- **Description:** Object literal tidak boleh memiliki property dengan nama yang sama. Saat membuat object dengan banyak properties, pastikan setiap nama unik.
- **Root Cause:** Copy-paste atau lupa bahwa nama sudah dipakai di bagian lain object
- **Solution:**
  1. Gunakan nama deskriptif yang spesifik (e.g., `crossBox` vs `cross`)
  2. Kelompokkan properties berdasarkan kategori dan review sebelum finalize
  3. Jika ada semantic overlap, tambahkan suffix (e.g., `Icon`, `Box`, `Char`)
- **Affected Files:** Object literal dengan banyak properties
- **Created:** 2025-12-25

---

### P-005: Shell Quoting Issue in Commit Messages

- **Description:** Karakter khusus seperti `()` dan `!` dalam commit message dapat menyebabkan shell parsing issue, terutama saat menggunakan double quotes.
- **Root Cause:** Shell (zsh/bash) menginterpretasi karakter khusus dalam double quotes
- **Solution:**
  1. Hindari `()` dan `!` dalam commit messages
  2. Gunakan format deskriptif tanpa karakter khusus: "TOPIC_003 Frontend Design COMPLETE"
  3. Atau gunakan single quotes untuk literal strings
- **Affected Files:** Semua git commit commands
- **Created:** 2025-12-25

---

### P-006: Error vs OrbitError Type Mismatch

- **Description:** `displayError()` expects `OrbitError` interface (dengan `code` dan `title`), bukan generic `Error`.
- **Root Cause:** Catch block menangkap generic Error, tapi try menggunakan function yang expects specific error type
- **Solution:**
  1. Gunakan `p.log.error(error.message)` untuk generic errors
  2. Gunakan `displayError()` hanya untuk `OrbitError` instances
  3. Type guard: `if (error instanceof OrbitBaseError) displayError(error)`
- **Affected Files:** Flow files yang catch generic errors
- **Created:** 2025-12-25

---

### P-007: Placeholder Code Left in Production (CRITICAL)

- **Description:** Placeholder code dengan `// TODO` comments dan simulated delays (`await setTimeout()`) dikirim ke production alih-alih implementasi nyata.
- **Root Cause:**
  1. AI membuat placeholder di satu layer (commands) tapi implementasi nyata di layer lain (flows)
  2. Entry point tidak dihubungkan dengan implementasi yang benar
  3. Tidak ada verification bahwa code actually works
- **Solution:**
  1. **SELALU** verify end-to-end bahwa entry point terhubung ke implementasi nyata
  2. **NEVER** leave `await setTimeout()` atau simulated delays di production code
  3. **ALWAYS** test core functionality manually sebelum consider "complete"
  4. Jika membuat placeholder, HARUS ada tracking mechanism untuk replacement
- **Prevention:** Run full integration test, not just build/typecheck
- **Created:** 2025-12-25

---

### P-008: Empty Placeholder Files Not Cleaned Up

- **Description:** File kosong dibuat sebagai placeholder untuk future implementation tapi tidak pernah di-implement atau di-delete.
- **Root Cause:**
  1. Folder structure dibuat di awal project dengan template files
  2. Requirements berubah dan file tidak dibutuhkan lagi
  3. Tidak ada audit untuk unused/empty files
- **Solution:**
  1. Jangan buat file kosong - buat saat ready untuk implement
  2. Jika ada unused dependencies (e.g., zod), evaluate apakah perlu di-remove
  3. Regular codebase audit untuk dead code
- **Created:** 2025-12-25

---

### P-009: Async Function Without Await

- **Description:** Function dideklarasikan sebagai `async` tapi tidak menggunakan `await` di dalamnya.
- **Root Cause:**
  1. Function awalnya async kemudian logic berubah
  2. Anticipated async operations yang tidak terjadi
  3. Copy-paste dari async function lain
- **Solution:**
  1. Jika tidak ada await, remove `async` keyword
  2. Jika return void, gunakan `function foo(): void` bukan `async function foo(): Promise<void>`
  3. ESLint rule `@typescript-eslint/require-await` menangkap ini
- **Created:** 2025-12-25

---

### P-010: String.match() vs RegExp.exec()

- **Description:** ESLint prefer `regex.exec(string)` daripada `string.match(regex)` untuk consistency dan performance.
- **Root Cause:** Developer habit - `.match()` lebih familiar
- **Solution:**

  ```typescript
  // Instead of:
  const match = output.match(/pattern/);

  // Use:
  const regex = /pattern/;
  const match = regex.exec(output);
  ```

- **Created:** 2025-12-25

---

### P-011: Test Files Excluded from TypeScript Project

- **Description:** tsconfig.json exclude array berisi `**/*.test.ts`, menyebabkan ESLint gagal parse test files.
- **Root Cause:** Mencegah test files dari build output tapi over-broad exclude
- **Solution:**
  1. Hapus `**/*.test.ts` dari exclude
  2. tsup/bundler biasanya sudah handle exclude test files dari bundle
  3. Atau buat separate tsconfig.test.json untuk tests
- **Created:** 2025-12-25

---

## 🧪 Test Cases

### TC-001 (Pattern: P-001)

- **Command:** `npm run typecheck`
- **Setup:** Buat interface dengan `prop?: string` dan assign `undefined`
- **Expected Result:** Type error jika pattern masih berlaku
- **Cleanup:** Revert file
- **Created:** 2025-12-25
- **Last Tested:** -

---

## 📦 Archived Patterns

> Pattern yang sudah tidak berlaku lagi (obsolete).

_Belum ada pattern yang diarsipkan._
