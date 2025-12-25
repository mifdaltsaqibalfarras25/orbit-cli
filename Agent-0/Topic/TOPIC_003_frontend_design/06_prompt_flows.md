# Prompt Flows — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Prompt Library (@clack/prompts)

### Import Pattern

```typescript
// src/ui/prompts.ts

import * as p from "@clack/prompts";
import { colors, c } from "./colors.js";
import { setTimeout } from "node:timers/promises";

export { p }; // Re-export untuk akses langsung
```

---

## 2. Complete Create Flow

### 2.1 Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     ORBIT CREATE FLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Banner (if interactive)                                     │
│     ↓                                                           │
│  2. [name?] → Project Name                                      │
│     ↓                                                           │
│  3. Framework Selection (select)                                │
│     ↓                                                           │
│  4. Stack Preset Selection (select)                             │
│     ↓                                                           │
│  5. Package Manager Selection (select)                          │
│     ↓                                                           │
│  6. Confirmation (confirm)                                      │
│     ↓                                                           │
│  7. Installation Tasks (tasks + spinner)                        │
│     ↓                                                           │
│  8. Success Message (outro)                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Complete Implementation

```typescript
// src/commands/create.ts

import * as p from "@clack/prompts";
import { showBanner } from "../ui/banner.js";
import { registry } from "../frameworks/index.js";
import { colors } from "../ui/colors.js";

interface CreateOptions {
  template?: string;
  pm?: string;
  stack?: string;
  yes?: boolean;
}

export async function runCreate(
  projectName: string | undefined,
  options: CreateOptions
): Promise<void> {
  // Skip banner if non-interactive
  if (!options.yes) {
    await showBanner();
  }

  p.intro(colors.primary("Create a new project"));

  // ═══════════════════════════════════════════════════════════
  // 1. PROMPT GROUP - Collect all inputs
  // ═══════════════════════════════════════════════════════════

  const project = await p.group(
    {
      // --- Project Name ---
      name: () => {
        if (projectName) return Promise.resolve(projectName);
        return p.text({
          message: "What is your project name?",
          placeholder: "my-awesome-app",
          validate: (value) => {
            if (!value) return "Project name is required";
            if (!/^[a-z0-9-]+$/.test(value)) {
              return "Use lowercase letters, numbers, and dashes only";
            }
            if (value.length > 50) return "Name too long (max 50 chars)";
          },
        });
      },

      // --- Framework ---
      framework: async () => {
        if (options.template) return options.template;
        const frameworks = await registry.getAll();
        return p.select({
          message: "Which framework would you like to use?",
          options: frameworks.map((fw) => ({
            value: fw.id,
            label: fw.name,
            hint: fw.description,
          })),
        });
      },

      // --- Stack Preset ---
      stack: async ({ results }) => {
        if (options.stack) return options.stack;
        const fw = await registry.get(results.framework!);
        return p.select({
          message: "Choose your stack preset:",
          options: fw.stacks.map((s) => ({
            value: s.id,
            label: s.name,
            hint: s.description,
          })),
        });
      },

      // --- Package Manager ---
      packageManager: () => {
        if (options.pm) return Promise.resolve(options.pm);
        return p.select({
          message: "Which package manager?",
          initialValue: "npm",
          options: [
            { value: "npm", label: "npm", hint: "Default" },
            { value: "pnpm", label: "pnpm", hint: "Fast, efficient" },
            { value: "yarn", label: "yarn", hint: "Classic" },
            { value: "bun", label: "bun", hint: "Ultra fast" },
          ],
        });
      },

      // --- Confirmation ---
      confirm: ({ results }) => {
        if (options.yes) return Promise.resolve(true);
        return p.confirm({
          message: `Create "${results.name}" with ${results.framework} (${results.stack}) using ${results.packageManager}?`,
          initialValue: true,
        });
      },
    },
    {
      onCancel: () => {
        p.cancel("Operation cancelled.");
        process.exit(0);
      },
    }
  );

  // ═══════════════════════════════════════════════════════════
  // 2. VALIDATION - User declined
  // ═══════════════════════════════════════════════════════════

  if (!project.confirm) {
    p.cancel("Operation cancelled.");
    process.exit(0);
  }

  // ═══════════════════════════════════════════════════════════
  // 3. EXECUTION - Run installation tasks
  // ═══════════════════════════════════════════════════════════

  await p.tasks([
    {
      title: "Creating project directory",
      task: async () => {
        // mkdir logic
        await setTimeout(500); // Simulated delay
        return "Directory created";
      },
    },
    {
      title: `Installing ${project.framework}`,
      task: async () => {
        // Execute create command
        await setTimeout(2000); // Simulated delay
        return `${project.framework} installed`;
      },
    },
    {
      title: "Installing additional dependencies",
      task: async () => {
        // Post-install deps
        await setTimeout(1000);
        return "Dependencies installed";
      },
    },
    {
      title: "Initializing git",
      task: async () => {
        await setTimeout(300);
        return "Git initialized";
      },
    },
  ]);

  // ═══════════════════════════════════════════════════════════
  // 4. SUCCESS - Show completion message
  // ═══════════════════════════════════════════════════════════

  p.outro(`
${colors.success("✓")} Project created successfully!

${colors.dim("Next steps:")}
  cd ${project.name}
  ${project.packageManager} run dev

${colors.dim("Happy coding!")} 🚀
  `);
}
```

---

## 3. Doctor Command Flow

```typescript
// src/commands/doctor.ts

import * as p from "@clack/prompts";
import { colors, c } from "../ui/colors.js";

export async function runDoctor(): Promise<void> {
  p.intro(colors.primary("System Check"));

  const s = p.spinner();
  s.start("Checking environment...");

  // Check tools
  const checks = [
    { name: "Node.js", required: true, check: checkNode },
    { name: "npm", required: true, check: checkNpm },
    { name: "git", required: true, check: checkGit },
    { name: "pnpm", required: false, check: checkPnpm },
    { name: "yarn", required: false, check: checkYarn },
    { name: "bun", required: false, check: checkBun },
  ];

  const results: {
    name: string;
    ok: boolean;
    version?: string;
    required: boolean;
  }[] = [];

  for (const tool of checks) {
    const result = await tool.check();
    results.push({
      name: tool.name,
      ok: result.ok,
      version: result.version,
      required: tool.required,
    });
  }

  s.stop("Environment check complete");

  // Display results
  console.log();
  for (const r of results) {
    if (r.ok) {
      console.log(c.ok(`${r.name} v${r.version}`));
    } else if (r.required) {
      console.log(c.fail(`${r.name} (not found - REQUIRED)`));
    } else {
      console.log(c.warn(`${r.name} (not found - optional)`));
    }
  }
  console.log();

  const allRequiredMet = results.every((r) => !r.required || r.ok);

  if (allRequiredMet) {
    p.outro(colors.success("All requirements met! You're ready to go."));
  } else {
    p.outro(colors.error("Missing required tools. Please install them first."));
    process.exit(1);
  }
}
```

---

## 4. List Command Flow

```typescript
// src/commands/list.ts

import * as p from "@clack/prompts";
import { colors } from "../ui/colors.js";
import { registry } from "../frameworks/index.js";

export async function runList(frameworkId?: string): Promise<void> {
  p.intro(colors.primary("Available Frameworks"));

  if (frameworkId) {
    // Show specific framework details
    const fw = await registry.get(frameworkId);

    console.log();
    console.log(colors.text(`${fw.name}`));
    console.log(colors.dim(fw.description));
    console.log(colors.dim(`Website: ${fw.website}`));
    console.log();
    console.log(colors.info("Stack Presets:"));

    for (const stack of fw.stacks) {
      console.log(
        `  ${colors.primary(stack.name)} - ${colors.dim(stack.description)}`
      );
    }
  } else {
    // List all frameworks
    const frameworks = await registry.getAll();

    console.log();
    console.log(colors.dim("Node.js Frameworks:"));
    for (const fw of frameworks.filter((f) => f.category === "nodejs")) {
      console.log(
        `  ${colors.primary(fw.name.padEnd(12))} ${colors.dim(fw.description)}`
      );
    }

    console.log();
    console.log(colors.dim("PHP Frameworks:"));
    for (const fw of frameworks.filter((f) => f.category === "php")) {
      console.log(
        `  ${colors.primary(fw.name.padEnd(12))} ${colors.dim(fw.description)}`
      );
    }
  }

  console.log();
  p.outro(
    colors.dim(
      `Run 'orbit create <name> --template <framework>' to get started`
    )
  );
}
```

---

## 5. Prompt Patterns Reference

### 5.1 Text Input with Validation

```typescript
const name = await p.text({
  message: "Project name?",
  placeholder: "my-project",
  initialValue: "",
  validate: (value) => {
    if (!value) return "Required";
    if (value.length < 2) return "Too short";
    if (!/^[a-z0-9-]+$/.test(value)) return "Invalid characters";
  },
});
```

### 5.2 Select with Hints

```typescript
const choice = await p.select({
  message: "Choose option:",
  initialValue: "a",
  options: [
    { value: "a", label: "Option A", hint: "Default" },
    { value: "b", label: "Option B", hint: "Alternative" },
  ],
});
```

### 5.3 Multi-select

```typescript
const features = await p.multiselect({
  message: "Select features:",
  required: false,
  options: [
    { value: "eslint", label: "ESLint" },
    { value: "prettier", label: "Prettier" },
    { value: "husky", label: "Husky" },
  ],
});
```

### 5.4 Confirm

```typescript
const shouldContinue = await p.confirm({
  message: "Continue?",
  initialValue: true,
});
```

### 5.5 Spinner

```typescript
const s = p.spinner();
s.start("Loading...");
await doAsyncWork();
s.stop("Complete");
```

### 5.6 Tasks

```typescript
await p.tasks([
  {
    title: "Task 1",
    task: async () => {
      await work();
      return "Done";
    },
  },
]);
```

### 5.7 Cancel Handling

```typescript
if (p.isCancel(value)) {
  p.cancel("Cancelled.");
  process.exit(0);
}
```
