# Color Palette — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Core Color System

### Dark Theme Foundation

```
┌─────────────────────────────────────────────────────────┐
│  ORBIT CLI - Dark & Sleek Theme                        │
│                                                         │
│  Background Layers:                                     │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Deep Space    #09090B  ████████████████████    │   │
│  │  Dark Matter   #18181B  ████████████████████    │   │
│  │  Asteroid      #27272A  ████████████████████    │   │
│  │  Comet         #3F3F46  ████████████████████    │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  Text Hierarchy:                                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Starlight     #FAFAFA  Primary text            │   │
│  │  Moonlight     #A1A1AA  Secondary/dim           │   │
│  │  Twilight      #71717A  Hints/placeholders      │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Semantic Colors (ANSI Compatible)

### Status Colors

| Status     | Name         | Hex       | ANSI 256 | ANSI Basic   | RGB           |
| :--------- | :----------- | :-------- | :------- | :----------- | :------------ |
| ✅ Success | Aurora Green | `#10B981` | 42       | green        | 16, 185, 129  |
| ❌ Error   | Mars Red     | `#EF4444` | 196      | red          | 239, 68, 68   |
| ⚠️ Warning | Solar Yellow | `#F59E0B` | 214      | yellow       | 245, 158, 11  |
| ℹ️ Info    | Cosmic Blue  | `#6366F1` | 99       | blue         | 99, 102, 241  |
| 💡 Hint    | Moonlight    | `#A1A1AA` | 248      | bright black | 161, 161, 170 |

### TypeScript Implementation

```typescript
// src/ui/colors.ts
import chalk from "chalk";

export const colors = {
  // Status colors
  success: chalk.hex("#10B981"),
  error: chalk.hex("#EF4444"),
  warning: chalk.hex("#F59E0B"),
  info: chalk.hex("#6366F1"),
  hint: chalk.hex("#A1A1AA"),

  // Brand colors
  primary: chalk.hex("#8B5CF6"),
  secondary: chalk.hex("#6366F1"),
  accent: chalk.hex("#22D3EE"),

  // Text colors
  text: chalk.hex("#FAFAFA"),
  dim: chalk.hex("#A1A1AA"),
  muted: chalk.hex("#71717A"),

  // Utilities
  bold: chalk.bold,
  underline: chalk.underline,

  // Composable styles
  successBold: chalk.hex("#10B981").bold,
  errorBold: chalk.hex("#EF4444").bold,
};

// Semantic shortcuts
export const c = {
  ok: (text: string) => `${chalk.hex("#10B981")("✓")} ${text}`,
  fail: (text: string) => `${chalk.hex("#EF4444")("✗")} ${text}`,
  warn: (text: string) => `${chalk.hex("#F59E0B")("⚠")} ${text}`,
  info: (text: string) => `${chalk.hex("#6366F1")("ℹ")} ${text}`,
};
```

---

## 3. Gradient Definitions

### Primary Gradients

```typescript
// src/ui/gradients.ts
import gradient from "gradient-string";

// Cosmic Nebula — Primary brand gradient (purple → blue → cyan)
export const nebula = gradient([
  "#8B5CF6", // Violet
  "#6366F1", // Indigo
  "#3B82F6", // Blue
  "#22D3EE", // Cyan
]);

// Cosmic Duo — Simplified brand gradient
export const cosmic = gradient([
  "#8B5CF6", // Violet
  "#6366F1", // Indigo
]);

// Aurora — Success/completion gradient
export const aurora = gradient([
  "#10B981", // Emerald
  "#14B8A6", // Teal
  "#22D3EE", // Cyan
]);

// Supernova — Error/alert gradient
export const supernova = gradient([
  "#EF4444", // Red
  "#F97316", // Orange
]);

// Stellar — Info/neutral gradient
export const stellar = gradient([
  "#6366F1", // Indigo
  "#8B5CF6", // Violet
]);

// Usage example
export function showGradientBanner(text: string): void {
  console.log(nebula.multiline(text));
}
```

### Visual Preview

```
GRADIENT: Nebula (Primary Banner)
████████████████████████████████████████
#8B5CF6 → #6366F1 → #3B82F6 → #22D3EE

GRADIENT: Cosmic (Accent Elements)
████████████████████████████████████████
#8B5CF6 → #6366F1

GRADIENT: Aurora (Success States)
████████████████████████████████████████
#10B981 → #14B8A6 → #22D3EE
```

---

## 4. Color Contrast Matrix

### Accessibility Check (WCAG AA)

| Background         | Foreground        | Ratio | Pass   |
| :----------------- | :---------------- | :---- | :----- |
| Deep Space #09090B | Starlight #FAFAFA | 21:1  | ✅ AAA |
| Deep Space #09090B | Moonlight #A1A1AA | 7.5:1 | ✅ AAA |
| Deep Space #09090B | Primary #8B5CF6   | 5.2:1 | ✅ AA  |
| Deep Space #09090B | Success #10B981   | 8.3:1 | ✅ AAA |
| Deep Space #09090B | Error #EF4444     | 5.1:1 | ✅ AA  |
| Deep Space #09090B | Warning #F59E0B   | 9.2:1 | ✅ AAA |

---

## 5. ANSI Fallback Colors

### For terminals without truecolor support

```typescript
// src/ui/colors-fallback.ts
import chalk from "chalk";

// Check if truecolor is supported
const supportsHex = chalk.level >= 3;

export const colors = supportsHex
  ? {
      success: chalk.hex("#10B981"),
      error: chalk.hex("#EF4444"),
      warning: chalk.hex("#F59E0B"),
      info: chalk.hex("#6366F1"),
      primary: chalk.hex("#8B5CF6"),
    }
  : {
      // Fallback to ANSI 16 colors
      success: chalk.green,
      error: chalk.red,
      warning: chalk.yellow,
      info: chalk.blue,
      primary: chalk.magenta,
    };
```

---

## 6. Theme Configuration Object

### Complete Theme Definition

```typescript
// src/ui/theme.ts
export interface OrbitTheme {
  name: string;
  colors: {
    // Background layers
    background: string;
    surface: string;
    border: string;

    // Text hierarchy
    text: string;
    textSecondary: string;
    textMuted: string;

    // Brand colors
    primary: string;
    secondary: string;
    accent: string;

    // Status colors
    success: string;
    error: string;
    warning: string;
    info: string;
  };
  gradients: {
    banner: string[];
    success: string[];
    error: string[];
  };
}

export const orbitTheme: OrbitTheme = {
  name: "orbit",
  colors: {
    // Background layers
    background: "#09090B",
    surface: "#18181B",
    border: "#27272A",

    // Text hierarchy
    text: "#FAFAFA",
    textSecondary: "#A1A1AA",
    textMuted: "#71717A",

    // Brand colors
    primary: "#8B5CF6",
    secondary: "#6366F1",
    accent: "#22D3EE",

    // Status colors
    success: "#10B981",
    error: "#EF4444",
    warning: "#F59E0B",
    info: "#6366F1",
  },
  gradients: {
    banner: ["#8B5CF6", "#6366F1", "#3B82F6", "#22D3EE"],
    success: ["#10B981", "#14B8A6", "#22D3EE"],
    error: ["#EF4444", "#F97316"],
  },
};
```

---

## 7. Color Usage Guidelines

### Do's ✅

```typescript
// Use colors semantically
console.log(colors.success("✓ Project created"));
console.log(colors.error("✗ Installation failed"));
console.log(colors.dim("Press Enter to continue"));

// Use gradients for emphasis
console.log(nebula.multiline(banner));
```

### Don'ts ❌

```typescript
// Don't overuse colors
console.log(chalk.red.bgYellow.bold.underline("Text")); // Too much

// Don't use colors for non-semantic purposes
console.log(chalk.red("Welcome!")); // Red ≠ error here

// Don't mix too many colors in one message
console.log(`${chalk.red("a")} ${chalk.blue("b")} ${chalk.green("c")}`);
```
