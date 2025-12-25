# Universal Project Generator CLI

**ID:** TOPIC_001 | **Status:** 💬 Aktif | **Prioritas:** 🔴 Tinggi
**Dibuat:** 2024-12-24 | **Update:** 2024-12-24
**Tipe:** 📄 Simple

---

## Deskripsi

Aplikasi CLI (Command Line Interface) berbasis Node.js yang berfungsi sebagai **universal project generator** dengan antarmuka modern, estetik, dan interaktif—terinspirasi dari CLI Astro dan Claude Code.

### Fitur Utama

1. **Visual Welcome Banner**

   - ASCII Art dengan pewarnaan gradasi
   - Menu navigasi bersih tanpa perintah manual panjang

2. **Smart Compatibility Check**

   - Deteksi otomatis environment (PHP, Composer, Node.js, package managers)
   - Instruksi instalasi yang jelas jika tools tidak ditemukan

3. **Framework Support**

   - Laravel, Next.js, Astro, Nuxt, Svelte, dan lainnya
   - Pilihan versi spesifik (contoh: Laravel 10, 11, 12)

4. **Recommended Stack (Kurasi Dependensi)**

   - Opsi dependensi terkurasi berdasarkan standar industri
   - Selain opsi default bawaan framework

5. **Professional Installation UX**
   - Animasi spinner yang halus
   - Sembunyikan kompleksitas teknis
   - Pilihan package manager (NPM, Yarn, Bun, PNPM)

---

## Poin Penting

- Tujuan: Mempercepat setup awal + membantu developer memilih stack terbaik
- Target UX: Memanjakan mata, profesional, zero-friction
- Tech: Node.js CLI (inquirer/prompts, chalk, ora, figlet)

### [2024-12-24 22:33] Discussion: Architecture Breakdown

**Library Stack Final:**

| Layer         | Library                        | Alasan                                  |
| :------------ | :----------------------------- | :-------------------------------------- |
| Visual        | chalk, gradient-string, figlet | Industry standard untuk CLI styling     |
| Spinner       | ora                            | Elegant, by sindresorhus                |
| Prompts       | **@clack/prompts**             | Modern, lightweight (4KB), beautiful UX |
| CLI Framework | commander.js                   | Argument parsing, mature                |

**Keputusan Teknis:**

- ✅ TypeScript (maintainability)
- ✅ ESM (modern standard)
- ✅ @clack/prompts > Inquirer (lebih modern & ringan)
- ✅ tsup untuk bundling

📄 **Detail lengkap:** [RESEARCH_001_cli_architecture.md](../Research/RESEARCH_001_cli_architecture.md)

### [2024-12-24 23:21] Discussion: Git Commit Workflow

**Keputusan User tentang Git Commit:**

- ✅ Commit hanya saat user perintahkan (bukan auto-commit)
- ✅ Format: 1 line, inti detail yang mencakup semua perubahan
- ✅ Commit lokal saja, tidak push ke remote
- ✅ AI harus propose message dan minta approval dulu
- ✅ Jika user reject, user ganti message sendiri

**Flow:**

```
User: implementasi → AI: buat kode → User: review → User: "commit"
→ AI: propose message → User: approve → AI: git add + git commit
```

📄 **Prosedur lengkap:** [STANDARDS.md Section 17](../../.agent/STANDARDS.md)

---

## Terkait

Research: [RESEARCH_001](../Research/RESEARCH_001_cli_architecture.md) | Find: - | Plan: -
