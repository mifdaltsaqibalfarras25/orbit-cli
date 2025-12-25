# Brand Identity — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Brand Concept

### Name: ORBIT

**Meaning:**

- **Orbital Motion** — Tool yang mengorbit di sekitar developer, siap membantu
- **Universal Rotation** — Dapat digunakan untuk berbagai framework (universal)
- **Cosmic/Space Theme** — Professional, futuristic, modern

### Tagline Options

| Option | Tagline                           |
| :----- | :-------------------------------- |
| 1      | "Launch your projects into orbit" |
| 2      | "Universal project generator"     |
| 3      | "Your development launchpad"      |

---

## 2. Logo Concept (ASCII Art)

### Primary Banner

```
   ____  _____  ____ _____ _______
  / __ \|  __ \|  _ \_   _|__   __|
 | |  | | |__) | |_) || |    | |
 | |  | |  _  /|  _ < | |    | |
 | |__| | | \ \| |_) || |_   | |
  \____/|_|  \_\____/_____|  |_|

  🚀 Universal Project Generator
```

### Figlet Fonts Recommended

| Font          | Look            | Usage          |
| :------------ | :-------------- | :------------- |
| `ANSI Shadow` | Bold, dramatic  | Primary banner |
| `Slant`       | Sleek, italic   | Alternative    |
| `Standard`    | Clean, readable | Minimal mode   |

### TypeScript Implementation

```typescript
// src/ui/banner.ts
import figlet from "figlet";
import gradient from "gradient-string";

// Custom cosmic gradient
const cosmicGradient = gradient([
  "#667eea", // Purple/indigo
  "#764ba2", // Violet
  "#6B8DD6", // Soft blue
  "#8E37D7", // Bright purple
]);

export async function showBanner(): Promise<void> {
  // Skip in non-TTY or CI
  if (!process.stdout.isTTY || process.env.CI) return;

  const banner = figlet.textSync("ORBIT", {
    font: "ANSI Shadow",
    horizontalLayout: "default",
  });

  console.log(cosmicGradient.multiline(banner));
  console.log();
  console.log("  🚀 Universal Project Generator");
  console.log();
}
```

---

## 3. Brand Colors

### Primary Palette (Cosmic Theme)

| Role          | Name          | Hex       | ANSI | Usage                     |
| :------------ | :------------ | :-------- | :--- | :------------------------ |
| **Primary**   | Nebula Purple | `#8B5CF6` | 135  | Gradient start, accent    |
| **Secondary** | Cosmic Blue   | `#6366F1` | 99   | Gradient end, interactive |
| **Accent**    | Stellar Cyan  | `#22D3EE` | 51   | Highlights, focus         |
| **Success**   | Aurora Green  | `#10B981` | 42   | Success states            |
| **Warning**   | Solar Yellow  | `#F59E0B` | 214  | Warnings                  |
| **Error**     | Mars Red      | `#EF4444` | 196  | Errors                    |

### Background & Text

| Role               | Name        | Hex       | Usage                   |
| :----------------- | :---------- | :-------- | :---------------------- |
| **Background**     | Deep Space  | `#09090B` | Terminal bg (reference) |
| **Surface**        | Dark Matter | `#18181B` | Cards, elevations       |
| **Text Primary**   | Starlight   | `#FAFAFA` | Main text               |
| **Text Secondary** | Moonlight   | `#A1A1AA` | Dim text, hints         |
| **Border**         | Asteroid    | `#27272A` | Dividers, outlines      |

### Gradient Definitions

```typescript
// gradient-string custom gradients
const gradients = {
  // Primary brand gradient (purple → blue)
  cosmic: gradient(["#8B5CF6", "#6366F1"]),

  // Success gradient (green → teal)
  aurora: gradient(["#10B981", "#14B8A6"]),

  // Error gradient (red → orange)
  supernova: gradient(["#EF4444", "#F97316"]),

  // Welcome banner gradient (multicolor)
  nebula: gradient([
    "#8B5CF6", // Purple
    "#6366F1", // Indigo
    "#3B82F6", // Blue
    "#22D3EE", // Cyan
  ]),
};
```

---

## 4. Brand Voice & Tone

### Personality

| Trait            | Description            | Example                                                         |
| :--------------- | :--------------------- | :-------------------------------------------------------------- |
| **Professional** | Tidak terlalu casual   | "Project created successfully" ✅ bukan "Awesome! You're done!" |
| **Helpful**      | Actionable messages    | "Run `cd my-app` to get started"                                |
| **Confident**    | Clear assertions       | "Installing dependencies..." bukan "Trying to install..."       |
| **Friendly**     | Approachable, not cold | Emoji sparingly, warm language                                  |

### Message Examples

```typescript
// ✅ Good
console.log("✓ Project created successfully");
console.log("  cd my-app && npm run dev");

// ❌ Too casual
console.log("🎉 Woohoo! Your awesome project is ready! 🚀🎊");

// ❌ Too cold
console.log("Operation completed. Exiting.");
```

---

## 5. Symbol System

### Status Indicators

| Symbol | Unicode | Meaning  | Usage             |
| :----- | :------ | :------- | :---------------- |
| ✓      | U+2713  | Success  | Completed actions |
| ✗      | U+2717  | Failure  | Errors            |
| ●      | U+25CF  | Active   | Current step      |
| ○      | U+25CB  | Inactive | Pending step      |
| ◆      | U+25C6  | Selected | Current selection |
| ◇      | U+25C7  | Option   | Unselected option |
| ▸      | U+25B8  | Arrow    | Navigation hint   |
| ⚠      | U+26A0  | Warning  | Warnings          |
| ℹ      | U+2139  | Info     | Information       |

### Emoji Usage (Sparingly)

| Emoji | Usage                     |
| :---- | :------------------------ |
| 🚀    | Launch/start actions      |
| 📦    | Package/framework related |
| ⚡    | Fast/performance hints    |
| 🔧    | Configuration/tools       |
| 💡    | Tips/suggestions          |

---

## 6. Competitive Positioning

| CLI         | Style                 | ORBIT Differentiation |
| :---------- | :-------------------- | :-------------------- |
| Astro       | Playful, colorful     | More professional     |
| Vite        | Minimal, fast         | Richer UX             |
| OpenCode    | Feature-rich, managed | Simpler, focused      |
| Claude Code | Conversational        | More structured       |

**ORBIT's Unique Position:**

> Professional, cosmic-themed CLI dengan gradient accents yang memberikan pengalaman premium tanpa overwhelming complexity.
