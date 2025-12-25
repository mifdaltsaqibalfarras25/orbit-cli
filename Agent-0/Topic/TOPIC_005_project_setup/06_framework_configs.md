# Framework Configurations — ORBIT CLI

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. Framework Registry

### Registry Structure

```typescript
// src/frameworks/types.ts

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
  readonly installCommand: InstallCommand;
  readonly stacks: readonly StackPreset[];
  readonly requiredTools: readonly string[];
}

export interface InstallCommand {
  readonly npm: string;
  readonly yarn: string;
  readonly pnpm: string;
  readonly bun: string;
  readonly flags: {
    readonly typescript?: string;
    readonly eslint?: string;
    readonly tailwind?: string;
    readonly srcDir?: string;
  };
}

export interface StackPreset {
  readonly id: "minimal" | "standard" | "full";
  readonly name: string;
  readonly description: string;
  readonly postInstallDeps?: readonly string[];
  readonly postInstallDevDeps?: readonly string[];
}
```

---

## 2. Framework Configurations

### 2.1 Next.js

```typescript
// src/frameworks/nextjs.ts

import type { Framework } from "./types.js";

const nextjs: Framework = {
  id: "nextjs",
  name: "Next.js",
  description: "The React Framework for the Web",
  category: "nodejs",
  website: "https://nextjs.org",
  requiredTools: ["node", "npm"],

  installCommand: {
    npm: "npx create-next-app@latest",
    yarn: "yarn create next-app",
    pnpm: "pnpm create next-app",
    bun: "bunx create-next-app",
    flags: {
      typescript: "--ts",
      eslint: "--eslint",
      tailwind: "--tailwind",
      srcDir: "--src-dir",
    },
  },

  stacks: [
    {
      id: "minimal",
      name: "Minimal",
      description: "Just Next.js with TypeScript",
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
    {
      id: "standard",
      name: "Standard",
      description: "Next.js + Tailwind + ESLint",
      postInstallDeps: [],
      postInstallDevDeps: ["prettier", "prettier-plugin-tailwindcss"],
    },
    {
      id: "full",
      name: "Full Stack",
      description: "Standard + Testing + Husky",
      postInstallDeps: ["zod"],
      postInstallDevDeps: [
        "vitest",
        "@testing-library/react",
        "husky",
        "lint-staged",
      ],
    },
  ],
};

export default nextjs;
```

### 2.2 Nuxt

```typescript
// src/frameworks/nuxt.ts

import type { Framework } from "./types.js";

const nuxt: Framework = {
  id: "nuxt",
  name: "Nuxt",
  description: "The Intuitive Vue Framework",
  category: "nodejs",
  website: "https://nuxt.com",
  requiredTools: ["node", "npm"],

  installCommand: {
    npm: "npx nuxi@latest init",
    yarn: "yarn dlx nuxi@latest init",
    pnpm: "pnpm dlx nuxi@latest init",
    bun: "bunx nuxi@latest init",
    flags: {}, // nuxi uses interactive prompts
  },

  stacks: [
    {
      id: "minimal",
      name: "Minimal",
      description: "Just Nuxt",
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
    {
      id: "standard",
      name: "Standard",
      description: "Nuxt + Tailwind + ESLint",
      postInstallDeps: [],
      postInstallDevDeps: [
        "@nuxtjs/tailwindcss",
        "@nuxtjs/eslint-config-typescript",
      ],
    },
    {
      id: "full",
      name: "Full Stack",
      description: "Standard + Pinia + Testing",
      postInstallDeps: ["@pinia/nuxt", "zod"],
      postInstallDevDeps: ["vitest", "@vue/test-utils", "husky"],
    },
  ],
};

export default nuxt;
```

### 2.3 Astro

```typescript
// src/frameworks/astro.ts

import type { Framework } from "./types.js";

const astro: Framework = {
  id: "astro",
  name: "Astro",
  description: "The web framework for content-driven websites",
  category: "nodejs",
  website: "https://astro.build",
  requiredTools: ["node", "npm"],

  installCommand: {
    npm: "npm create astro@latest",
    yarn: "yarn create astro",
    pnpm: "pnpm create astro@latest",
    bun: "bun create astro@latest",
    flags: {
      typescript: "--template with-typescript",
      tailwind: "--add tailwind",
    },
  },

  stacks: [
    {
      id: "minimal",
      name: "Minimal",
      description: "Just Astro",
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
    {
      id: "standard",
      name: "Standard",
      description: "Astro + Tailwind + MDX",
      postInstallDeps: ["@astrojs/mdx"],
      postInstallDevDeps: ["@astrojs/tailwind", "prettier-plugin-astro"],
    },
    {
      id: "full",
      name: "Full Stack",
      description: "Standard + React + Testing",
      postInstallDeps: ["@astrojs/react", "react", "react-dom"],
      postInstallDevDeps: ["vitest", "husky"],
    },
  ],
};

export default astro;
```

### 2.4 SvelteKit

```typescript
// src/frameworks/sveltekit.ts

import type { Framework } from "./types.js";

const sveltekit: Framework = {
  id: "sveltekit",
  name: "SvelteKit",
  description: "Web development, streamlined",
  category: "nodejs",
  website: "https://kit.svelte.dev",
  requiredTools: ["node", "npm"],

  installCommand: {
    npm: "npx sv create",
    yarn: "yarn create svelte",
    pnpm: "pnpm create svelte",
    bun: "bunx sv create",
    flags: {},
  },

  stacks: [
    {
      id: "minimal",
      name: "Minimal",
      description: "Just SvelteKit",
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
    {
      id: "standard",
      name: "Standard",
      description: "SvelteKit + Tailwind + Prettier",
      postInstallDeps: [],
      postInstallDevDeps: ["tailwindcss", "postcss", "autoprefixer"],
    },
    {
      id: "full",
      name: "Full Stack",
      description: "Standard + Drizzle + Testing",
      postInstallDeps: ["drizzle-orm", "zod"],
      postInstallDevDeps: ["drizzle-kit", "vitest", "@testing-library/svelte"],
    },
  ],
};

export default sveltekit;
```

### 2.5 Vue

```typescript
// src/frameworks/vue.ts

import type { Framework } from "./types.js";

const vue: Framework = {
  id: "vue",
  name: "Vue",
  description: "The Progressive JavaScript Framework",
  category: "nodejs",
  website: "https://vuejs.org",
  requiredTools: ["node", "npm"],

  installCommand: {
    npm: "npm create vue@latest",
    yarn: "yarn create vue",
    pnpm: "pnpm create vue",
    bun: "bun create vue@latest",
    flags: {},
  },

  stacks: [
    {
      id: "minimal",
      name: "Minimal",
      description: "Just Vue",
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
    {
      id: "standard",
      name: "Standard",
      description: "Vue + Router + Pinia",
      postInstallDeps: ["vue-router", "pinia"],
      postInstallDevDeps: ["tailwindcss"],
    },
    {
      id: "full",
      name: "Full Stack",
      description: "Standard + Testing + Husky",
      postInstallDeps: ["zod", "axios"],
      postInstallDevDeps: ["vitest", "@vue/test-utils", "husky"],
    },
  ],
};

export default vue;
```

### 2.6 Remix

```typescript
// src/frameworks/remix.ts

import type { Framework } from "./types.js";

const remix: Framework = {
  id: "remix",
  name: "Remix",
  description: "Full stack web framework",
  category: "nodejs",
  website: "https://remix.run",
  requiredTools: ["node", "npm"],

  installCommand: {
    npm: "npx create-remix@latest",
    yarn: "yarn create remix",
    pnpm: "pnpm create remix@latest",
    bun: "bunx create-remix@latest",
    flags: {},
  },

  stacks: [
    {
      id: "minimal",
      name: "Minimal",
      description: "Just Remix",
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
    {
      id: "standard",
      name: "Standard",
      description: "Remix + Tailwind",
      postInstallDeps: [],
      postInstallDevDeps: ["tailwindcss", "postcss", "autoprefixer"],
    },
    {
      id: "full",
      name: "Full Stack",
      description: "Standard + Prisma + Testing",
      postInstallDeps: ["@prisma/client", "zod"],
      postInstallDevDeps: ["prisma", "vitest", "@testing-library/react"],
    },
  ],
};

export default remix;
```

### 2.7 Laravel

```typescript
// src/frameworks/laravel.ts

import type { Framework } from "./types.js";

const laravel: Framework = {
  id: "laravel",
  name: "Laravel",
  description: "The PHP Framework for Web Artisans",
  category: "php",
  website: "https://laravel.com",
  requiredTools: ["php", "composer"],

  installCommand: {
    npm: "composer create-project laravel/laravel", // npm tidak dipakai
    yarn: "composer create-project laravel/laravel",
    pnpm: "composer create-project laravel/laravel",
    bun: "composer create-project laravel/laravel",
    flags: {},
  },

  stacks: [
    {
      id: "minimal",
      name: "Minimal",
      description: "Just Laravel",
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
    {
      id: "standard",
      name: "Standard",
      description: "Laravel + Breeze + Inertia Vue",
      // Note: Breeze installation is different
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
    {
      id: "full",
      name: "Full Stack",
      description: "Standard + Pest + Telescope",
      postInstallDeps: [],
      postInstallDevDeps: [],
    },
  ],
};

export default laravel;
```

---

## 3. Install Command Builder

```typescript
// src/core/services/command-builder.ts

import type { Framework, StackPreset } from "../frameworks/types.js";
import type { PackageManager } from "../domain/project.js";

interface BuildResult {
  primary: { command: string; args: string[] };
  postInstall?: { command: string; args: string[] }[];
}

export function buildInstallCommand(
  framework: Framework,
  projectName: string,
  pm: PackageManager,
  stack: StackPreset,
  options: {
    typescript?: boolean;
    eslint?: boolean;
    tailwind?: boolean;
  } = {}
): BuildResult {
  const baseCmd = framework.installCommand[pm];
  const [command, ...baseArgs] = baseCmd.split(" ");

  // Add project name
  const args = [...baseArgs, projectName];

  // Add flags
  const flags = framework.installCommand.flags;
  if (options.typescript && flags.typescript) {
    args.push(flags.typescript);
  }
  if (options.eslint && flags.eslint) {
    args.push(flags.eslint);
  }
  if (options.tailwind && flags.tailwind) {
    args.push(flags.tailwind);
  }

  // Build post-install commands
  const postInstall: { command: string; args: string[] }[] = [];

  if (stack.postInstallDeps?.length) {
    postInstall.push({
      command: pm,
      args: ["add", ...stack.postInstallDeps],
    });
  }

  if (stack.postInstallDevDeps?.length) {
    postInstall.push({
      command: pm,
      args: ["add", "-D", ...stack.postInstallDevDeps],
    });
  }

  return {
    primary: { command, args },
    postInstall: postInstall.length > 0 ? postInstall : undefined,
  };
}
```

---

## 4. Usage Example

```typescript
// Contoh penggunaan di use case

const framework = await registry.get("nextjs");
const stack = framework.stacks.find((s) => s.id === "standard")!;

const commands = buildInstallCommand(framework, "my-app", "pnpm", stack, {
  typescript: true,
  tailwind: true,
});

// commands.primary:
// { command: 'pnpm', args: ['create', 'next-app', 'my-app', '--ts', '--tailwind'] }

// commands.postInstall:
// [{ command: 'pnpm', args: ['add', '-D', 'prettier', 'prettier-plugin-tailwindcss'] }]
```
