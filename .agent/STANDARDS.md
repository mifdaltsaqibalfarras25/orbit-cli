# Agent-0 Standards & Conventions

Dokumen ini adalah **Single Source of Truth** untuk semua konvensi penamaan, struktur file, prosedur operasional, dan **standar kualitas kode** di dalam Agent-0. Semua workflow **WAJIB** merujuk ke dokumen ini pada langkah pertama.

---

## 1. Konvensi Penamaan (Naming Convention)

### A. Format ID

Gunakan **Underscore (`_`)** sebagai pemisah antar komponen ID, bukan dash (`-`).

- **Topic:** `TOPIC_NNN` (Contoh: `TOPIC_001`)
- **Plan:** `PLAN_NNN` (Contoh: `PLAN_001`)
- **Finding:** `FIND_NNN` (Contoh: `FIND_001`)
- **Knowledge:** `K_NNN` (Contoh: `K_001`)

### B. Slug & Nama File

Format: `{ID}_{slug}.md`

- **Slug:** 2-4 kata, lowercase, dipisahkan underscore.
- **Contoh Benar:** `TOPIC_001_project_setup.md`

---

## 2. Struktur Folder & Lokasi

- **Topic:** `Agent-0/Topic/`
- **Plan:** `Agent-0/Plan/`
- **Finding:** `Agent-0/Find/`
- **Knowledge:** `Agent-0/Knowledge/{domain}/`
- **Log:** `Agent-0/Log/`
- **Standards:** `.agent/STANDARDS.md` (Dokumen ini)

---

## 3. Prinsip Dasar (HHH)

1. **Helpfulness (Kemanfaatan):** Berusaha maksimal memberikan solusi efektif.
2. **Harmlessness (Ketidakberbahayaan):** Tidak merusak sistem atau data.
3. **Honesty (Kejujuran):** Transparan tentang limitasi dan ketidakpastian.

---

## 4. Prosedur Update Index

Setiap kali membuat artefak baru, Agent **WAJIB** mengupdate file `index.md` di folder terkait.

---

## 5. Prosedur Logging

Setiap aksi signifikan dicatat di `Agent-0/Log/aktivitas.md`:

- Format: `| {HH:mm} | {Aktivitas} | {ID} | {Keterangan} |`

---

## 6. Self-Learning

Setiap kegagalan tool/command dicatat di `Agent-0/Log/failures.md` untuk pembelajaran.

---

## 7. Filosofi Agent-0

Agent-0 adalah **mesin berpikir eksternal** — perpanjangan pikiran yang belajar dari cara user bekerja, bukan user yang beradaptasi dengan Agent.

---

# 🧑‍💻 STANDAR KODE - SENIOR ENGINEER LEVEL

> **Filosofi:** Kode yang baik adalah kode yang bisa dipahami oleh engineer lain dalam 5 menit pertama tanpa penjelasan tambahan.

---

## 8. Prinsip Clean Code

### A. SOLID Principles (WAJIB)

1. **Single Responsibility Principle (SRP)**

   - Setiap class/function hanya punya SATU alasan untuk berubah
   - ❌ `UserService` yang handle create user, send email, dan logging
   - ✅ `UserRepository`, `EmailService`, `Logger` terpisah

2. **Open/Closed Principle (OCP)**

   - Terbuka untuk extension, tertutup untuk modification
   - Gunakan interfaces/abstract classes untuk extensibility
   - ✅ Tambah fitur baru dengan implement interface, bukan edit code existing

3. **Liskov Substitution Principle (LSP)**

   - Subtype harus bisa menggantikan parent tanpa break behavior
   - Derived class extend behavior, tidak mengubah kontrak

4. **Interface Segregation Principle (ISP)**

   - Interface kecil dan spesifik, bukan besar dan general
   - Client tidak dipaksa depend pada method yang tidak digunakan

5. **Dependency Inversion Principle (DIP)**
   - High-level module tidak depend pada low-level module
   - Keduanya depend pada abstractions (interfaces)
   - ✅ Inject dependencies, jangan hardcode concrete implementations

### B. TypeScript Best Practices

```typescript
// ✅ WAJIB: Strict mode
// tsconfig.json: "strict": true

// ✅ WAJIB: Explicit return types untuk public functions
export function calculateTotal(items: CartItem[]): number {}

// ❌ DILARANG: any type tanpa alasan kuat
function process(data: any) {} // BURUK

// ✅ WAJIB: Prefer unknown over any
function process(data: unknown) {
  if (isValidData(data)) {
    /* type guard */
  }
}

// ✅ WAJIB: Readonly untuk immutability
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}

// ✅ WAJIB: Exhaustive checks dengan never
type Status = "pending" | "active" | "done";
function handleStatus(status: Status): string {
  switch (status) {
    case "pending":
      return "Waiting";
    case "active":
      return "In progress";
    case "done":
      return "Complete";
    default:
      const _exhaustive: never = status;
      throw new Error(`Unknown status: ${_exhaustive}`);
  }
}

// ✅ WAJIB: Utility types untuk DRY
type UserUpdate = Partial<Pick<User, "name" | "email">>;

// ✅ WAJIB: Discriminated unions untuk state
type Result<T> = { success: true; data: T } | { success: false; error: Error };
```

### C. Naming Conventions

| Entity     | Convention              | Example                            |
| :--------- | :---------------------- | :--------------------------------- |
| Variables  | camelCase, deskriptif   | `userCount`, `isLoggedIn`          |
| Functions  | camelCase, verb prefix  | `getUserById`, `validateInput`     |
| Classes    | PascalCase, noun        | `UserService`, `OrderRepository`   |
| Interfaces | PascalCase, no I prefix | `User`, `Config` (bukan `IUser`)   |
| Types      | PascalCase              | `UserRole`, `ApiResponse`          |
| Constants  | SCREAMING_SNAKE_CASE    | `MAX_RETRIES`, `API_BASE_URL`      |
| Files      | kebab-case              | `user-service.ts`, `api-client.ts` |

### D. Function Guidelines

```typescript
// ✅ WAJIB: Single purpose
// ✅ WAJIB: Max 20 lines (ideal), max 50 lines (absolute max)
// ✅ WAJIB: Max 3 parameters, gunakan object untuk lebih
// ✅ WAJIB: Early return untuk guard clauses

// ❌ BURUK
function processUser(name, email, age, address, phone, company) {}

// ✅ BAIK
interface CreateUserParams {
  name: string;
  email: string;
  age?: number;
  address?: Address;
}

function createUser(params: CreateUserParams): User {
  // Guard clause - early return
  if (!params.name) {
    throw new ValidationError("Name is required");
  }

  // Main logic...
}
```

---

## 9. Error Handling Standards

### A. Error Classes (WAJIB untuk production)

```typescript
// Base error dengan context
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Specific errors
export class ValidationError extends AppError {
  constructor(message: string) {
    super(message, "VALIDATION_ERROR", 400);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, "NOT_FOUND", 404);
  }
}

export class UnauthorizedError extends AppError {
  constructor() {
    super("Unauthorized access", "UNAUTHORIZED", 401);
  }
}
```

### B. Error Handling Patterns

```typescript
// ✅ WAJIB: Never swallow errors
try {
  await dangerousOperation();
} catch (error) {
  // ❌ DILARANG: catch kosong
  // ❌ DILARANG: console.log(error) tanpa rethrow

  // ✅ WAJIB: Log + handle atau rethrow
  logger.error("Operation failed", { error, context });
  throw new AppError("Operation failed", "OP_FAILED");
}

// ✅ WAJIB: Result type untuk expected failures
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

async function fetchUser(id: string): Promise<Result<User>> {
  try {
    const user = await db.users.findById(id);
    if (!user) {
      return { ok: false, error: new NotFoundError("User") };
    }
    return { ok: true, value: user };
  } catch (error) {
    return { ok: false, error };
  }
}

// ✅ WAJIB: Informative error messages
// ❌ "Error occurred"
// ✅ "Failed to connect to database: connection refused on port 5432"
```

---

## 10. Security Standards

### A. Input Validation

```typescript
// ✅ WAJIB: Validate all external input
// ✅ WAJIB: Sanitize before use

// Command injection prevention
// ❌ BERBAHAYA
const cmd = `rm -rf ${userInput}`;
exec(cmd);

// ✅ AMAN: Use spawn with array
spawn("rm", ["-rf", sanitizedPath], { shell: false });

// Path traversal prevention
function validatePath(input: string): boolean {
  if (input.includes("..") || input.includes("/") || input.includes("\\")) {
    return false;
  }
  return /^[a-z0-9-_]+$/i.test(input);
}
```

### B. Secrets Management

```typescript
// ❌ DILARANG: Hardcode secrets
const apiKey = "sk-1234567890";

// ✅ WAJIB: Environment variables
const apiKey = process.env.API_KEY;
if (!apiKey) {
  throw new ConfigError("API_KEY environment variable is required");
}

// ✅ WAJIB: Don't log secrets
logger.info("Connecting with key", { key: apiKey }); // ❌
logger.info("Connecting with API key"); // ✅
```

---

## 11. Testing Standards

### A. Test Structure

```typescript
// ✅ WAJIB: AAA Pattern (Arrange, Act, Assert)
describe("UserService", () => {
  describe("createUser", () => {
    it("should create user with valid data", async () => {
      // Arrange
      const userData = { name: "John", email: "john@test.com" };

      // Act
      const result = await userService.createUser(userData);

      // Assert
      expect(result.name).toBe("John");
      expect(result.id).toBeDefined();
    });

    it("should throw ValidationError when name is empty", async () => {
      // Arrange
      const userData = { name: "", email: "john@test.com" };

      // Act & Assert
      await expect(userService.createUser(userData)).rejects.toThrow(
        ValidationError
      );
    });
  });
});
```

### B. Coverage Requirements

| Type              | Minimum Coverage |
| :---------------- | :--------------- |
| Unit Tests        | 80% lines        |
| Integration Tests | Critical paths   |
| Edge Cases        | All error paths  |

---

# 🎨 STANDAR UI/UX CLI - PROFESSIONAL LEVEL

> **Filosofi:** CLI yang baik membuat user merasa powerful, bukan frustasi.

---

## 12. CLI Visual Standards

### A. Banner & Branding

```typescript
// ✅ WAJIB: Welcome banner untuk interactive mode
// ✅ WAJIB: Gradient colors untuk visual appeal
// ✅ WAJIB: Skip banner jika: --quiet, piped output, CI environment

async function showBanner(): Promise<void> {
  // Skip di non-interactive environment
  if (!process.stdout.isTTY || process.env.CI) {
    return;
  }

  const banner = figlet.textSync("ORBIT", { font: "ANSI Shadow" });
  console.log(gradient.atlas.multiline(banner));
}
```

### B. Progress Feedback

```typescript
// ✅ WAJIB: Spinner untuk operasi > 500ms
// ✅ WAJIB: Progress bar untuk operasi dengan known length
// ✅ WAJIB: Update text saat status berubah

const spinner = ora("Creating project...").start();
spinner.text = "Installing dependencies...";
spinner.succeed("Project created successfully!");

// ❌ DILARANG: Silent operations (user stares at blinking cursor)
// ❌ DILARANG: Wall of text tanpa struktur
```

### C. Color Semantics

| Color       | Usage                 | Example                     |
| :---------- | :-------------------- | :-------------------------- |
| 🟢 Green    | Success, completion   | `✓ Done`                    |
| 🔴 Red      | Error, critical       | `✗ Failed`                  |
| 🟡 Yellow   | Warning, attention    | `⚠ Warning`                 |
| 🔵 Blue     | Info, neutral         | `ℹ Note`                    |
| ⚪ Gray/Dim | Secondary info, hints | `(press Enter to continue)` |

### D. Output Formatting

```typescript
// ✅ WAJIB: Structured output dengan visual hierarchy
console.log(); // Breathing room
console.log(chalk.bold("Created project successfully!"));
console.log();
console.log("  Next steps:");
console.log();
console.log(chalk.cyan("    cd my-project"));
console.log(chalk.cyan("    npm run dev"));
console.log();

// ✅ WAJIB: Machine-readable output option
// orbit list --json
// orbit list --plain (untuk grep compatibility)
```

---

## 13. CLI Interaction Standards

### A. Prompts

```typescript
// ✅ WAJIB: Clear question dengan sensible defaults
// ✅ WAJIB: Hints untuk opsi yang tidak jelas
// ✅ WAJIB: Escape hatch (Ctrl+C handling)

const framework = await p.select({
  message: "Which framework would you like to use?",
  options: [
    { value: "nextjs", label: "Next.js", hint: "React framework" },
    { value: "nuxt", label: "Nuxt", hint: "Vue framework" },
  ],
  initialValue: "nextjs", // Sensible default
});

// ✅ WAJIB: Graceful cancellation
if (p.isCancel(framework)) {
  p.cancel("Operation cancelled.");
  process.exit(0); // Exit code 0 untuk user-initiated cancel
}
```

### B. Error Messages

```typescript
// ❌ BURUK: Generic error
console.error("Error: Something went wrong");

// ✅ BAIK: Actionable error dengan suggestion
console.error();
console.error(chalk.red("✗ Node.js not found"));
console.error();
console.error("  Please install Node.js 18 or higher:");
console.error(chalk.dim("  https://nodejs.org/"));
console.error();
console.error("  After installing, run this command again.");
console.error();

// ✅ WAJIB: Did you mean? suggestions
console.error(chalk.yellow(`Unknown command: "${input}"`));
console.error(`Did you mean "${suggestCommand(input)}"?`);
```

### C. Help Text

```typescript
// ✅ WAJIB: Comprehensive help dengan examples
// ✅ WAJIB: Group related options
// ✅ WAJIB: Show most common flags first

/*
ORBIT CLI - Universal Project Generator

USAGE
  $ orbit <command> [options]

COMMANDS
  create [name]    Create a new project
  list             List available frameworks
  doctor           Check system requirements

OPTIONS
  -t, --template   Use specific template
  -p, --pm         Package manager (npm|yarn|pnpm|bun)
  -y, --yes        Skip prompts, use defaults
  -h, --help       Show help
  -v, --version    Show version

EXAMPLES
  $ orbit create my-app
  $ orbit create my-app -t nextjs-full
  $ orbit list nextjs
*/
```

---

# 🤖 STANDAR AI - ANTI-HALLUCINATION

> **Filosofi:** Lebih baik tidak menjawab daripada menjawab salah.

---

## 14. Anti-Hallucination Guidelines

### A. Verification Before Action

```markdown
✅ WAJIB sebelum generate code:

1. Cek apakah library/API yang dimention masih exist dan up-to-date
2. Jika ragu, gunakan search_web atau context7 untuk verify
3. Jangan generate code berdasarkan "ingatan" — selalu verify

✅ WAJIB sebelum file operations:

1. Gunakan view_file untuk lihat current state
2. Jangan assume content file — BACA DULU
3. Verify path exists sebelum edit
```

### B. Confidence Signaling

```markdown
✅ WAJIB: Akui ketidakpastian

- "Berdasarkan dokumentasi yang saya temukan..."
- "Saya tidak 100% yakin tentang X, sebaiknya verify..."
- "Ini berdasarkan pattern umum, mungkin perlu adjustment..."

❌ DILARANG: Overconfidence

- "Ini pasti benar..." (tanpa verification)
- Generate code untuk API tanpa check docs
```

### C. Source Citation

```markdown
✅ WAJIB: Cite source untuk factual claims

- "Menurut [Context7 docs]..."
- "Berdasarkan hasil search..."
- "Dari file [path] line [X]..."

✅ WAJIB: Distinguish antara:

- Fakta dari dokumentasi (verified)
- Best practice umum (pattern)
- Opini/suggestion (subjective)
```

### D. Code Generation Standards

```typescript
// ✅ WAJIB: Verify sebelum suggest library
// ❌ Jangan suggest package yang mungkin tidak exist
// ✅ Gunakan context7 resolve-library-id untuk verify

// ✅ WAJIB: Specific versions
// ❌ "Install the latest version"
// ✅ "Install version X.Y.Z (verified compatible)"

// ✅ WAJIB: Test commands sebelum suggest
// Verify command syntax dengan documentation
// Jangan assume flags tanpa verify
```

---

## 15. Quality Checklist

### Before Submitting Code (AI Self-Check)

- [ ] Apakah semua types explicit dan specific?
- [ ] Apakah error handling complete untuk semua failure cases?
- [ ] Apakah naming descriptive dan konsisten?
- [ ] Apakah functions single-purpose dan < 50 lines?
- [ ] Apakah ada magic numbers/strings yang harus di-constantify?
- [ ] Apakah semua external input di-validate?
- [ ] Apakah tests cover edge cases?
- [ ] Apakah code bisa dipahami tanpa komentar?

### Before Suggesting Libraries/APIs

- [ ] Apakah sudah verify library exist via search/context7?
- [ ] Apakah version yang disuggest compatible?
- [ ] Apakah API/syntax yang digunakan masih current?
- [ ] Apakah ada deprecation warnings?

---

## 16. File Organization Standards

```
src/
├── index.ts              # Entry point (minimal)
├── cli.ts                # CLI flow (lazy loaded)
│
├── core/                 # Business logic
│   ├── types.ts          # Shared types
│   ├── errors.ts         # Error classes
│   └── [feature]/        # Feature modules
│
├── ui/                   # User interface
│   ├── banner.ts
│   ├── prompts.ts
│   └── spinner.ts
│
├── utils/                # Utilities
│   ├── logger.ts
│   ├── validation.ts
│   └── system.ts
│
└── __tests__/            # Tests mirror src structure
    ├── core/
    └── utils/
```

---

## 17. Git Commit Standards

### A. Commit Message Format

```
<type>(<scope>): <subject>

Types:
- feat: New feature
- fix: Bug fix
- refactor: Code change that neither fixes nor adds
- docs: Documentation only
- style: Formatting, missing semicolons
- test: Adding tests
- chore: Maintenance

Examples:
feat(cli): add framework selection prompt with version picker
fix(detector): handle missing node_modules gracefully
refactor(installer): extract validation logic into separate module
```

### B. User-Triggered Commit Procedure

> ⚠️ **PENTING:** AI **TIDAK BOLEH** auto-commit. Commit hanya dilakukan saat **USER perintahkan**.

**Flow:**

```
1. User perintahkan implementasi
2. AI buat/edit files
3. User test/review (mungkin beberapa iterasi)
4. User puas → User perintahkan: "commit" / "simpan ke git"
5. AI propose commit message → minta approval
6. User approve → AI execute: git add + git commit
   User reject → User ganti message sendiri
```

**Aturan:**

- ❌ **DILARANG:** Commit otomatis setelah edit file
- ❌ **DILARANG:** Commit tanpa izin user
- ✅ **WAJIB:** Tunggu perintah eksplisit dari user
- ✅ **WAJIB:** Propose message, minta approval dulu
- ✅ **WAJIB:** Commit lokal saja (tidak push)

### C. Commit Message Guidelines

```
✅ BAIK: 1 line, mencakup inti detail

feat(ui): implement banner with ASCII art, gradient colors, and TTY detection
fix(prompts): handle user cancellation gracefully with proper exit codes
refactor(frameworks): split config into separate modules for lazy loading

❌ BURUK: Terlalu generic

feat: add feature
fix: fix bug
update: update code
```

### D. AI Commit Workflow

Saat user perintahkan commit:

```markdown
1. AI analyze perubahan yang dibuat sejak commit terakhir
2. AI generate commit message yang mencakup:
   - Type (feat/fix/refactor/etc)
   - Scope (module yang terkena)
   - Subject (inti perubahan dalam 1 line)
3. AI tampilkan ke user:
```

📝 Proposed commit message:

feat(installer): add dependency installation with spinner feedback

Approve commit? [Y/n]

````
4. User approve → AI execute:
```bash
git add .
git commit -m "feat(installer): add dependency installation with spinner feedback"
````

5. User reject → User inform message yang diinginkan

`````

---

## 18. Documentation Standards

````typescript
/**
 * Creates a new project with the specified configuration.
 *
 * @param config - Project configuration options
 * @returns Promise resolving to the created project path
 * @throws {ValidationError} When project name is invalid
 * @throws {EnvironmentError} When required tools are missing
 *
 * @example
 * ```typescript
 * const path = await createProject({
 *   name: 'my-app',
 *   framework: 'nextjs',
 *   stack: 'full'
 * });
 * ```
 */
async function createProject(config: ProjectConfig): Promise<string> {
  // ...
}
`````

---

**Last Updated:** 2024-12-24
**Version:** 2.0.0
