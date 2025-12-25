# Frontend Integration — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Architecture Separation

### 1.1 Layer Responsibilities

```
┌──────────────────────────────────────────────────────────────┐
│                      ORBIT CLI Architecture                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                    UI LAYER (Frontend)                 │  │
│  │  Owned by: Frontend Developer                         │  │
│  │  Contains: banner, colors, prompts, spinners, output  │  │
│  └─────────────────────────┬──────────────────────────────┘  │
│                            │                                 │
│                            │ Calls via interfaces            │
│                            ▼                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                  CORE LAYER (Backend)                  │  │
│  │  Owned by: Backend Developer                          │  │
│  │  Contains: usecases, services, domain, validation     │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 1.2 Dependency Rule

```
UI Layer ────────▶ Core Layer ────────▶ External World
             (depends on)           (interacts with)

❌ Core Layer TIDAK BOLEH depend pada UI Layer
✅ UI Layer BOLEH depend pada Core Layer (types, services)
```

---

## 2. Interface Contract

### 2.1 Core Exports for UI

```typescript
// src/core/index.ts - Public API for UI layer

// Types
export type {
  Framework,
  FrameworkId,
  FrameworkVersion,
} from "./domain/framework";
export type {
  ProjectConfig,
  PackageManager,
  ProjectOptions,
} from "./domain/project";
export type {
  EnvironmentStatus,
  ToolStatus,
  SystemInfo,
} from "./domain/environment";

// Use Cases
export { CreateProjectUseCase } from "./usecases/create-project";
export type {
  CreateProjectInput,
  CreateProjectOutput,
} from "./usecases/create-project";
export { CheckEnvironmentUseCase } from "./usecases/check-environment";

// Errors (for handling)
export { OrbitError, ValidationError, EnvironmentError } from "./errors";

// Container
export { createContainer, type Container } from "./container";

// Framework Registry
export { registry as frameworkRegistry } from "../frameworks";
```

### 2.2 UI Does NOT Export to Core

```typescript
// src/ui/index.ts - Private to UI layer

// These are INTERNAL utilities, not imported by core
export { colors, c } from "./colors";
export { gradients, nebula } from "./gradients";
export { text } from "./text";
export { showBanner } from "./banner";
export { createSpinner } from "./spinner";
```

---

## 3. Collaboration Patterns

### 3.1 Event-Like Communication

UI layer listens to core operations via callbacks:

```typescript
// Core defines progress interface
export interface ProgressReporter {
  onStart(message: string): void;
  onProgress(step: string): void;
  onComplete(message: string): void;
  onError(error: Error): void;
}

// Core use case accepts reporter
export class CreateProjectUseCase {
  async execute(
    input: CreateProjectInput,
    reporter?: ProgressReporter
  ): Promise<CreateProjectOutput> {
    reporter?.onStart("Creating project...");

    await this.createProject();
    reporter?.onProgress("Installing dependencies...");

    await this.installDeps();
    reporter?.onComplete("Project created!");

    return { success: true, projectPath: "..." };
  }
}
```

UI layer implements reporter:

```typescript
// src/ui/progress-reporter.ts
import { createSpinner } from "./spinner";
import type { ProgressReporter } from "../core";

export function createUIReporter(): ProgressReporter {
  let spinner: ReturnType<typeof createSpinner>;

  return {
    onStart(message) {
      spinner = createSpinner({ text: message });
      spinner.start();
    },
    onProgress(step) {
      spinner.text = step;
    },
    onComplete(message) {
      spinner.succeed(message);
    },
    onError(error) {
      spinner.fail(error.message);
    },
  };
}

// Usage in command flow
const reporter = createUIReporter();
await createProjectUseCase.execute(input, reporter);
```

### 3.2 Result Type for Error Handling

```typescript
// Core returns Result type (not throws for expected errors)
export type Result<T, E = OrbitError> =
  | { ok: true; value: T }
  | { ok: false; error: E };

// Use case returns Result
export class CreateProjectUseCase {
  async execute(input: CreateProjectInput): Promise<Result<ProjectOutput>> {
    const validation = this.validate(input);
    if (!validation.ok) {
      return validation; // Forward error
    }

    try {
      const project = await this.create(input);
      return { ok: true, value: project };
    } catch (error) {
      return { ok: false, error: error as OrbitError };
    }
  }
}

// UI handles Result
const result = await createProject.execute(input);

if (result.ok) {
  console.log(c.ok(`Project created at ${result.value.path}`));
} else {
  console.log(c.fail(result.error.message));
  if (result.error instanceof EnvironmentError) {
    showInstallInstructions(result.error);
  }
}
```

---

## 4. Shared Types Location

### 4.1 Type Ownership

| Type Category      | Location                | Owner    |
| :----------------- | :---------------------- | :------- |
| Domain entities    | `src/core/domain/`      | Backend  |
| Use case I/O       | `src/core/usecases/`    | Backend  |
| Validation schemas | `src/core/validation/`  | Backend  |
| UI component props | `src/ui/types.ts`       | Frontend |
| CLI options        | `src/commands/types.ts` | Frontend |

### 4.2 Import Rules

```typescript
// ✅ UI importing from Core (allowed)
import type { Framework, PackageManager } from "../core";

// ❌ Core importing from UI (FORBIDDEN)
import { showBanner } from "../ui/banner"; // NEVER DO THIS!

// ✅ Core importing from Core (allowed)
import { ValidationError } from "./errors";
import type { Framework } from "./domain/framework";
```

---

## 5. Flow Integration Example

### 5.1 Complete Create Flow

```typescript
// src/flows/create-flow.ts
// This file bridges UI and Core

import * as p from "@clack/prompts";
import { showBanner, c, text, spacer } from "../ui";
import {
  createContainer,
  frameworkRegistry,
  type CreateProjectInput,
} from "../core";
import { createUIReporter } from "../ui/progress-reporter";

export async function runCreateFlow(
  name: string | undefined,
  options: CLIOptions
): Promise<void> {
  const container = createContainer();

  // 1. UI: Show banner
  await showBanner();

  // 2. UI: Interactive prompts (if not --yes)
  const input = options.yes
    ? await getDefaultInput(name)
    : await promptForInput(name);

  if (!input) {
    p.cancel("Operation cancelled.");
    process.exit(0);
  }

  // 3. Core: Check environment
  const envResult = await container.usecases.checkEnvironment.execute(
    input.framework
  );

  if (!envResult.allMet) {
    // UI: Show missing requirements
    showMissingRequirements(envResult.missing);
    process.exit(2);
  }

  // 4. Core + UI: Create project with progress reporting
  const reporter = createUIReporter();
  const result = await container.usecases.createProject.execute(
    input,
    reporter
  );

  // 5. UI: Show result
  if (result.ok) {
    showSuccess(result.value);
  } else {
    showError(result.error);
    process.exit(result.error.exitCode);
  }
}

// Helper functions
async function promptForInput(
  initialName?: string
): Promise<CreateProjectInput | null> {
  // @clack/prompts integration
  const frameworks = await frameworkRegistry.getAll();

  const framework = await p.select({
    message: "Which framework?",
    options: frameworks.map((f) => ({
      value: f.id,
      label: f.name,
      hint: f.description,
    })),
  });

  if (p.isCancel(framework)) return null;

  // ... more prompts ...

  return {
    name: name as string,
    framework: framework as FrameworkId,
    version: version as string,
    packageManager: pm as PackageManager,
    stack: stack as string,
    options: { typescript: true, git: true },
  };
}

function showSuccess(output: ProjectOutput): void {
  spacer();
  console.log(c.ok("Project created successfully!"));
  spacer();
  console.log(text.title("Next steps:"));
  spacer();
  output.nextSteps.forEach((step) => {
    console.log(`  ${text.command(step)}`);
  });
  spacer();
}

function showError(error: OrbitError): void {
  spacer();
  console.log(c.fail(error.message));

  if (error.code === "ENVIRONMENT_ERROR") {
    // Show installation instructions
  }
  spacer();
}
```

---

## 6. Testing Strategy

### 6.1 Core Layer (Unit Tests)

```typescript
// __tests__/core/create-project.test.ts

describe("CreateProjectUseCase", () => {
  it("should create project with valid input", async () => {
    // Mock services
    const mockInstaller = { install: vi.fn().mockResolvedValue("/path") };
    const mockEnvChecker = {
      check: vi.fn().mockResolvedValue({ allMet: true }),
    };

    const usecase = new CreateProjectUseCase(
      mockEnvChecker,
      mockInstaller
      // ... other deps
    );

    const result = await usecase.execute({
      name: "test-app",
      framework: "nextjs",
      // ...
    });

    expect(result.ok).toBe(true);
    expect(mockInstaller.install).toHaveBeenCalled();
  });
});
```

### 6.2 UI Layer (Snapshot Tests)

```typescript
// __tests__/ui/banner.test.ts

describe("showBanner", () => {
  it("should output gradient banner", async () => {
    const output: string[] = [];
    vi.spyOn(console, "log").mockImplementation((msg) => output.push(msg));

    await showBanner();

    expect(output.join("\n")).toMatchSnapshot();
  });
});
```

### 6.3 Integration Tests

```typescript
// __tests__/integration/create-flow.test.ts

describe('Create Flow Integration', () => {
  it('should complete create flow end-to-end', async () => {
    // Actually run the flow with mocked child_process
    vi.mock('child_process', () => ({
      spawn: vi.fn().mockImplementation(() => /* mock stream */),
    }));

    await runCreateFlow('test-app', { yes: true });

    // Verify file system changes
    expect(fs.existsSync('/tmp/test-app')).toBe(true);
  });
});
```

---

## 7. Developer Workflow

### 7.1 For Frontend Developer

```bash
# Work on UI components
src/ui/

# Run visual tests
npm run test:ui

# Preview banner/colors
npm run dev:ui
```

### 7.2 For Backend Developer

```bash
# Work on core logic
src/core/
src/frameworks/

# Run unit tests
npm run test:core

# Test commands
npm run dev -- create test-app --yes
```

### 7.3 Integration Points

```bash
# Full integration test
npm run test:integration

# Build and test CLI
npm run build
./dist/index.js create my-app
```
