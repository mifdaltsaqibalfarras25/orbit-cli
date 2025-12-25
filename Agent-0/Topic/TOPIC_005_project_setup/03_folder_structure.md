# Folder Structure — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Complete Project Tree

```
orbit-cli/
│
├── 📁 src/                          # Source code
│   │
│   ├── index.ts                     # Entry point (MINIMAL)
│   ├── cli.ts                       # CLI orchestration (lazy loaded)
│   │
│   ├── 📁 commands/                 # Command definitions
│   │   ├── index.ts                 # Re-export all commands
│   │   ├── create.ts                # 'orbit create' command
│   │   ├── list.ts                  # 'orbit list' command
│   │   └── doctor.ts                # 'orbit doctor' command
│   │
│   ├── 📁 core/                     # Business logic layer
│   │   │
│   │   ├── index.ts                 # Public exports for core
│   │   ├── errors.ts                # Custom error classes
│   │   ├── container.ts             # Dependency injection
│   │   ├── types.ts                 # Shared types
│   │   │
│   │   ├── 📁 domain/               # Domain entities
│   │   │   ├── framework.ts         # Framework entity
│   │   │   ├── project.ts           # Project config entity
│   │   │   └── environment.ts       # Environment entity
│   │   │
│   │   ├── 📁 usecases/             # Application services
│   │   │   ├── create-project.ts    # CreateProject use case
│   │   │   └── check-environment.ts # CheckEnvironment use case
│   │   │
│   │   ├── 📁 services/             # Domain services
│   │   │   ├── tool-detector.ts     # Detect installed tools
│   │   │   ├── framework-installer.ts # Install frameworks
│   │   │   ├── config-applier.ts    # Apply stack configs
│   │   │   └── git-initializer.ts   # Initialize git
│   │   │
│   │   └── 📁 validation/           # Input validation
│   │       ├── schemas.ts           # Zod schemas
│   │       └── validate.ts          # Validation utilities
│   │
│   ├── 📁 ui/                       # User interface layer
│   │   ├── index.ts                 # Re-export UI utilities
│   │   ├── banner.ts                # ASCII banner display
│   │   ├── colors.ts                # Color definitions
│   │   ├── gradients.ts             # Gradient definitions
│   │   ├── text.ts                  # Text styling utilities
│   │   ├── spinner.ts               # Loading spinner wrapper
│   │   ├── prompts.ts               # Prompt wrappers
│   │   ├── symbols.ts               # Unicode symbols
│   │   └── box.ts                   # Box drawing utilities
│   │
│   ├── 📁 frameworks/               # Framework configurations
│   │   ├── index.ts                 # Framework registry
│   │   ├── types.ts                 # Framework types
│   │   ├── nextjs.ts                # Next.js config
│   │   ├── nuxt.ts                  # Nuxt config
│   │   ├── astro.ts                 # Astro config
│   │   ├── sveltekit.ts             # SvelteKit config
│   │   ├── vue.ts                   # Vue config
│   │   ├── remix.ts                 # Remix config
│   │   └── laravel.ts               # Laravel config
│   │
│   ├── 📁 utils/                    # Shared utilities
│   │   ├── index.ts                 # Re-export utilities
│   │   ├── executor.ts              # Safe command execution
│   │   ├── filesystem.ts            # File operations
│   │   ├── safe-path.ts             # Path validation
│   │   ├── safe-env.ts              # Environment sanitization
│   │   └── logger.ts                # Logging utilities
│   │
│   └── 📁 __tests__/                # Test files
│       ├── 📁 unit/                 # Unit tests
│       │   ├── core/
│       │   └── utils/
│       └── 📁 integration/          # Integration tests
│           └── commands/
│
├── 📁 dist/                         # Build output (gitignored)
│   ├── index.js
│   ├── index.js.map
│   └── index.d.ts
│
├── 📄 package.json                  # Package manifest
├── 📄 package-lock.json             # Lock file
├── 📄 tsconfig.json                 # TypeScript config
├── 📄 tsup.config.ts                # Bundler config
├── 📄 eslint.config.js              # ESLint config (flat)
├── 📄 .prettierrc                   # Prettier config
├── 📄 .gitignore                    # Git ignore rules
├── 📄 README.md                     # Project documentation
└── 📄 LICENSE                       # License file
```

---

## 2. File Purposes

### Entry Points

| File           | Purpose                     | Load Time |
| :------------- | :-------------------------- | :-------- |
| `src/index.ts` | CLI entry — MINIMAL imports | Immediate |
| `src/cli.ts`   | Full CLI orchestration      | Lazy      |

### Commands Layer

| File                 | Command                  | Description               |
| :------------------- | :----------------------- | :------------------------ |
| `commands/create.ts` | `orbit create [name]`    | Create new project        |
| `commands/list.ts`   | `orbit list [framework]` | List available frameworks |
| `commands/doctor.ts` | `orbit doctor`           | Check system requirements |

### Core Layer

| File                   | Responsibility                                    |
| :--------------------- | :------------------------------------------------ |
| `core/domain/*.ts`     | Domain entities (Framework, Project, Environment) |
| `core/usecases/*.ts`   | Application services (business logic)             |
| `core/services/*.ts`   | Infrastructure services (external interactions)   |
| `core/validation/*.ts` | Input validation with Zod                         |
| `core/errors.ts`       | Custom error classes                              |
| `core/container.ts`    | Dependency injection factory                      |

### UI Layer

| File              | Responsibility                 |
| :---------------- | :----------------------------- |
| `ui/banner.ts`    | ASCII art banner with gradient |
| `ui/colors.ts`    | Semantic color definitions     |
| `ui/gradients.ts` | Gradient text effects          |
| `ui/spinner.ts`   | Loading indicators             |
| `ui/prompts.ts`   | Interactive prompts wrapper    |

### Frameworks

| File                      | Framework     |
| :------------------------ | :------------ |
| `frameworks/nextjs.ts`    | Next.js 14/15 |
| `frameworks/nuxt.ts`      | Nuxt 3        |
| `frameworks/astro.ts`     | Astro 4       |
| `frameworks/sveltekit.ts` | SvelteKit 2   |
| `frameworks/vue.ts`       | Vue 3         |
| `frameworks/remix.ts`     | Remix 2       |
| `frameworks/laravel.ts`   | Laravel 11    |

### Utils

| File                  | Responsibility            |
| :-------------------- | :------------------------ |
| `utils/executor.ts`   | Safe spawn execution      |
| `utils/filesystem.ts` | File operations           |
| `utils/safe-path.ts`  | Path traversal prevention |
| `utils/safe-env.ts`   | Environment sanitization  |

---

## 3. Module Dependencies

```
                    ┌─────────────┐
                    │  index.ts   │  Entry point
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   cli.ts    │  Orchestration
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │  commands/  │ │     ui/     │ │    core/    │
    └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
           │               │               │
           │               │       ┌───────┴───────┐
           │               │       ▼               ▼
           │               │ ┌─────────────┐ ┌─────────────┐
           │               │ │   usecases/ │ │   services/ │
           │               │ └─────────────┘ └──────┬──────┘
           │               │                        │
           └───────────────┴──────────────┬─────────┘
                                          ▼
                                   ┌─────────────┐
                                   │    utils/   │
                                   └─────────────┘
```

**Dependency Rules:**

- ✅ Commands → Core, UI
- ✅ Core → Utils
- ✅ Services → Utils
- ❌ Core → UI (FORBIDDEN)
- ❌ Utils → Core (FORBIDDEN)

---

## 4. Create Structure Script

```bash
#!/bin/bash

# Create all directories
mkdir -p src/{commands,core/{domain,usecases,services,validation},ui,frameworks,utils,__tests__/{unit/{core,utils},integration/commands}}

# Create all files
touch src/index.ts
touch src/cli.ts

# Commands
touch src/commands/{index,create,list,doctor}.ts

# Core
touch src/core/{index,errors,container,types}.ts
touch src/core/domain/{framework,project,environment}.ts
touch src/core/usecases/{create-project,check-environment}.ts
touch src/core/services/{tool-detector,framework-installer,config-applier,git-initializer}.ts
touch src/core/validation/{schemas,validate}.ts

# UI
touch src/ui/{index,banner,colors,gradients,text,spinner,prompts,symbols,box}.ts

# Frameworks
touch src/frameworks/{index,types,nextjs,nuxt,astro,sveltekit,vue,remix,laravel}.ts

# Utils
touch src/utils/{index,executor,filesystem,safe-path,safe-env,logger}.ts

echo "✅ Folder structure created!"
```

---

## 5. File Count Summary

| Directory              | Files  | Purpose           |
| :--------------------- | :----- | :---------------- |
| `src/`                 | 2      | Entry points      |
| `src/commands/`        | 4      | CLI commands      |
| `src/core/`            | 4      | Core exports      |
| `src/core/domain/`     | 3      | Domain entities   |
| `src/core/usecases/`   | 2      | Use cases         |
| `src/core/services/`   | 4      | Services          |
| `src/core/validation/` | 2      | Validation        |
| `src/ui/`              | 9      | UI components     |
| `src/frameworks/`      | 9      | Framework configs |
| `src/utils/`           | 6      | Utilities         |
| **Total**              | **45** | TypeScript files  |
