# Initial Files — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Entry Point

### src/index.ts

```typescript
#!/usr/bin/env node
/**
 * ORBIT CLI - Entry Point
 *
 * PENTING: File ini harus MINIMAL!
 * - Hanya import commander
 * - Semua logic di-lazy load
 */

import { program } from "commander";

// Package info
const VERSION = "1.0.0";
const NAME = "orbit";
const DESCRIPTION = "Universal Project Generator";

program.name(NAME).description(DESCRIPTION).version(VERSION);

// Register commands (lazy loaded internally)
program
  .command("create [name]")
  .description("Create a new project")
  .option("-t, --template <template>", "Framework template")
  .option("-p, --pm <manager>", "Package manager (npm|yarn|pnpm|bun)")
  .option("-s, --stack <stack>", "Stack preset (minimal|standard|full)")
  .option("-y, --yes", "Skip prompts, use defaults")
  .action(async (name, options) => {
    const { runCreate } = await import("./commands/create.js");
    await runCreate(name, options);
  });

program
  .command("list [framework]")
  .description("List available frameworks and versions")
  .action(async (framework) => {
    const { runList } = await import("./commands/list.js");
    await runList(framework);
  });

program
  .command("doctor")
  .description("Check system requirements")
  .action(async () => {
    const { runDoctor } = await import("./commands/doctor.js");
    await runDoctor();
  });

program.parse();
```

---

## 2. Commands

### src/commands/index.ts

```typescript
/**
 * Commands Index
 * Re-export command functions for use in CLI
 */

export { runCreate } from "./create.js";
export { runList } from "./list.js";
export { runDoctor } from "./doctor.js";
```

### src/commands/create.ts

```typescript
/**
 * Create Command
 * Main command for creating new projects
 */

import type { CreateCommandOptions } from './types.js';

export async function runCreate(
  name: string | undefined,
  options: CreateCommandOptions,
): Promise<void> {
  // Lazy load heavy dependencies
  const { showBanner } = await import('../ui/banner.js');
  const { createContainer } = await import('../core/container.js');
  const * as p from '@clack/prompts';

  // Show banner in interactive mode
  if (!options.yes) {
    await showBanner();
  }

  // TODO: Implement create flow
  console.log('Create command:', { name, options });
}
```

### src/commands/list.ts

```typescript
/**
 * List Command
 * Display available frameworks and versions
 */

export async function runList(framework?: string): Promise<void> {
  const { registry } = await import("../frameworks/index.js");
  const { c } = await import("../ui/colors.js");

  if (framework) {
    // Show specific framework details
    const fw = await registry.get(framework);
    console.log(c.ok(`${fw.name} versions:`));
    fw.versions.forEach((v) => {
      const label = v.isLatest ? " (latest)" : v.isLTS ? " (LTS)" : "";
      console.log(`  - ${v.version}${label}`);
    });
  } else {
    // List all frameworks
    const frameworks = await registry.getAll();
    console.log(c.info("Available frameworks:"));
    frameworks.forEach((fw) => {
      console.log(`  - ${fw.name}: ${fw.description}`);
    });
  }
}
```

### src/commands/doctor.ts

```typescript
/**
 * Doctor Command
 * Check system requirements and installed tools
 */

export async function runDoctor(): Promise<void> {
  const { c } = await import("../ui/colors.js");
  const { createSpinner } = await import("../ui/spinner.js");
  const { createContainer } = await import("../core/container.js");

  const container = createContainer();
  const spinner = createSpinner({ text: "Checking environment..." }).start();

  // Check all tools
  const result = await container.usecases.checkEnvironment.execute("_all");

  spinner.stop();

  console.log("\nSystem Check Results:\n");

  result.tools.forEach((tool) => {
    if (tool.isInstalled) {
      console.log(c.ok(`${tool.name} v${tool.version}`));
    } else if (tool.isRequired) {
      console.log(c.fail(`${tool.name} (not found)`));
    } else {
      console.log(c.warn(`${tool.name} (optional, not found)`));
    }
  });

  console.log();

  if (result.allMet) {
    console.log(c.ok("All requirements met!"));
  } else {
    console.log(c.fail(`Missing: ${result.missing.join(", ")}`));
    process.exit(1);
  }
}
```

---

## 3. Core Layer

### src/core/index.ts

```typescript
/**
 * Core Module - Public Exports
 * Only export what UI layer needs
 */

// Types
export type {
  Framework,
  FrameworkId,
  FrameworkVersion,
} from "./domain/framework.js";
export type {
  ProjectConfig,
  PackageManager,
  ProjectOptions,
} from "./domain/project.js";
export type { EnvironmentStatus, ToolStatus } from "./domain/environment.js";

// Use Cases
export { CreateProjectUseCase } from "./usecases/create-project.js";
export { CheckEnvironmentUseCase } from "./usecases/check-environment.js";

// Errors
export { OrbitError, ValidationError, EnvironmentError } from "./errors.js";

// Container
export { createContainer } from "./container.js";
```

### src/core/errors.ts

```typescript
/**
 * Custom Error Classes
 */

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

### src/core/container.ts

```typescript
/**
 * Dependency Injection Container
 * Simple factory-based DI
 */

import { ToolDetector } from "./services/tool-detector.js";
import { CheckEnvironmentUseCase } from "./usecases/check-environment.js";

export interface Container {
  services: {
    toolDetector: ToolDetector;
  };
  usecases: {
    checkEnvironment: CheckEnvironmentUseCase;
  };
}

export function createContainer(): Container {
  // Services
  const toolDetector = new ToolDetector();

  // Use Cases
  const checkEnvironment = new CheckEnvironmentUseCase(toolDetector);

  return {
    services: {
      toolDetector,
    },
    usecases: {
      checkEnvironment,
    },
  };
}
```

---

## 4. UI Layer

### src/ui/index.ts

```typescript
/**
 * UI Module - Public Exports
 */

export { colors, c } from "./colors.js";
export { gradients, nebula, cosmic } from "./gradients.js";
export { text } from "./text.js";
export { showBanner } from "./banner.js";
export { createSpinner } from "./spinner.js";
export { symbols } from "./symbols.js";
```

### src/ui/colors.ts

```typescript
/**
 * Color Definitions
 */

import chalk from "chalk";

export const colors = {
  // Status
  success: chalk.hex("#10B981"),
  error: chalk.hex("#EF4444"),
  warning: chalk.hex("#F59E0B"),
  info: chalk.hex("#6366F1"),

  // Brand
  primary: chalk.hex("#8B5CF6"),
  secondary: chalk.hex("#6366F1"),
  accent: chalk.hex("#22D3EE"),

  // Text
  text: chalk.hex("#FAFAFA"),
  dim: chalk.hex("#A1A1AA"),
  muted: chalk.hex("#71717A"),
};

// Semantic shortcuts
export const c = {
  ok: (t: string) => `${chalk.hex("#10B981")("✓")} ${t}`,
  fail: (t: string) => `${chalk.hex("#EF4444")("✗")} ${t}`,
  warn: (t: string) => `${chalk.hex("#F59E0B")("⚠")} ${t}`,
  info: (t: string) => `${chalk.hex("#6366F1")("ℹ")} ${t}`,
};
```

### src/ui/banner.ts

```typescript
/**
 * ASCII Banner Display
 * LAZY LOAD - Only when needed
 */

export async function showBanner(): Promise<void> {
  // Skip in CI or non-TTY
  if (!process.stdout.isTTY || process.env.CI || process.env.NO_BANNER) {
    return;
  }

  // Lazy load heavy dependencies
  const figlet = await import("figlet");
  const { nebula } = await import("./gradients.js");
  const { text } = await import("./text.js");

  const banner = figlet.default.textSync("ORBIT", {
    font: "ANSI Shadow",
  });

  console.log();
  console.log(nebula.multiline(banner));
  console.log();
  console.log(`  🚀 ${text.dim("Universal Project Generator  v1.0.0")}`);
  console.log();
}
```

### src/ui/spinner.ts

```typescript
/**
 * Spinner Wrapper
 */

import ora, { type Ora } from "ora";

export interface SpinnerOptions {
  text?: string;
}

export function createSpinner(options: SpinnerOptions = {}): Ora {
  return ora({
    text: options.text ?? "Loading...",
    color: "magenta",
    spinner: "dots",
  });
}
```

---

## 5. Frameworks

### src/frameworks/index.ts

```typescript
/**
 * Framework Registry
 * Lazy loading pattern for framework configs
 */

import type { Framework, FrameworkId } from "../core/domain/framework.js";

const loaders: Record<FrameworkId, () => Promise<Framework>> = {
  nextjs: () => import("./nextjs.js").then((m) => m.default),
  nuxt: () => import("./nuxt.js").then((m) => m.default),
  astro: () => import("./astro.js").then((m) => m.default),
  sveltekit: () => import("./sveltekit.js").then((m) => m.default),
  vue: () => import("./vue.js").then((m) => m.default),
  remix: () => import("./remix.js").then((m) => m.default),
  laravel: () => import("./laravel.js").then((m) => m.default),
};

class FrameworkRegistry {
  private cache = new Map<string, Framework>();

  getAvailableIds(): FrameworkId[] {
    return Object.keys(loaders) as FrameworkId[];
  }

  async get(id: FrameworkId): Promise<Framework> {
    if (this.cache.has(id)) {
      return this.cache.get(id)!;
    }

    const loader = loaders[id];
    if (!loader) {
      throw new Error(`Unknown framework: ${id}`);
    }

    const framework = await loader();
    this.cache.set(id, framework);
    return framework;
  }

  async getAll(): Promise<Framework[]> {
    return Promise.all(this.getAvailableIds().map((id) => this.get(id)));
  }
}

export const registry = new FrameworkRegistry();
```

---

## 6. Utils

### src/utils/executor.ts

```typescript
/**
 * Safe Command Executor
 * SECURITY: Always use spawn with shell: false
 */

import { spawn, type SpawnOptions } from "child_process";
import { CommandExecutionError } from "../core/errors.js";

export interface ExecResult {
  code: number;
  stdout: string;
  stderr: string;
}

export async function execute(
  command: string,
  args: readonly string[],
  options?: Partial<SpawnOptions>
): Promise<ExecResult> {
  return new Promise((resolve, reject) => {
    const child = spawn(command, [...args], {
      shell: false, // SECURITY: Never use shell!
      stdio: ["pipe", "pipe", "pipe"],
      ...options,
    });

    let stdout = "";
    let stderr = "";

    child.stdout?.on("data", (data) => {
      stdout += data.toString();
    });
    child.stderr?.on("data", (data) => {
      stderr += data.toString();
    });

    child.on("close", (code) => {
      resolve({ code: code ?? 1, stdout, stderr });
    });

    child.on("error", (err) => {
      reject(
        new CommandExecutionError(
          err.message,
          `${command} ${args.join(" ")}`,
          stderr
        )
      );
    });
  });
}
```

---

## 7. Domain Types

### src/core/domain/framework.ts

```typescript
/**
 * Framework Domain Entity
 */

export type FrameworkId =
  | "nextjs"
  | "nuxt"
  | "astro"
  | "sveltekit"
  | "vue"
  | "remix"
  | "laravel";

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
}

export interface StackPreset {
  readonly id: string;
  readonly name: string;
  readonly description: string;
  readonly dependencies: readonly string[];
  readonly devDependencies: readonly string[];
}
```

---

## File Implementation Checklist

- [x] `src/index.ts` — Entry point with lazy loading
- [x] `src/commands/*.ts` — Command implementations
- [x] `src/core/errors.ts` — Error classes
- [x] `src/core/container.ts` — DI container
- [x] `src/ui/colors.ts` — Color definitions
- [x] `src/ui/banner.ts` — Banner with lazy load
- [x] `src/ui/spinner.ts` — Spinner wrapper
- [x] `src/frameworks/index.ts` — Registry pattern
- [x] `src/utils/executor.ts` — Safe spawn
- [x] `src/core/domain/framework.ts` — Types
