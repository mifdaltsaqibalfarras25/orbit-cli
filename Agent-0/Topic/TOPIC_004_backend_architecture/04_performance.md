# Performance Optimization — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Performance Targets

| Metric           | Target    | Reason                      |
| :--------------- | :-------- | :-------------------------- |
| **Cold Start**   | < 200ms   | First run harus responsif   |
| **Warm Start**   | < 100ms   | Subsequent runs lebih cepat |
| **Bundle Size**  | < 500KB   | npx install cepat           |
| **Dependencies** | < 10 prod | Minimize install time       |
| **Memory Peak**  | < 50MB    | Efficient resource usage    |

---

## 2. Lazy Loading Strategy

### 2.1 Entry Point (MINIMAL)

```typescript
// src/index.ts - KEEP THIS MINIMAL!
#!/usr/bin/env node

// Only import what's absolutely needed for parse
import { program } from 'commander';

program
  .name('orbit')
  .description('Universal Project Generator')
  .version('1.0.0');

// Commands are lazy loaded internally
program
  .command('create [name]')
  .description('Create a new project')
  .action(async (name, options) => {
    // LAZY LOAD: Only when command is invoked
    const { runCreate } = await import('./commands/create');
    await runCreate(name, options);
  });

program
  .command('list [framework]')
  .description('List available frameworks')
  .action(async (framework) => {
    const { runList } = await import('./commands/list');
    await runList(framework);
  });

program
  .command('doctor')
  .description('Check system requirements')
  .action(async () => {
    const { runDoctor } = await import('./commands/doctor');
    await runDoctor();
  });

program.parse();
```

### 2.2 Module Loading Pattern

```typescript
// ❌ BURUK - Loads everything immediately
import { showBanner } from "./ui/banner";
import { FrameworkRegistry } from "./frameworks";
import { ToolDetector } from "./core/services/tool-detector";
import { allFrameworks } from "./frameworks/all";

// ✅ BAIK - Lazy load when needed
async function runCreate(name: string) {
  // UI only loaded when we actually need it
  const { showBanner } = await import("./ui/banner");
  await showBanner();

  // Framework registry loaded on demand
  const { registry } = await import("./frameworks");
  const frameworks = await registry.getAll();
}
```

### 2.3 Module Map

| Module            | Load Time      | Strategy                       |
| :---------------- | :------------- | :----------------------------- |
| `commander`       | Startup        | Immediate (required for parse) |
| `ui/*`            | On command     | Lazy load                      |
| `core/services/*` | On command     | Lazy load                      |
| `frameworks/*`    | On select      | Lazy load per-framework        |
| `@clack/prompts`  | On interactive | Lazy load                      |
| `figlet`          | On banner      | Lazy load                      |
| `gradient-string` | On banner      | Lazy load                      |

---

## 3. Bundle Optimization (tsup)

### 3.1 tsup Configuration

```typescript
// tsup.config.ts

import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"], // ESM only (modern)
  target: "node18", // Target Node 18+
  outDir: "dist",
  clean: true,

  // Bundle all dependencies into single file
  noExternal: [
    "chalk",
    "gradient-string",
    "ora",
    // BUT NOT: @clack/prompts, figlet (large, lazy load)
  ],

  // Tree shaking
  treeshake: true,

  // Minify for smaller bundle
  minify: true,

  // Source maps for debugging
  sourcemap: true,

  // Split chunks for lazy loading
  splitting: true,

  // Define constants
  define: {
    "process.env.NODE_ENV": '"production"',
  },
});
```

### 3.2 Dependency Size Analysis

```bash
# Check bundle size
npx size-limit

# Analyze imports
npx source-map-explorer dist/index.js

# Find heavy dependencies
npx depcheck
npx bundlephobia [package-name]
```

### 3.3 Recommended Dependency Sizes

| Package           | Size    | Keep?   | Notes                  |
| :---------------- | :------ | :------ | :--------------------- |
| `commander`       | ~34KB   | ✅ Yes  | Essential, small       |
| `chalk`           | ~31KB   | ✅ Yes  | Essential, small       |
| `ora`             | ~12KB   | ✅ Yes  | Spinners, small        |
| `gradient-string` | ~8KB    | ✅ Yes  | Gradients, tiny        |
| `figlet`          | ~700KB  | ⚠️ Lazy | Large - lazy load only |
| `@clack/prompts`  | ~50KB   | ⚠️ Lazy | Medium - lazy load     |
| `zod`             | ~60KB   | ✅ Yes  | Validation, essential  |
| `enquirer`        | ~180KB  | ❌ No   | Too heavy, use @clack  |
| `inquirer`        | ~300KB+ | ❌ No   | Way too heavy          |

---

## 4. Async Best Practices

### 4.1 Parallel Operations

```typescript
// ❌ SEQUENTIAL - Slow
const nodeStatus = await detector.detect("node");
const npmStatus = await detector.detect("npm");
const gitStatus = await detector.detect("git");

// ✅ PARALLEL - Fast
const [nodeStatus, npmStatus, gitStatus] = await Promise.all([
  detector.detect("node"),
  detector.detect("npm"),
  detector.detect("git"),
]);

// ✅ EVEN BETTER - Promise.allSettled for non-critical
const results = await Promise.allSettled([
  detector.detect("node"),
  detector.detect("yarn"), // Optional, might fail
  detector.detect("pnpm"), // Optional, might fail
]);
```

### 4.2 Avoid Sync Operations

```typescript
// ❌ BLOCKING - Freezes CLI
import * as fs from "fs";
const config = JSON.parse(fs.readFileSync("config.json", "utf-8"));

// ✅ ASYNC - Non-blocking
import * as fs from "fs/promises";
const config = JSON.parse(await fs.readFile("config.json", "utf-8"));
```

### 4.3 Stream Large Files

```typescript
// ❌ MEMORY HOG - Loads entire file
const content = await fs.readFile("large-file.txt");

// ✅ STREAMING - Constant memory
import { createReadStream, createWriteStream } from "fs";
import { pipeline } from "stream/promises";

await pipeline(createReadStream("source.txt"), createWriteStream("dest.txt"));
```

---

## 5. Caching Strategies

### 5.1 In-Memory Cache for Session

```typescript
// src/utils/cache.ts

export class SessionCache<T> {
  private cache = new Map<string, T>();

  get(key: string): T | undefined {
    return this.cache.get(key);
  }

  set(key: string, value: T): void {
    this.cache.set(key, value);
  }

  getOrSet(key: string, factory: () => T): T {
    if (this.cache.has(key)) {
      return this.cache.get(key)!;
    }
    const value = factory();
    this.cache.set(key, value);
    return value;
  }

  async getOrSetAsync<T>(key: string, factory: () => Promise<T>): Promise<T> {
    if (this.cache.has(key)) {
      return this.cache.get(key) as T;
    }
    const value = await factory();
    this.cache.set(key, value as any);
    return value;
  }
}

// Usage
const toolCache = new SessionCache<ToolStatus>();
const nodeVersion = await toolCache.getOrSetAsync("node", () =>
  detector.detect("node")
);
```

### 5.2 Framework Registry Caching

```typescript
// src/frameworks/registry.ts

export class FrameworkRegistry {
  private cache = new Map<string, Framework>();
  private allLoaded = false;

  async get(id: string): Promise<Framework> {
    // Check cache first
    if (this.cache.has(id)) {
      return this.cache.get(id)!;
    }

    // Lazy load just this framework
    const framework = await this.loadFramework(id);
    this.cache.set(id, framework);
    return framework;
  }

  async getAll(): Promise<Framework[]> {
    if (this.allLoaded) {
      return Array.from(this.cache.values());
    }

    // Load all at once
    const frameworks = await Promise.all(
      Object.keys(loaders).map((id) => this.get(id))
    );
    this.allLoaded = true;
    return frameworks;
  }
}
```

---

## 6. Startup Time Optimization

### 6.1 Measure Startup Time

```typescript
// Measure registration to first action
const startTime = performance.now();

// ... CLI parsing ...

const parseTime = performance.now();
console.log(`Parse time: ${(parseTime - startTime).toFixed(2)}ms`);
```

### 6.2 Reduce Top-Level Imports

```typescript
// package.json - sideEffects declaration
{
  "sideEffects": false
}
```

### 6.3 NODE_COMPILE_CACHE (Node.js 22+)

```bash
# Enable bytecode caching for faster cold starts
export NODE_COMPILE_CACHE=~/.cache/node-compile

# Run ORBIT - subsequent runs will be faster
orbit create my-app
```

---

## 7. Performance Monitoring

### 7.1 Built-in Profiling

```typescript
// src/utils/profiler.ts

const perfMarks: Record<string, number> = {};

export function markStart(name: string): void {
  perfMarks[name] = performance.now();
}

export function markEnd(name: string): number {
  const start = perfMarks[name];
  if (!start) return 0;

  const duration = performance.now() - start;
  delete perfMarks[name];

  if (process.env.DEBUG) {
    console.log(`⏱️ ${name}: ${duration.toFixed(2)}ms`);
  }

  return duration;
}

// Usage
markStart("environment-check");
await checkEnvironment();
markEnd("environment-check");
// Output: ⏱️ environment-check: 45.23ms
```

### 7.2 Memory Usage Check

```typescript
export function logMemoryUsage(): void {
  if (!process.env.DEBUG) return;

  const usage = process.memoryUsage();
  console.log("📊 Memory:", {
    heap: `${Math.round(usage.heapUsed / 1024 / 1024)}MB`,
    rss: `${Math.round(usage.rss / 1024 / 1024)}MB`,
  });
}
```

---

## 8. Performance Checklist

### Before Release

- [ ] Entry point imports only `commander`
- [ ] Heavy modules are lazy loaded
- [ ] No sync file operations
- [ ] Bundle size < 500KB
- [ ] Cold start < 200ms measured
- [ ] Dependencies < 10 production
- [ ] Tree shaking enabled
- [ ] No unused exports
