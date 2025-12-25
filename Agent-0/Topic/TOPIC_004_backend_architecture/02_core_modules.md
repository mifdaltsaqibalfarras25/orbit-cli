# Core Modules — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Module Architecture

### 1.1 Folder Structure

```
src/
├── index.ts                 # Entry point (MINIMAL)
├── cli.ts                   # CLI orchestration (lazy loaded)
│
├── commands/                # Command definitions
│   ├── create.ts
│   ├── list.ts
│   └── doctor.ts
│
├── core/                    # Business logic (domain)
│   ├── domain/              # Entities & types
│   │   ├── framework.ts
│   │   ├── project.ts
│   │   └── environment.ts
│   │
│   ├── usecases/            # Application services
│   │   ├── create-project.ts
│   │   └── check-environment.ts
│   │
│   ├── services/            # Domain services
│   │   ├── tool-detector.ts
│   │   ├── framework-installer.ts
│   │   ├── config-applier.ts
│   │   └── git-initializer.ts
│   │
│   ├── errors.ts            # Custom error classes
│   ├── container.ts         # DI container
│   └── types.ts             # Shared types
│
├── frameworks/              # Framework-specific configs
│   ├── index.ts             # Registry
│   ├── nextjs.ts
│   ├── nuxt.ts
│   ├── astro.ts
│   ├── sveltekit.ts
│   ├── vue.ts
│   ├── remix.ts
│   └── laravel.ts
│
├── ui/                      # User interface (from TOPIC_003)
│   ├── index.ts
│   ├── banner.ts
│   ├── colors.ts
│   ├── prompts.ts
│   └── spinner.ts
│
├── utils/                   # Utilities
│   ├── executor.ts          # Command execution
│   ├── filesystem.ts        # File operations
│   ├── validation.ts        # Input validation
│   └── logger.ts            # Logging
│
└── __tests__/               # Tests
    ├── unit/
    └── integration/
```

---

## 2. Core Services

### 2.1 ToolDetector

Deteksi tool yang terinstall di sistem.

```typescript
// src/core/services/tool-detector.ts

import { spawn } from "child_process";
import { promisify } from "util";
import type { ToolStatus } from "../domain/environment";

export class ToolDetector {
  /**
   * Detect if a tool is installed and get its version
   */
  async detect(toolName: string): Promise<ToolStatus> {
    const path = await this.findPath(toolName);

    if (!path) {
      return {
        name: toolName,
        version: null,
        path: null,
        isInstalled: false,
        isRequired: true,
      };
    }

    const version = await this.getVersion(toolName);

    return {
      name: toolName,
      version,
      path,
      isInstalled: true,
      isRequired: true,
    };
  }

  /**
   * Find executable path using 'which' (unix) or 'where' (windows)
   */
  private async findPath(command: string): Promise<string | null> {
    const isWindows = process.platform === "win32";
    const finder = isWindows ? "where" : "which";

    try {
      const result = await this.exec(finder, [command]);
      return result.stdout.trim().split("\n")[0] || null;
    } catch {
      return null;
    }
  }

  /**
   * Get version by running '<tool> --version'
   */
  private async getVersion(tool: string): Promise<string | null> {
    try {
      const result = await this.exec(tool, ["--version"]);
      // Parse version from output (varies by tool)
      const match = result.stdout.match(/(\d+\.\d+\.\d+)/);
      return match?.[1] || null;
    } catch {
      return null;
    }
  }

  /**
   * Execute command safely using spawn (NOT exec!)
   */
  private exec(
    command: string,
    args: string[]
  ): Promise<{ stdout: string; stderr: string }> {
    return new Promise((resolve, reject) => {
      const child = spawn(command, args, {
        shell: false, // SECURITY: Never use shell
        stdio: ["pipe", "pipe", "pipe"],
      });

      let stdout = "";
      let stderr = "";

      child.stdout.on("data", (data) => {
        stdout += data;
      });
      child.stderr.on("data", (data) => {
        stderr += data;
      });

      child.on("close", (code) => {
        if (code === 0) {
          resolve({ stdout, stderr });
        } else {
          reject(new Error(`Command failed with code ${code}`));
        }
      });

      child.on("error", reject);
    });
  }
}
```

### 2.2 FrameworkInstaller

Install framework menggunakan package manager.

```typescript
// src/core/services/framework-installer.ts

import { spawn } from "child_process";
import type { CreateProjectInput } from "../usecases/create-project";
import { CommandExecutionError } from "../errors";

export class FrameworkInstaller {
  private readonly frameworks: Map<string, FrameworkConfig>;

  constructor() {
    this.frameworks = new Map();
    this.registerFrameworks();
  }

  /**
   * Install framework project
   */
  async install(input: CreateProjectInput): Promise<string> {
    const config = this.frameworks.get(input.framework);
    if (!config) {
      throw new Error(`Unknown framework: ${input.framework}`);
    }

    const { command, args } = config.getInstallCommand(input);

    await this.executeCommand(command, args, process.cwd());

    return path.join(process.cwd(), input.name);
  }

  /**
   * Execute installation command with spawn (SECURE)
   */
  private executeCommand(
    command: string,
    args: readonly string[],
    cwd: string
  ): Promise<void> {
    return new Promise((resolve, reject) => {
      const child = spawn(command, [...args], {
        cwd,
        shell: false, // SECURITY: Never use shell!
        stdio: ["inherit", "inherit", "pipe"],
        env: this.getSafeEnv(),
      });

      let stderr = "";
      child.stderr?.on("data", (data) => {
        stderr += data;
      });

      child.on("close", (code) => {
        if (code === 0) {
          resolve();
        } else {
          reject(
            new CommandExecutionError(
              `Installation failed with code ${code}`,
              `${command} ${args.join(" ")}`,
              stderr
            )
          );
        }
      });

      child.on("error", (err) => {
        reject(
          new CommandExecutionError(err.message, `${command} ${args.join(" ")}`)
        );
      });
    });
  }

  /**
   * Create safe environment without sensitive vars
   */
  private getSafeEnv(): NodeJS.ProcessEnv {
    const env = { ...process.env };
    // Remove potentially sensitive environment variables
    const sensitiveKeys = [
      "AWS_SECRET_ACCESS_KEY",
      "GITHUB_TOKEN",
      "NPM_TOKEN",
      "DATABASE_URL",
    ];
    sensitiveKeys.forEach((key) => delete env[key]);
    return env;
  }

  private registerFrameworks(): void {
    // Lazy load framework configs
    // This is populated at runtime
  }
}

interface FrameworkConfig {
  getInstallCommand(input: CreateProjectInput): {
    command: string;
    args: readonly string[];
  };
}
```

### 2.3 ConfigApplier

Apply stack configurations to project.

```typescript
// src/core/services/config-applier.ts

import * as fs from "fs/promises";
import * as path from "path";

export class ConfigApplier {
  /**
   * Apply stack preset configurations to project
   */
  async apply(projectPath: string, stackId: string): Promise<void> {
    const config = await this.loadStackConfig(stackId);

    // 1. Install additional dependencies
    if (config.dependencies.length > 0) {
      await this.installDeps(projectPath, config.dependencies, false);
    }

    if (config.devDependencies.length > 0) {
      await this.installDeps(projectPath, config.devDependencies, true);
    }

    // 2. Apply configuration files
    for (const [filename, content] of Object.entries(config.files || {})) {
      const filepath = path.join(projectPath, filename);
      await fs.writeFile(filepath, content, "utf-8");
    }

    // 3. Update package.json scripts
    if (config.scripts) {
      await this.updatePackageJson(projectPath, { scripts: config.scripts });
    }
  }

  private async installDeps(
    projectPath: string,
    deps: readonly string[],
    isDev: boolean
  ): Promise<void> {
    // Implementation uses spawn with package manager
  }

  private async loadStackConfig(stackId: string): Promise<StackConfig> {
    // Load from frameworks/[framework]/stacks/[stackId].ts
    return {} as StackConfig;
  }

  private async updatePackageJson(
    projectPath: string,
    updates: Record<string, unknown>
  ): Promise<void> {
    const pkgPath = path.join(projectPath, "package.json");
    const pkg = JSON.parse(await fs.readFile(pkgPath, "utf-8"));
    const updated = { ...pkg, ...updates };
    await fs.writeFile(pkgPath, JSON.stringify(updated, null, 2));
  }
}

interface StackConfig {
  dependencies: readonly string[];
  devDependencies: readonly string[];
  files?: Record<string, string>;
  scripts?: Record<string, string>;
}
```

### 2.4 GitInitializer

Initialize git repository.

```typescript
// src/core/services/git-initializer.ts

import { spawn } from "child_process";

export class GitInitializer {
  /**
   * Initialize git repository in project
   */
  async init(projectPath: string): Promise<void> {
    // 1. git init
    await this.runGit(projectPath, ["init"]);

    // 2. git add .
    await this.runGit(projectPath, ["add", "."]);

    // 3. git commit -m "Initial commit from ORBIT CLI"
    await this.runGit(projectPath, [
      "commit",
      "-m",
      "Initial commit from ORBIT CLI",
    ]);
  }

  private runGit(cwd: string, args: string[]): Promise<void> {
    return new Promise((resolve, reject) => {
      const child = spawn("git", args, {
        cwd,
        shell: false, // SECURITY!
        stdio: ["pipe", "pipe", "pipe"],
      });

      child.on("close", (code) => {
        if (code === 0) {
          resolve();
        } else {
          // Git failure is non-critical, just warn
          console.warn(`Git command failed: git ${args.join(" ")}`);
          resolve();
        }
      });

      child.on("error", () => {
        // Git not installed - non-critical
        resolve();
      });
    });
  }
}
```

---

## 3. Framework Registry

### 3.1 Registry Pattern

```typescript
// src/frameworks/index.ts

import type { Framework } from "../core/domain/framework";

// Lazy load framework configs
const frameworkLoaders: Record<string, () => Promise<Framework>> = {
  nextjs: () => import("./nextjs").then((m) => m.default),
  nuxt: () => import("./nuxt").then((m) => m.default),
  astro: () => import("./astro").then((m) => m.default),
  sveltekit: () => import("./sveltekit").then((m) => m.default),
  vue: () => import("./vue").then((m) => m.default),
  remix: () => import("./remix").then((m) => m.default),
  laravel: () => import("./laravel").then((m) => m.default),
};

export class FrameworkRegistry {
  private cache = new Map<string, Framework>();

  /**
   * Get all available framework IDs (no loading)
   */
  getAvailableIds(): string[] {
    return Object.keys(frameworkLoaders);
  }

  /**
   * Lazy load a specific framework config
   */
  async get(id: string): Promise<Framework> {
    if (this.cache.has(id)) {
      return this.cache.get(id)!;
    }

    const loader = frameworkLoaders[id];
    if (!loader) {
      throw new Error(`Unknown framework: ${id}`);
    }

    const framework = await loader();
    this.cache.set(id, framework);
    return framework;
  }

  /**
   * Get all frameworks (loads all configs)
   */
  async getAll(): Promise<Framework[]> {
    return Promise.all(Object.keys(frameworkLoaders).map((id) => this.get(id)));
  }
}

export const registry = new FrameworkRegistry();
```

### 3.2 Framework Config Example (Next.js)

```typescript
// src/frameworks/nextjs.ts

import type { Framework } from "../core/domain/framework";

const nextjs: Framework = {
  id: "nextjs",
  name: "Next.js",
  description: "The React Framework for the Web",
  category: "nodejs",
  website: "https://nextjs.org",

  versions: [
    {
      version: "15",
      releaseDate: "2024-10",
      isLTS: false,
      isLatest: true,
      minNodeVersion: "18.18",
    },
    {
      version: "14",
      releaseDate: "2023-10",
      isLTS: true,
      isLatest: false,
      minNodeVersion: "18.17",
    },
  ],

  stacks: [
    {
      id: "minimal",
      name: "Minimal",
      description: "TypeScript only",
      dependencies: [],
      devDependencies: [],
    },
    {
      id: "standard",
      name: "Standard",
      description: "TypeScript + ESLint + Prettier",
      dependencies: [],
      devDependencies: ["prettier", "eslint-config-prettier"],
    },
    {
      id: "full",
      name: "Full",
      description: "TypeScript + ESLint + Prettier + Tailwind + Husky",
      dependencies: [],
      devDependencies: [
        "prettier",
        "prettier-plugin-tailwindcss",
        "husky",
        "lint-staged",
      ],
      scripts: {
        prepare: "husky install",
      },
    },
  ],
};

export default nextjs;
```

---

## 4. Utility Modules

### 4.1 Safe Command Executor

```typescript
// src/utils/executor.ts

import { spawn, type SpawnOptions } from "child_process";
import { CommandExecutionError } from "../core/errors";

export interface ExecResult {
  code: number;
  stdout: string;
  stderr: string;
}

/**
 * Execute command safely using spawn
 * NEVER uses shell by default!
 */
export async function execute(
  command: string,
  args: readonly string[],
  options: Partial<SpawnOptions> = {}
): Promise<ExecResult> {
  return new Promise((resolve, reject) => {
    const child = spawn(command, [...args], {
      shell: false, // SECURITY: Default to no shell
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

/**
 * Execute and throw on non-zero exit
 */
export async function executeOrThrow(
  command: string,
  args: readonly string[],
  options?: Partial<SpawnOptions>
): Promise<ExecResult> {
  const result = await execute(command, args, options);

  if (result.code !== 0) {
    throw new CommandExecutionError(
      `Command exited with code ${result.code}`,
      `${command} ${args.join(" ")}`,
      result.stderr
    );
  }

  return result;
}
```

### 4.2 Input Validation

```typescript
// src/utils/validation.ts

import { z } from "zod";
import { ValidationError } from "../core/errors";

// Project name schema
export const projectNameSchema = z
  .string()
  .min(1, "Project name is required")
  .max(214, "Project name too long")
  .regex(
    /^(?:@[a-z0-9-*~][a-z0-9-*._~]*\/)?[a-z0-9-~][a-z0-9-._~]*$/,
    "Invalid project name. Must be a valid npm package name."
  );

// Path validation (prevent traversal)
export const safePathSchema = z
  .string()
  .refine(
    (val) => !val.includes("..") && !val.includes("\0"),
    "Invalid path: contains forbidden characters"
  );

// Framework ID schema
export const frameworkIdSchema = z.enum([
  "nextjs",
  "nuxt",
  "astro",
  "sveltekit",
  "vue",
  "remix",
  "laravel",
]);

// Package manager schema
export const packageManagerSchema = z.enum(["npm", "yarn", "pnpm", "bun"]);

/**
 * Validate input and throw ValidationError if invalid
 */
export function validate<T>(schema: z.ZodSchema<T>, data: unknown): T {
  const result = schema.safeParse(data);

  if (!result.success) {
    const message = result.error.errors
      .map((e) => `${e.path.join(".")}: ${e.message}`)
      .join(", ");
    throw new ValidationError(message);
  }

  return result.data;
}
```
