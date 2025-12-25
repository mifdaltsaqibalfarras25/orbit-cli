# Typography — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Font Recommendations

### Primary: JetBrains Mono

**Why JetBrains Mono:**

- Optimized for code readability
- 142 code-specific ligatures
- Increased x-height for small sizes
- Clear distinction between similar chars (0/O, 1/l/I)
- Free and open source

```
JetBrains Mono Preview:

ORBIT CLI - Universal Project Generator

const framework = 'Next.js';
const isReady = environment.ready ?? false;
const arrow = () => console.log('→');

Ligatures: != == === => -> <- >= <=
```

### Alternative: Fira Code

**When to recommend Fira Code:**

- User prefers different aesthetic
- Wider character spacing needed
- More subtle ligatures

### Fallback Stack

```css
font-family: "JetBrains Mono", "Fira Code", "Cascadia Code", "SF Mono",
  "Consolas", "Monaco", monospace;
```

---

## 2. Typography Hierarchy

### Terminal Output Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   ████████╗  FIGLET BANNER (ASCII Art)                  │
│   ╚══██╔══╝  Font: ANSI Shadow, Bold weight             │
│      ██║     Style: Gradient coloring                   │
│      ██║                                                │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   PRIMARY MESSAGE                                       │
│   chalk.bold() — Important announcements               │
│   Example: "✓ Project created successfully"             │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   Secondary information                                 │
│   Normal weight — Details and descriptions             │
│   Example: "Installing 23 packages..."                  │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   dim hint text                                        │
│   chalk.dim() — Hints, shortcuts, meta info            │
│   Example: "(press Enter to continue)"                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### TypeScript Implementation

```typescript
// src/ui/text.ts
import chalk from "chalk";

export const text = {
  // Level 1: Headers/Titles
  title: (t: string) => chalk.bold(t),

  // Level 2: Primary content
  body: (t: string) => t,

  // Level 3: Secondary content
  secondary: (t: string) => chalk.hex("#A1A1AA")(t),

  // Level 4: Hints/Meta
  dim: (t: string) => chalk.dim(t),
  hint: (t: string) => chalk.hex("#71717A")(t),

  // Emphasis
  bold: (t: string) => chalk.bold(t),
  underline: (t: string) => chalk.underline(t),
  italic: (t: string) => chalk.italic(t),

  // Code/Technical
  code: (t: string) => chalk.hex("#22D3EE")(`\`${t}\``),
  path: (t: string) => chalk.underline.hex("#A1A1AA")(t),
  command: (t: string) => chalk.bold.hex("#22D3EE")(t),
};

// Usage examples
console.log(text.title("Project Configuration"));
console.log(text.body("Framework: Next.js 15"));
console.log(text.secondary("Using TypeScript with strict mode"));
console.log(text.hint("(press arrow keys to navigate)"));
console.log(`Run ${text.command("npm run dev")} to start`);
```

---

## 3. Spacing & Layout

### Vertical Rhythm

```typescript
// src/ui/layout.ts
export const layout = {
  // Empty line for breathing room
  spacer: () => console.log(),

  // Section divider
  divider: () => console.log(chalk.dim("─".repeat(50))),

  // Indentation levels
  indent: {
    l1: "  ", // 2 spaces
    l2: "    ", // 4 spaces
    l3: "      ", // 6 spaces
  },
};

// Example usage
function showProjectSummary(config: Config) {
  layout.spacer();
  console.log(text.title("Project Summary"));
  layout.spacer();
  console.log(`${layout.indent.l1}Framework:  ${config.framework}`);
  console.log(`${layout.indent.l1}Version:    ${config.version}`);
  console.log(`${layout.indent.l1}Stack:      ${config.stack}`);
  layout.spacer();
}
```

### Box Drawing Characters

```
┌──────────────────────────────────────┐
│  Configuration Summary               │
├──────────────────────────────────────┤
│                                      │
│  Framework:     Next.js 15           │
│  Stack:         full                 │
│  Package Mgr:   bun                  │
│  Project:       my-awesome-app       │
│                                      │
└──────────────────────────────────────┘

Characters used:
┌ ─ ┐   Top corners and horizontal line
│       Vertical line
├ ─ ┤   Intersection points
└ ─ ┘   Bottom corners
```

---

## 4. Text Styling Patterns

### Status Messages

```typescript
// Success
console.log(`${chalk.green("✓")} Project created successfully`);

// Error
console.log(`${chalk.red("✗")} Installation failed`);
console.log(chalk.dim(`  Error: ENOENT - directory not found`));

// Warning
console.log(`${chalk.yellow("⚠")} Node.js version is outdated`);
console.log(chalk.dim(`  Recommended: v18.0.0 or higher`));

// Info
console.log(`${chalk.blue("ℹ")} Using default configuration`);
```

### Progress Indicators

```typescript
// Spinner text
spinner.start("Creating project structure...");
spinner.text = "Installing dependencies...";
spinner.text = "Applying configurations...";
spinner.succeed("Setup complete");

// Step counter
console.log(chalk.dim("Step 1/4:") + " Framework selection");
console.log(chalk.dim("Step 2/4:") + " Version selection");
```

### Interactive Prompts (@clack/prompts style)

```
┌  Which framework would you like to use?
│
◆  Next.js        React framework for production
│  Nuxt           Vue framework with SSR
│  Astro          Content-focused framework
│  SvelteKit      Svelte meta-framework
│
└  Use arrow keys to navigate, Enter to select
```

---

## 5. Figlet Fonts Comparison

### ANSI Shadow (Recommended for Banner)

```
 ██████╗ ██████╗ ██████╗ ██╗████████╗
██╔═══██╗██╔══██╗██╔══██╗██║╚══██╔══╝
██║   ██║██████╔╝██████╔╝██║   ██║
██║   ██║██╔══██╗██╔══██╗██║   ██║
╚██████╔╝██║  ██║██████╔╝██║   ██║
 ╚═════╝ ╚═╝  ╚═╝╚═════╝ ╚═╝   ╚═╝
```

### Slant (Alternative - More Compact)

```
   ____  ____  ____  __________
  / __ \/ __ \/ __ )/  _/_  __/
 / / / / /_/ / __  || /  / /
/ /_/ / _, _/ /_/ // /  / /
\____/_/ |_/_____/___/ /_/
```

### Standard (Minimal Mode)

```
  ___  ____  ____ ___ _____
 / _ \|  _ \| __ )_ _|_   _|
| | | | |_) |  _ \| |  | |
| |_| |  _ <| |_) | |  | |
 \___/|_| \_\____/___| |_|
```

---

## 6. Character Set Reference

### Special Characters for CLI

| Category | Characters            | Usage             |
| :------- | :-------------------- | :---------------- |
| Arrows   | → ← ↑ ↓ ▸ ▾           | Navigation hints  |
| Bullets  | • ◦ ▪ ▫               | List items        |
| Checks   | ✓ ✗ ● ○               | Status indicators |
| Shapes   | ◆ ◇ ■ □               | Selection states  |
| Lines    | ─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼ | Box drawing       |
| Progress | ░ ▒ ▓ █               | Progress bars     |
| Misc     | ⚡ ⚠ ℹ ★ ☆            | Icons             |
