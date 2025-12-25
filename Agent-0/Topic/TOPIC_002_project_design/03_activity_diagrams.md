# Activity Diagrams

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## AD-01: Environment Detection

### Description

Proses deteksi environment berdasarkan framework yang dipilih.

### Diagram

```mermaid
flowchart TD
    Start([Start: detectEnvironment]) --> A{Framework Type?}

    A -->|Node-based| B[checkNodeJS]
    B --> C{Node.js installed?}
    C -->|No| D[return: MissingNode]
    C -->|Yes| E[getNodeVersion]
    E --> F{Version >= 18?}
    F -->|No| G[return: NodeVersionTooLow]
    F -->|Yes| H[detectPackageManagers]
    H --> I[Check npm]
    I --> J[Check yarn]
    J --> K[Check pnpm]
    K --> L[Check bun]
    L --> M[return: NodeReady + PM list]

    A -->|PHP-based| N[checkPHP]
    N --> O{PHP installed?}
    O -->|No| P[return: MissingPHP]
    O -->|Yes| Q[getPHPVersion]
    Q --> R{Version >= 8.1?}
    R -->|No| S[return: PHPVersionTooLow]
    R -->|Yes| T[checkComposer]
    T --> U{Composer installed?}
    U -->|No| V[return: MissingComposer]
    U -->|Yes| W[return: PHPReady]

    D --> End([End])
    G --> End
    M --> End
    P --> End
    S --> End
    V --> End
    W --> End
```

### Pseudocode

```typescript
async function detectEnvironment(framework: Framework): Promise<EnvResult> {
  const type = framework.environmentType;

  if (type === "node" || type === "hybrid") {
    const nodeCheck = await checkNodeJS();
    if (!nodeCheck.installed) {
      return {
        ready: false,
        missing: "Node.js",
        instructions: getNodeInstructions(),
      };
    }
    if (nodeCheck.version < "18.0.0") {
      return { ready: false, error: "NodeVersionTooLow", required: ">=18.0.0" };
    }
    const pms = await detectPackageManagers();
    if (pms.length === 0) {
      return {
        ready: false,
        missing: "PackageManager",
        instructions: getNpmInstructions(),
      };
    }
  }

  if (type === "php" || type === "hybrid") {
    const phpCheck = await checkPHP();
    if (!phpCheck.installed) {
      return {
        ready: false,
        missing: "PHP",
        instructions: getPHPInstructions(),
      };
    }
    if (phpCheck.version < "8.1.0") {
      return { ready: false, error: "PHPVersionTooLow", required: ">=8.1.0" };
    }
    const composerCheck = await checkComposer();
    if (!composerCheck.installed) {
      return {
        ready: false,
        missing: "Composer",
        instructions: getComposerInstructions(),
      };
    }
  }

  return { ready: true, packageManagers: pms };
}
```

---

## AD-02: Project Installation

### Description

Proses instalasi project dari awal sampai selesai.

### Diagram

```mermaid
flowchart TD
    Start([Start: installProject]) --> A[Validate Config]
    A --> B{Config Valid?}
    B -->|No| C[throw ConfigError]
    B -->|Yes| D[Resolve Target Directory]

    D --> E{Directory Exists?}
    E -->|Yes| F{Directory Empty?}
    F -->|No| G[Prompt: Overwrite?]
    G -->|No| H[Abort]
    G -->|Yes| I[Clear Directory]
    F -->|Yes| J[Use Directory]
    E -->|No| K[Create Directory]
    I --> J
    K --> J

    J --> L[Start Spinner]
    L --> M[Build Create Command]
    M --> N[Execute Create Command]
    N --> O{Command Success?}
    O -->|No| P[Stop Spinner: Fail]
    P --> Q[Throw InstallError]

    O -->|Yes| R[Update Spinner: Installing deps]
    R --> S{Skip Install?}
    S -->|Yes| T[Skip]
    S -->|No| U[Run Package Manager Install]
    U --> V{Install Success?}
    V -->|No| W[Stop Spinner: Warn]
    V -->|Yes| X[Update Spinner: Applying stack]

    T --> Y[Apply Stack Configs]
    X --> Y
    W --> Y

    Y --> Z[Run Post-Install Scripts]
    Z --> AA{Init Git?}
    AA -->|Yes| AB[git init + first commit]
    AA -->|No| AC[Skip git]

    AB --> AD[Stop Spinner: Success]
    AC --> AD
    AD --> AE[return: Success]

    C --> End([End])
    H --> End
    Q --> End
    AE --> End
```

### Pseudocode

```typescript
async function installProject(config: ProjectConfig): Promise<InstallResult> {
  // Validate
  validateConfig(config);

  // Resolve directory
  const targetDir = path.resolve(config.projectName);
  await ensureDirectoryEmpty(targetDir);

  // Start installation
  const spinner = ora().start("Creating project...");

  try {
    // Build and execute create command
    const command = buildCreateCommand(config);
    await executor.run(command, { cwd: process.cwd() });

    // Install dependencies
    if (!config.skipInstall) {
      spinner.text = "Installing dependencies...";
      await executor.run(`${config.packageManager} install`, {
        cwd: targetDir,
      });
    }

    // Apply stack configurations
    spinner.text = "Applying stack configurations...";
    await applyStackConfigs(config.stack, targetDir);

    // Post-install scripts
    await runPostInstallScripts(config.framework, targetDir);

    // Git init
    if (config.git) {
      await executor.run("git init", { cwd: targetDir });
      await executor.run("git add .", { cwd: targetDir });
      await executor.run('git commit -m "Initial commit via ORBIT CLI"', {
        cwd: targetDir,
      });
    }

    spinner.succeed("Project created successfully!");
    return { success: true, path: targetDir };
  } catch (error) {
    spinner.fail("Installation failed");
    throw new InstallError(error);
  }
}
```

---

## AD-03: Stack Configuration Apply

### Description

Proses aplikasi stack configuration ke project yang baru dibuat.

### Diagram

```mermaid
flowchart TD
    Start([Start: applyStackConfigs]) --> A[Load Stack Definition]
    A --> B[Get Stack Plugins]

    B --> C{Has TypeScript?}
    C -->|Yes| D[Configure tsconfig.json]
    C -->|No| E[Skip TS]

    D --> F{Has ESLint?}
    E --> F
    F -->|Yes| G[Create .eslintrc.js]
    G --> H[Add ESLint deps to package.json]
    F -->|No| I[Skip ESLint]

    H --> J{Has Prettier?}
    I --> J
    J -->|Yes| K[Create .prettierrc]
    K --> L[Add Prettier deps]
    J -->|No| M[Skip Prettier]

    L --> N{Has Tailwind?}
    M --> N
    N -->|Yes| O[Configure tailwind.config.js]
    O --> P[Update CSS imports]
    P --> Q[Add Tailwind deps]
    N -->|No| R[Skip Tailwind]

    Q --> S{Has Husky?}
    R --> S
    S -->|Yes| T[Init Husky]
    T --> U[Add pre-commit hooks]
    U --> V[Add lint-staged]
    S -->|No| W[Skip Husky]

    V --> X[Update package.json scripts]
    W --> X
    X --> Y[Run npm/yarn/bun install]
    Y --> End([End])
```

---

## AD-04: Interactive Prompt Flow

### Description

Alur interactive prompts untuk mengumpulkan user input.

### Diagram

```mermaid
flowchart TD
    Start([Start: runInteractivePrompts]) --> A[p.intro]

    A --> B[p.select: Framework]
    B --> C{User cancelled?}
    C -->|Yes| D[p.cancel + exit]
    C -->|No| E[Store framework]

    E --> F[Check Environment]
    F --> G{Env Ready?}
    G -->|No| H[Show missing deps]
    H --> I[p.confirm: Retry?]
    I -->|Yes| F
    I -->|No| D

    G -->|Yes| J[p.select: Version]
    J --> K{User cancelled?}
    K -->|Yes| D
    K -->|No| L[Store version]

    L --> M[p.select: Package Manager]
    M --> N{User cancelled?}
    N -->|Yes| D
    N -->|No| O[Store PM]

    O --> P[p.select: Stack]
    P --> Q{User cancelled?}
    Q -->|Yes| D
    Q -->|No| R[Store stack]

    R --> S[p.text: Project Name]
    S --> T{Valid name?}
    T -->|No| U[Show error]
    U --> S
    T -->|Yes| V[Store name]

    V --> W[Show Summary]
    W --> X[p.confirm: Proceed?]
    X -->|No| B
    X -->|Yes| Y[Return config]

    D --> End([End: Cancelled])
    Y --> End2([End: Config Ready])
```

---

## AD-05: Command Execution

### Description

Proses eksekusi shell command dengan error handling.

### Diagram

```mermaid
flowchart TD
    Start([Start: executeCommand]) --> A[Parse command string]
    A --> B[Resolve binary path]
    B --> C{Binary exists?}
    C -->|No| D[throw: CommandNotFound]

    C -->|Yes| E[Spawn child process]
    E --> F[Setup stderr capturing]
    F --> G[Setup stdout capturing]
    G --> H[Wait for process]

    H --> I{Exit code = 0?}
    I -->|Yes| J[return: Success + stdout]
    I -->|No| K{Known error pattern?}
    K -->|Yes| L[Parse error + suggest fix]
    K -->|No| M[Generic error message]

    L --> N[throw: CommandError]
    M --> N

    D --> End([End])
    J --> End
    N --> End
```

### Pseudocode

```typescript
async function executeCommand(
  command: string,
  options: ExecOptions
): Promise<ExecResult> {
  const [binary, ...args] = command.split(" ");

  // Check if binary exists
  const binaryPath = await which(binary).catch(() => null);
  if (!binaryPath) {
    throw new CommandNotFoundError(binary);
  }

  return new Promise((resolve, reject) => {
    const child = spawn(binary, args, {
      cwd: options.cwd,
      stdio: ["inherit", "pipe", "pipe"],
      env: { ...process.env, ...options.env },
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
        resolve({ success: true, stdout, stderr });
      } else {
        const error = parseKnownError(stderr) || new CommandError(stderr);
        reject(error);
      }
    });

    child.on("error", (err) => {
      reject(new CommandError(err.message));
    });
  });
}
```
