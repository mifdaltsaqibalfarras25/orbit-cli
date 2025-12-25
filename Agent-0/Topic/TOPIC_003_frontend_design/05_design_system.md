# Design System — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Complete UI Module Structure

```
src/ui/
├── index.ts          # Re-exports all UI components
├── theme.ts          # Theme configuration
├── colors.ts         # Color definitions
├── gradients.ts      # Gradient definitions
├── text.ts           # Typography utilities
├── banner.ts         # ASCII banner display
├── spinner.ts        # Loading spinner wrapper
├── prompts.ts        # Interactive prompts
├── box.ts            # Box drawing utilities
└── symbols.ts        # Unicode symbols
```

---

## 2. Core UI Exports

### Main Export File

```typescript
// src/ui/index.ts
export { colors, c } from "./colors";
export { gradients, nebula, cosmic, aurora } from "./gradients";
export { text } from "./text";
export { showBanner } from "./banner";
export { createSpinner } from "./spinner";
export { box, divider, spacer } from "./box";
export { symbols } from "./symbols";
export { orbitTheme } from "./theme";
```

---

## 3. Complete Implementation Files

### colors.ts

```typescript
// src/ui/colors.ts
import chalk from "chalk";

export const colors = {
  // Status colors
  success: chalk.hex("#10B981"),
  error: chalk.hex("#EF4444"),
  warning: chalk.hex("#F59E0B"),
  info: chalk.hex("#6366F1"),

  // Brand colors
  primary: chalk.hex("#8B5CF6"),
  secondary: chalk.hex("#6366F1"),
  accent: chalk.hex("#22D3EE"),

  // Text colors
  text: chalk.hex("#FAFAFA"),
  dim: chalk.hex("#A1A1AA"),
  muted: chalk.hex("#71717A"),

  // Modifiers
  bold: chalk.bold,
  underline: chalk.underline,
};

// Semantic shortcuts
export const c = {
  ok: (text: string) => `${chalk.hex("#10B981")("✓")} ${text}`,
  fail: (text: string) => `${chalk.hex("#EF4444")("✗")} ${text}`,
  warn: (text: string) => `${chalk.hex("#F59E0B")("⚠")} ${text}`,
  info: (text: string) => `${chalk.hex("#6366F1")("ℹ")} ${text}`,
  step: (n: number, total: number, text: string) =>
    `${chalk.dim(`[${n}/${total}]`)} ${text}`,
};
```

### gradients.ts

```typescript
// src/ui/gradients.ts
import gradient from "gradient-string";

// Primary brand gradient (nebula - multicolor)
export const nebula = gradient([
  "#8B5CF6", // Violet
  "#6366F1", // Indigo
  "#3B82F6", // Blue
  "#22D3EE", // Cyan
]);

// Simplified brand gradient (cosmic)
export const cosmic = gradient(["#8B5CF6", "#6366F1"]);

// Success gradient (aurora)
export const aurora = gradient(["#10B981", "#14B8A6", "#22D3EE"]);

// Error gradient (supernova)
export const supernova = gradient(["#EF4444", "#F97316"]);

// Info gradient (stellar)
export const stellar = gradient(["#6366F1", "#8B5CF6"]);

export const gradients = {
  nebula,
  cosmic,
  aurora,
  supernova,
  stellar,
};
```

### text.ts

```typescript
// src/ui/text.ts
import chalk from "chalk";

export const text = {
  // Hierarchy levels
  title: (s: string) => chalk.bold(s),
  body: (s: string) => s,
  secondary: (s: string) => chalk.hex("#A1A1AA")(s),
  dim: (s: string) => chalk.dim(s),
  hint: (s: string) => chalk.hex("#71717A")(s),

  // Inline styles
  bold: (s: string) => chalk.bold(s),
  code: (s: string) => chalk.hex("#22D3EE")(`\`${s}\``),
  path: (s: string) => chalk.underline.hex("#A1A1AA")(s),
  command: (s: string) => chalk.bold.hex("#22D3EE")(s),
  link: (s: string) => chalk.underline.hex("#6366F1")(s),
};
```

### banner.ts

```typescript
// src/ui/banner.ts
import figlet from "figlet";
import { nebula } from "./gradients";
import { text } from "./text";

const VERSION = "1.0.0";

export async function showBanner(): Promise<void> {
  // Skip in non-interactive environments
  if (!process.stdout.isTTY || process.env.CI || process.env.NO_BANNER) {
    return;
  }

  const banner = figlet.textSync("ORBIT", {
    font: "ANSI Shadow",
    horizontalLayout: "default",
  });

  console.log();
  console.log(nebula.multiline(banner));
  console.log();
  console.log(`  🚀 ${text.dim(`Universal Project Generator  v${VERSION}`)}`);
  console.log();
}
```

### spinner.ts

```typescript
// src/ui/spinner.ts
import ora, { Ora } from "ora";
import { colors } from "./colors";

export interface SpinnerOptions {
  text?: string;
}

export function createSpinner(options: SpinnerOptions = {}): Ora {
  return ora({
    text: options.text || "Loading...",
    color: "magenta",
    spinner: "dots",
  });
}

// Convenience methods
export const spinner = {
  start: (text: string) => createSpinner({ text }).start(),

  async wrap<T>(text: string, fn: () => Promise<T>): Promise<T> {
    const s = createSpinner({ text }).start();
    try {
      const result = await fn();
      s.succeed();
      return result;
    } catch (error) {
      s.fail();
      throw error;
    }
  },
};
```

### symbols.ts

```typescript
// src/ui/symbols.ts

export const symbols = {
  // Status
  success: "✓",
  error: "✗",
  warning: "⚠",
  info: "ℹ",

  // Selection
  selected: "◆",
  unselected: "○",
  pointer: "▸",

  // Progress
  active: "●",
  pending: "○",

  // Misc
  rocket: "🚀",
  package: "📦",
  lightning: "⚡",
  wrench: "🔧",
  bulb: "💡",

  // Box drawing
  topLeft: "┌",
  topRight: "┐",
  bottomLeft: "└",
  bottomRight: "┘",
  horizontal: "─",
  vertical: "│",
  teeRight: "├",
  teeLeft: "┤",
};
```

### box.ts

```typescript
// src/ui/box.ts
import chalk from "chalk";
import { symbols } from "./symbols";

export function spacer(): void {
  console.log();
}

export function divider(width = 50): void {
  console.log(chalk.dim(symbols.horizontal.repeat(width)));
}

export function box(lines: string[], options: { title?: string } = {}): void {
  const maxLen = Math.max(
    ...lines.map((l) => l.length),
    options.title?.length || 0
  );
  const width = maxLen + 4;

  const top = options.title
    ? `${symbols.topLeft}─ ${options.title} ${"─".repeat(
        width - options.title.length - 5
      )}${symbols.topRight}`
    : `${symbols.topLeft}${"─".repeat(width - 2)}${symbols.topRight}`;

  console.log(chalk.dim(top));
  for (const line of lines) {
    console.log(
      chalk.dim(symbols.vertical) +
        " " +
        line.padEnd(maxLen + 1) +
        chalk.dim(symbols.vertical)
    );
  }
  console.log(
    chalk.dim(
      `${symbols.bottomLeft}${"─".repeat(width - 2)}${symbols.bottomRight}`
    )
  );
}
```

---

## 4. Usage Examples

### Complete CLI Flow Example

```typescript
// src/cli.ts
import {
  showBanner,
  colors,
  c,
  text,
  spacer,
  divider,
  box,
  spinner,
} from "./ui";
import * as p from "@clack/prompts";

async function main() {
  // 1. Show banner
  await showBanner();

  // 2. Framework selection
  const framework = await p.select({
    message: "Which framework would you like to use?",
    options: [
      { value: "nextjs", label: "Next.js", hint: "React framework" },
      { value: "nuxt", label: "Nuxt", hint: "Vue framework" },
      { value: "astro", label: "Astro", hint: "Content-focused" },
    ],
  });

  if (p.isCancel(framework)) {
    p.cancel("Operation cancelled.");
    process.exit(0);
  }

  // 3. Project name
  const projectName = await p.text({
    message: "What is your project name?",
    placeholder: "my-app",
    validate: (value) => {
      if (!value) return "Name is required";
      if (!/^[a-z0-9-]+$/.test(value)) return "Invalid name format";
    },
  });

  if (p.isCancel(projectName)) {
    p.cancel("Operation cancelled.");
    process.exit(0);
  }

  // 4. Summary
  spacer();
  box([`Framework:  ${framework}`, `Project:    ${projectName}`], {
    title: "Configuration",
  });
  spacer();

  // 5. Installation
  const s = spinner.start("Creating project...");

  // Simulate work
  await new Promise((r) => setTimeout(r, 1000));
  s.text = "Installing dependencies...";

  await new Promise((r) => setTimeout(r, 1500));
  s.succeed("Project created successfully!");

  // 6. Next steps
  spacer();
  console.log(text.title("Next steps:"));
  spacer();
  console.log(`  ${text.command(`cd ${projectName}`)}`);
  console.log(`  ${text.command("npm run dev")}`);
  spacer();
  console.log(text.dim("Happy coding! 🚀"));
  spacer();
}

main().catch(console.error);
```

---

## 5. Design Tokens Summary

| Token               | Value          | Usage                    |
| :------------------ | :------------- | :----------------------- |
| `--color-primary`   | #8B5CF6        | Brand accent, highlights |
| `--color-secondary` | #6366F1        | Secondary accents        |
| `--color-accent`    | #22D3EE        | Commands, links          |
| `--color-success`   | #10B981        | Success states           |
| `--color-error`     | #EF4444        | Error states             |
| `--color-warning`   | #F59E0B        | Warning states           |
| `--color-text`      | #FAFAFA        | Primary text             |
| `--color-dim`       | #A1A1AA        | Secondary text           |
| `--color-muted`     | #71717A        | Hints, placeholders      |
| `--font-primary`    | JetBrains Mono | All terminal text        |
| `--spacing-xs`      | 1 char         | Tight spacing            |
| `--spacing-sm`      | 2 chars        | Default indent           |
| `--spacing-md`      | 4 chars        | Section indent           |
| `--spinner-type`    | dots           | Loading indicator        |
| `--border-char`     | ─ │ ┌ ┐ └ ┘    | Box drawing              |

---

## 6. Accessibility Checklist

- [x] Color contrast ratio ≥ 4.5:1 (WCAG AA)
- [x] No color-only information (always use symbols + color)
- [x] Keyboard navigation support (via @clack/prompts)
- [x] Screen reader hints where applicable
- [x] Graceful degradation for non-truecolor terminals
- [x] Skip banner option (CI/NO_BANNER env)
