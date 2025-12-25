# Business Logic — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Klarifikasi: ORBIT CLI Bukan API Server

> ⚠️ **PENTING:** ORBIT CLI adalah **Command Line Interface tool** — bukan web server atau API.

| Aspek          | ORBIT CLI                     | Web API             |
| :------------- | :---------------------------- | :------------------ |
| **Arsitektur** | CLI executable                | HTTP server         |
| **Input**      | Command line arguments, stdin | HTTP requests       |
| **Output**     | stdout, stderr, exit codes    | HTTP responses      |
| **State**      | Stateless per-invocation      | Session-based       |
| **Lifetime**   | Runs → completes → exits      | Long-running daemon |

**Konsekuensi:**

- Tidak ada REST endpoints
- Tidak ada database connection pooling
- Tidak ada authentication tokens
- Security fokus pada **command execution** dan **file system access**

---

## 2. Business Logic Flow

### 2.1 High-Level Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     ORBIT CLI Business Logic                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌───────────┐  │
│  │ Command │───▶│ Validate │───▶│ Execute  │───▶│ Report    │  │
│  │ Parse   │    │ & Check  │    │ Action   │    │ Result    │  │
│  └─────────┘    └──────────┘    └──────────┘    └───────────┘  │
│       │              │               │                │         │
│       ▼              ▼               ▼                ▼         │
│  commander.js   Environment      Framework       UI feedback    │
│  arguments      detector         installer                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Business Rules

| Rule ID | Rule                                                   | Implementation                    |
| :------ | :----------------------------------------------------- | :-------------------------------- |
| BR-001  | Framework harus tersedia di sistem                     | Environment check sebelum install |
| BR-002  | Project name harus valid npm package name              | Regex validation                  |
| BR-003  | Target directory tidak boleh exist                     | File system check                 |
| BR-004  | Stack preset harus compatible dengan framework version | Framework config validation       |
| BR-005  | Package manager harus tersedia                         | Which/where check                 |

---

## 3. Domain Entities

### 3.1 Framework Entity

```typescript
// src/core/domain/framework.ts

export interface Framework {
  readonly id: FrameworkId;
  readonly name: string;
  readonly description: string;
  readonly category: "nodejs" | "php";
  readonly website: string;
  readonly versions: readonly FrameworkVersion[];
  readonly stacks: readonly StackPreset[];
}

export interface FrameworkVersion {
  readonly version: string;
  readonly releaseDate: string;
  readonly isLTS: boolean;
  readonly isLatest: boolean;
  readonly minNodeVersion?: string;
  readonly minPhpVersion?: string;
}

export interface StackPreset {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly dependencies: readonly string[];
  readonly devDependencies: readonly string[];
  readonly scripts?: Record<string, string>;
  readonly config?: Record<string, unknown>;
}

export type FrameworkId =
  | "nextjs"
  | "nuxt"
  | "astro"
  | "sveltekit"
  | "vue"
  | "remix"
  | "laravel";
```

### 3.2 Project Configuration Entity

```typescript
// src/core/domain/project.ts

export interface ProjectConfig {
  readonly name: string;
  readonly directory: string;
  readonly framework: FrameworkId;
  readonly version: string;
  readonly packageManager: PackageManager;
  readonly stack: string;
  readonly options: ProjectOptions;
}

export interface ProjectOptions {
  readonly typescript: boolean;
  readonly eslint: boolean;
  readonly prettier: boolean;
  readonly git: boolean;
  readonly installDeps: boolean;
}

export type PackageManager = "npm" | "yarn" | "pnpm" | "bun";
```

### 3.3 Environment Entity

```typescript
// src/core/domain/environment.ts

export interface EnvironmentStatus {
  readonly tools: readonly ToolStatus[];
  readonly allMet: boolean;
  readonly missing: readonly string[];
}

export interface ToolStatus {
  readonly name: string;
  readonly version: string | null;
  readonly path: string | null;
  readonly isInstalled: boolean;
  readonly isRequired: boolean;
}

export interface SystemInfo {
  readonly os: "darwin" | "linux" | "win32";
  readonly arch: "x64" | "arm64";
  readonly nodeVersion: string;
  readonly cwd: string;
  readonly homedir: string;
}
```

---

## 4. Use Cases (Application Services)

### 4.1 CreateProjectUseCase

```typescript
// src/core/usecases/create-project.ts

export interface CreateProjectInput {
  name: string;
  framework: FrameworkId;
  version: string;
  packageManager: PackageManager;
  stack: string;
  options: Partial<ProjectOptions>;
}

export interface CreateProjectOutput {
  success: boolean;
  projectPath: string;
  nextSteps: string[];
  errors?: string[];
}

export class CreateProjectUseCase {
  constructor(
    private readonly envChecker: EnvironmentChecker,
    private readonly installer: FrameworkInstaller,
    private readonly configApplier: ConfigApplier,
    private readonly gitInitializer: GitInitializer
  ) {}

  async execute(input: CreateProjectInput): Promise<CreateProjectOutput> {
    // 1. Validate input
    this.validateInput(input);

    // 2. Check environment
    const envStatus = await this.envChecker.check(input.framework);
    if (!envStatus.allMet) {
      return this.failWithMissingRequirements(envStatus.missing);
    }

    // 3. Create project
    const projectPath = await this.installer.install(input);

    // 4. Apply stack configurations
    await this.configApplier.apply(projectPath, input.stack);

    // 5. Initialize git (optional)
    if (input.options.git !== false) {
      await this.gitInitializer.init(projectPath);
    }

    return {
      success: true,
      projectPath,
      nextSteps: this.generateNextSteps(input),
    };
  }

  private validateInput(input: CreateProjectInput): void {
    // Validation logic with Zod or custom validators
  }

  private failWithMissingRequirements(missing: string[]): CreateProjectOutput {
    return {
      success: false,
      projectPath: "",
      nextSteps: [],
      errors: missing.map((m) => `Missing requirement: ${m}`),
    };
  }

  private generateNextSteps(input: CreateProjectInput): string[] {
    return [`cd ${input.name}`, `${input.packageManager} run dev`];
  }
}
```

### 4.2 CheckEnvironmentUseCase

```typescript
// src/core/usecases/check-environment.ts

export class CheckEnvironmentUseCase {
  constructor(private readonly detector: ToolDetector) {}

  async execute(framework: FrameworkId): Promise<EnvironmentStatus> {
    const requiredTools = this.getRequiredTools(framework);
    const results = await Promise.all(
      requiredTools.map((tool) => this.detector.detect(tool))
    );

    return {
      tools: results,
      allMet: results.every((r) => r.isInstalled || !r.isRequired),
      missing: results
        .filter((r) => !r.isInstalled && r.isRequired)
        .map((r) => r.name),
    };
  }

  private getRequiredTools(framework: FrameworkId): string[] {
    const base = ["node", "npm"];

    switch (framework) {
      case "laravel":
        return [...base, "php", "composer"];
      default:
        return base;
    }
  }
}
```

---

## 5. Command Layer

### 5.1 Command Structure

```typescript
// src/commands/create.ts
import { Command } from "commander";
import { CreateProjectUseCase } from "../core/usecases/create-project";

export function createCommand(program: Command): void {
  program
    .command("create [name]")
    .description("Create a new project")
    .option("-t, --template <template>", "Framework template")
    .option("-p, --pm <manager>", "Package manager")
    .option("-y, --yes", "Skip prompts, use defaults")
    .action(async (name, options) => {
      // Lazy load heavy dependencies
      const { runCreateFlow } = await import("../flows/create-flow");
      await runCreateFlow(name, options);
    });
}
```

### 5.2 Entry Point

```typescript
// src/index.ts (MINIMAL - fast startup)
#!/usr/bin/env node

import { program } from 'commander';

program
  .name('orbit')
  .description('Universal Project Generator')
  .version('1.0.0');

// Register commands (lazy loaded internally)
import { createCommand } from './commands/create';
import { listCommand } from './commands/list';
import { doctorCommand } from './commands/doctor';

createCommand(program);
listCommand(program);
doctorCommand(program);

program.parse();
```

---

## 6. Dependency Injection Container

```typescript
// src/core/container.ts

import { CreateProjectUseCase } from "./usecases/create-project";
import { CheckEnvironmentUseCase } from "./usecases/check-environment";
import { ToolDetector } from "./services/tool-detector";
import { FrameworkInstaller } from "./services/framework-installer";
import { ConfigApplier } from "./services/config-applier";
import { GitInitializer } from "./services/git-initializer";

// Simple factory-based DI (no heavy DI framework needed for CLI)
export function createContainer() {
  // Infrastructure
  const toolDetector = new ToolDetector();
  const frameworkInstaller = new FrameworkInstaller();
  const configApplier = new ConfigApplier();
  const gitInitializer = new GitInitializer();

  // Use cases
  const checkEnvironment = new CheckEnvironmentUseCase(toolDetector);
  const createProject = new CreateProjectUseCase(
    checkEnvironment,
    frameworkInstaller,
    configApplier,
    gitInitializer
  );

  return {
    usecases: {
      createProject,
      checkEnvironment,
    },
    services: {
      toolDetector,
      frameworkInstaller,
      configApplier,
      gitInitializer,
    },
  };
}

export type Container = ReturnType<typeof createContainer>;
```

---

## 7. Error Handling Strategy

```typescript
// src/core/errors.ts

export abstract class OrbitError extends Error {
  abstract readonly code: string;
  abstract readonly exitCode: number;
  readonly isOperational = true;

  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends OrbitError {
  readonly code = "VALIDATION_ERROR";
  readonly exitCode = 1;
}

export class EnvironmentError extends OrbitError {
  readonly code = "ENVIRONMENT_ERROR";
  readonly exitCode = 2;
}

export class InstallationError extends OrbitError {
  readonly code = "INSTALLATION_ERROR";
  readonly exitCode = 3;
}

export class FileSystemError extends OrbitError {
  readonly code = "FILESYSTEM_ERROR";
  readonly exitCode = 4;
}

export class CommandExecutionError extends OrbitError {
  readonly code = "COMMAND_ERROR";
  readonly exitCode = 5;

  constructor(
    message: string,
    readonly command: string,
    readonly stderr?: string
  ) {
    super(message);
  }
}
```
