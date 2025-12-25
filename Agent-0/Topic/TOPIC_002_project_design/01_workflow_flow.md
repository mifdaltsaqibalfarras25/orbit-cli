# Workflow & Application Flow

**Parent:** [← Kembali ke Main](_main.md)
**Status:** ✅ Created

---

## 1. High-Level Workflow

```mermaid
flowchart TD
    subgraph Entry["🚀 Entry Point"]
        A[User runs: orbit create] --> B[Parse CLI arguments]
        B --> C{Has --template flag?}
    end

    subgraph Interactive["💬 Interactive Mode"]
        C -->|No| D[Show Welcome Banner]
        D --> E[Select Framework]
        E --> F[Check Environment]
        F --> G{Environment OK?}
        G -->|No| H[Show Install Instructions]
        H --> I[User fixes environment]
        I --> F
        G -->|Yes| J[Select Version]
        J --> K[Select Package Manager]
        K --> L[Select Stack/Presets]
        L --> M[Input Project Name]
        M --> N[Show Configuration Summary]
        N --> O{User Confirms?}
        O -->|No| E
    end

    subgraph Quick["⚡ Quick Mode"]
        C -->|Yes| P[Load Template Config]
        P --> Q[Validate Template]
        Q --> R{Template Valid?}
        R -->|No| S[Error: Invalid Template]
        R -->|Yes| T[Merge with defaults]
    end

    subgraph Install["📦 Installation"]
        O -->|Yes| U[Start Spinner]
        T --> U
        U --> V[Execute Create Command]
        V --> W[Install Dependencies]
        W --> X[Apply Stack Configs]
        X --> Y[Post-install Scripts]
    end

    subgraph Complete["✅ Completion"]
        Y --> Z[Show Success Message]
        Z --> AA[Display Next Steps]
        AA --> AB[End]
    end
```

---

## 2. Command Structure

### Primary Commands

| Command                             | Description                       | Example                              |
| :---------------------------------- | :-------------------------------- | :----------------------------------- |
| `orbit create [name]`               | Create new project (interactive)  | `orbit create my-app`                |
| `orbit create [name] -t <template>` | Create with template (quick mode) | `orbit create my-app -t nextjs-full` |
| `orbit list`                        | List available frameworks         | `orbit list`                         |
| `orbit list <framework>`            | List stacks for framework         | `orbit list nextjs`                  |
| `orbit doctor`                      | Check system environment          | `orbit doctor`                       |
| `orbit --version`                   | Show version                      | `orbit -V`                           |
| `orbit --help`                      | Show help                         | `orbit -h`                           |

### Command Options

```typescript
interface CreateOptions {
  template?: string; // -t, --template <name>
  packageManager?: string; // -p, --pm <npm|yarn|pnpm|bun>
  skipInstall?: boolean; // --skip-install
  git?: boolean; // --git / --no-git
  yes?: boolean; // -y, --yes (use defaults)
}
```

---

## 3. State Machine: Create Flow

```mermaid
stateDiagram-v2
    [*] --> Idle

    Idle --> Parsing: orbit create
    Parsing --> QuickMode: has template flag
    Parsing --> InteractiveMode: no template flag

    state InteractiveMode {
        [*] --> ShowBanner
        ShowBanner --> SelectFramework
        SelectFramework --> CheckEnvironment
        CheckEnvironment --> EnvError: missing deps
        CheckEnvironment --> SelectVersion: deps OK
        EnvError --> ShowInstructions
        ShowInstructions --> CheckEnvironment: retry
        SelectVersion --> SelectPM
        SelectPM --> SelectStack
        SelectStack --> InputName
        InputName --> ShowSummary
        ShowSummary --> SelectFramework: user cancels
        ShowSummary --> ReadyToInstall: user confirms
    }

    state QuickMode {
        [*] --> LoadTemplate
        LoadTemplate --> ValidateTemplate
        ValidateTemplate --> TemplateError: invalid
        ValidateTemplate --> MergeDefaults: valid
        MergeDefaults --> ReadyToInstall
    }

    ReadyToInstall --> Installing

    state Installing {
        [*] --> ExecuteCreate
        ExecuteCreate --> InstallDeps
        InstallDeps --> ApplyStack
        ApplyStack --> PostInstall
        PostInstall --> [*]
    }

    Installing --> Success: completed
    Installing --> Failure: error

    Success --> [*]
    Failure --> [*]
    TemplateError --> [*]
```

---

## 4. Sequence Diagram: Interactive Create

```mermaid
sequenceDiagram
    participant U as User
    participant CLI as CLI Entry
    participant UI as UI Module
    participant Det as Detector
    participant Fw as Framework
    participant Inst as Installer
    participant Exec as Executor

    U->>CLI: orbit create
    CLI->>UI: showBanner()
    UI-->>U: Display ASCII Banner

    CLI->>UI: selectFramework()
    U-->>UI: Select "Next.js"
    UI-->>CLI: framework = "nextjs"

    CLI->>Det: checkEnvironment("nextjs")
    Det->>Det: Check Node.js
    Det->>Det: Detect Package Managers
    Det-->>CLI: { ready: true, pms: ["npm", "bun"] }

    CLI->>Fw: getVersions("nextjs")
    Fw-->>CLI: ["15", "14"]

    CLI->>UI: selectVersion(versions)
    U-->>UI: Select "15"

    CLI->>UI: selectPackageManager(pms)
    U-->>UI: Select "bun"

    CLI->>Fw: getStacks("nextjs")
    Fw-->>CLI: ["minimal", "standard", "full"]

    CLI->>UI: selectStack(stacks)
    U-->>UI: Select "full"

    CLI->>UI: inputProjectName()
    U-->>UI: Enter "my-app"

    CLI->>UI: showSummary(config)
    U-->>UI: Confirm

    CLI->>Inst: install(config)
    Inst->>UI: spinner.start("Creating project...")
    Inst->>Exec: runCommand("bunx create-next-app@latest...")
    Exec-->>Inst: success
    Inst->>UI: spinner.succeed("Project created!")

    CLI->>UI: showNextSteps(projectPath)
    UI-->>U: Display next steps
```

---

## 5. Error Handling Flow

```mermaid
flowchart TD
    A[Operation] --> B{Success?}
    B -->|Yes| C[Continue Flow]
    B -->|No| D{Error Type?}

    D -->|User Cancel| E[Graceful Exit]
    E --> F[Show goodbye message]
    F --> G[Exit code 0]

    D -->|Missing Dependency| H[Show Install Instructions]
    H --> I{Retry?}
    I -->|Yes| A
    I -->|No| J[Exit code 1]

    D -->|Network Error| K[Show Retry Option]
    K --> L{Retry?}
    L -->|Yes| A
    L -->|No| M[Exit code 1]

    D -->|Command Failed| N[Show Error Details]
    N --> O[Suggest Solutions]
    O --> P[Exit code 1]

    D -->|Unknown| Q[Log Error]
    Q --> R[Show Generic Message]
    R --> S[Exit code 1]
```

---

## 6. Module Interaction Map

```mermaid
flowchart LR
    subgraph Entry
        index[index.ts]
    end

    subgraph Core
        cli[cli.ts]
        detector[detector.ts]
        installer[installer.ts]
        executor[executor.ts]
    end

    subgraph UI
        banner[banner.ts]
        prompts[prompts.ts]
        spinner[spinner.ts]
    end

    subgraph Frameworks
        registry[index.ts]
        nextjs[nextjs/]
        nuxt[nuxt/]
        astro[astro/]
        svelte[svelte/]
        vue[vue/]
        remix[remix/]
        laravel[laravel/]
    end

    subgraph Utils
        system[system.ts]
        pm[pm.ts]
        logger[logger.ts]
    end

    index --> cli
    cli --> banner
    cli --> prompts
    cli --> detector
    cli --> registry
    cli --> installer

    detector --> system
    detector --> pm

    installer --> spinner
    installer --> executor
    installer --> registry

    executor --> logger

    registry --> nextjs
    registry --> nuxt
    registry --> astro
    registry --> svelte
    registry --> vue
    registry --> remix
    registry --> laravel
```
