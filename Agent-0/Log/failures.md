# Failure Log

Log kegagalan tool/command untuk pembelajaran Agent.

---

## 📊 Statistik

| Tool          | Total Failures | Last Failure |
| ------------- | -------------- | ------------ |
| run_command   | 1              | 2025-12-25   |
| write_to_file | 8              | 2025-12-25   |

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
