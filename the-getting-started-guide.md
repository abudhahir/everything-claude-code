# Getting Started with Everything Claude Code

> A practical introduction to what ECC is, how it's structured, and how to install it.

---

## What Is Everything Claude Code?

Everything Claude Code (ECC) is a **Claude Code plugin** — a collection of production-ready components that enhance how Claude Code works. It is not just configuration files. It is a complete development system built over 10+ months of intensive daily use building real products.

It provides:

- **Agents** — Specialized subagents you can delegate tasks to (planner, code-reviewer, tdd-guide, security-reviewer, etc.)
- **Skills** — Workflow definitions and domain knowledge that guide how Claude approaches tasks
- **Commands** — Slash commands you invoke directly (`/tdd`, `/plan`, `/e2e`, `/code-review`, `/build-fix`, etc.)
- **Hooks** — Trigger-based automations that fire on tool calls and lifecycle events
- **Rules** — Always-follow guidelines injected into every session (security, coding style, testing requirements)
- **MCP Configs** — Pre-configured MCP server integrations for external tools

It works across **Claude Code**, **Codex**, **Cowork**, and other AI agent harnesses.

---

## Project Structure

```
everything-claude-code/
├── agents/          # Subagent definitions (planner, tdd-guide, code-reviewer, etc.)
├── skills/          # Workflow and domain knowledge files
├── commands/        # Slash command definitions
├── hooks/           # Event-triggered automations
├── rules/           # Always-on coding standards and guidelines
│   ├── common/      # Language-agnostic principles (always installed)
│   ├── typescript/  # TypeScript/JavaScript specific
│   ├── python/      # Python specific
│   ├── golang/      # Go specific
│   ├── swift/       # Swift specific
│   └── php/         # PHP specific
├── mcp-configs/     # MCP server configurations
├── scripts/         # Node.js installer utilities
└── tests/           # Test suite for scripts and utilities
```

---

## Installing ECC

ECC is published to npm as `ecc-universal`. There are two ways to run the installer — both are equivalent and support all the same flags:

| Method | When to use |
|--------|-------------|
| `npx ecc` | You have not cloned the repo. Runs the latest published version from npm, no setup required. |
| `./install.sh` | You have the repo cloned locally. The shell script resolves symlinks and delegates to the same Node.js runtime. |

```bash
# Via npm (no clone needed)
npx ecc typescript

# Via local clone
./install.sh typescript
```

All examples in this guide show `npx ecc`. Replace with `./install.sh` if you are working from a local clone.

### Quick Start — Language Mode

The simplest way to get started. Pass one or more language names:

```bash
# Via npm
npx ecc typescript
npx ecc python
npx ecc golang typescript   # multiple languages at once

# Via local clone
./install.sh typescript
./install.sh python
./install.sh golang typescript
```

This installs:
- `rules/common/` — universal coding standards, security, testing, git workflow
- `rules/<language>/` — language-specific rules for each language you specify

Files land in `~/.claude/rules/` by default.

---

## Install Modes

### 1. Language Mode (Legacy)

```bash
npx ecc typescript python
```

The original install method. Pass language names directly as positional arguments. Installs `rules/common/` plus the rules directory for each language specified. Best for most individual developers.

**Available languages:** typescript, python, golang, swift, php (and others — run `npx ecc --help` to see the full list)

---

### 2. Profile Mode

```bash
npx ecc --profile <name>
npx ecc --profile developer --with security --without orchestration
```

Installs a **named bundle** of modules defined in the ECC manifests. A profile is a pre-curated set of components (agents, skills, rules, hooks) grouped under a logical name. You do not need to know which individual modules to select — the profile handles that.

#### Available Profiles

| Profile | Description | Includes |
|---------|-------------|----------|
| `core` | Minimal baseline | rules, agents, commands, hooks, platform configs, quality workflow |
| `developer` | Default for most engineers | everything in `core` + language frameworks, database skills, orchestration |
| `security` | Security-heavy setup | everything in `core` + security review, scanning, and framework-specific security guidance |
| `research` | Research & content workflows | everything in `core` + research APIs, business content, social distribution |
| `full` | Complete install — everything | all modules across all categories |

**Important:** Profiles are **role-oriented**, not language-specific. There is no "python-backend" or "java-typescript" profile. For language-specific rules, combine profile mode with language mode (see below) or use `--modules` to handpick exactly what you need.

#### Language-Specific Setups

For a Python backend developer:
```bash
# Install developer profile + Python rules
npx ecc --profile developer python
```

For a Java + TypeScript developer:
```bash
# Install developer profile + Java and TypeScript rules
npx ecc --profile developer typescript
# Java rules are covered by the framework-language module inside developer profile
# (includes springboot-patterns, java-coding-standards, jpa-patterns)
```

For a Go developer who also wants security:
```bash
npx ecc --profile security golang
```

For maximum control, use `--modules` to pick exactly what you need:
```bash
# Python backend: core modules + framework skills + database skills
npx ecc --modules rules-core,agents-core,commands-core,hooks-runtime,platform-configs,workflow-quality,framework-language,database python
```

Customize profiles with:
- `--with <component>` — add extra components on top of the profile
- `--without <component>` — exclude a specific component from the profile

---

### 3. Modules Mode

```bash
npx ecc --modules id1,id2,id3
```

Installs specific, explicit module IDs you provide. More granular than profiles — you handpick exactly what you want. Useful when you know precisely which components are needed and want full control over the install.

Also supports `--with` / `--without` overrides.

---

### 4. Config Mode

```bash
npx ecc --config ecc-install.json
```

Reads install intent from a JSON file instead of CLI flags. Useful for:
- Repeatable installs (CI/CD, team onboarding scripts)
- Storing your install configuration in version control
- Complex installs that are too verbose for a single command line

---

## Install Targets

By default, everything installs into your home directory under `~/.claude/`. You can change the target with `--target`:

| Target | Scope | Install Root | Notes |
|--------|-------|--------------|-------|
| `claude` (default) | global/home | `~/.claude/` | Rules land in `~/.claude/rules/`. State tracked in `~/.claude/ecc/install-state.json` |
| `cursor` | project-local | `./.cursor/` | Rules → `.cursor/rules/`. Other assets (agents, hooks, skills, commands) copied alongside. State in `.cursor/ecc-install-state.json` |
| `antigravity` | project-local | `./.agent/` | Rules → `.agent/rules/`. Commands remapped to `.agent/workflows/`. Agents remapped to `.agent/skills/`. State in `.agent/ecc-install-state.json` |
| `codex` | global/home | `~/.codex/` | For OpenAI Codex agent harness. State in `~/.codex/ecc-install-state.json` |
| `opencode` | global/home | `~/.opencode/` | For the opencode agent harness. State in `~/.opencode/ecc-install-state.json` |

**home** targets install once globally and apply to all projects. **project-local** targets install into the current working directory and are scoped to that project only.

---

## Utility Flags

These flags work with any install mode:

| Flag | Effect |
|------|--------|
| `--dry-run` | Preview the file operations without copying anything |
| `--json` | Emit machine-readable JSON output instead of human-readable text |
| `--help` | Show usage information and available languages |

Example — preview what would be installed without touching any files:

```bash
npx ecc --dry-run typescript python
npx ecc --dry-run --profile developer --json
```

---

## How the Installer Works Internally

The install pipeline has three layers:

1. **`install.sh`** — Resolves symlinks to find the real script directory (so it works when installed as an npm bin), then immediately delegates to Node: `exec node scripts/install-apply.js "$@"`

2. **`scripts/install-apply.js`** — The CLI runtime. Parses your arguments, normalizes the install request (merging CLI flags and any config file), builds an install plan (resolves which files go where), then applies it by copying files.

3. **Manifests and adapters** — The plan is built by looking up module definitions in `manifests/` and routing through the appropriate target adapter (`claude`, `cursor`, or `antigravity`).

---

## Key Commands After Install

Once installed, these slash commands are available in Claude Code:

| Command | Purpose |
|---------|---------|
| `/tdd` | Test-driven development workflow |
| `/plan` | Implementation planning |
| `/e2e` | Generate and run E2E tests |
| `/code-review` | Quality review |
| `/build-fix` | Fix build errors |
| `/learn` | Extract patterns from sessions |
| `/skill-create` | Generate skills from git history |

---

## Scenario-Based Installation Guide

The right install command depends on your situation. Use the scenarios below as a starting point.

---

### Scenario 1: Starting a New Project (Solo Developer)

You are beginning a fresh codebase and want Claude Code to follow consistent standards from day one.

**Step 1 — Install ECC globally once** (if you haven't already):
```bash
npx ecc --profile developer typescript   # adjust language to match your stack
```
This puts rules, agents, commands, and hooks into `~/.claude/` where they apply to every project you open.

**Step 2 — Preview what will be installed before committing:**
```bash
npx ecc --dry-run --profile developer typescript
```

**Step 3 — Open your project in Claude Code and start using commands:**
```
/plan      → plan the feature before writing code
/tdd       → write tests first, then implement
/code-review → review what you've written
```

---

### Scenario 2: Onboarding a Team to a Shared Project

You want everyone on the team to use the same ECC setup without each person running manual install commands.

**Step 1 — Create an `ecc-install.json` in your repo root:**
```json
{
  "profile": "developer",
  "languages": ["typescript"],
  "target": "claude"
}
```

**Step 2 — Add the install step to your onboarding script or `Makefile`:**
```bash
# In Makefile or setup.sh
ecc-install:
    npx ecc --config ecc-install.json
```

**Step 3 — Each developer runs once after cloning:**
```bash
make ecc-install
# or
npx ecc --config ecc-install.json
```

Everyone gets an identical setup. When you update the config, the team re-runs to sync.

---

### Scenario 3: Adding ECC to an Existing Project

You have an existing codebase and want to introduce ECC without disrupting current workflows.

**Start minimal — avoid overwhelming an existing project:**
```bash
npx ecc --profile core typescript
```

`core` gives you rules, agents, commands, hooks, and quality workflow without the heavier skill bundles. Expand once the team is comfortable.

**Preview first to understand what changes:**
```bash
npx ecc --dry-run --profile core typescript
# or: ./install.sh --dry-run --profile core typescript
```

**Expand incrementally once settled:**
```bash
npx ecc --profile developer typescript   # add framework skills and database skills
# or: ./install.sh --profile developer typescript
```

---

### Scenario 4: Working in Cursor

You use Cursor as your editor and want ECC rules and configs scoped to your project (not global).

```bash
cd your-project
npx ecc --target cursor --profile developer typescript
# or: ./install.sh --target cursor --profile developer typescript
```

This installs into `./.cursor/` inside your project directory. Rules, hooks, agents, and MCP configs are all scoped to this project only. Commit the `.cursor/` directory to share the setup with your team.

---

### Scenario 5: Security-Focused Project

You are working on a product where security review is a first-class concern — auth systems, payment flows, infrastructure code.

```bash
npx ecc --profile security python
# or: ./install.sh --profile security python
```

This adds the full security module on top of core: security review skills, framework-specific security guidance (Django, Spring Boot, Laravel), scanning tools, and the security guide.

Use `/code-review` after every meaningful change — the security profile wires in the security-reviewer agent automatically.

---

### Scenario 6: Research or Content Work

You are using Claude Code for investigation, synthesis, writing, or publishing — not primarily for software development.

```bash
npx ecc --profile research
# or: ./install.sh --profile research
```

This installs research APIs (Claude API, Exa search, deep research skills), business content skills (article writing, investor materials, market research), and social distribution (crosspost, X API) on top of the core baseline.

---

### Scenario 7: Building AI Agents or LLM Pipelines

You are building agentic systems, autonomous loops, or LLM-powered pipelines and need patterns beyond standard app development.

```bash
npx ecc --profile full --without supply-chain-domain --without document-processing
# or: ./install.sh --profile full --without supply-chain-domain --without document-processing
```

`full` includes the `agentic-patterns` module which covers agent harness construction, autonomous loops, cost-aware LLM pipelines, Claude DevFleet, and more. Exclude domain modules that are irrelevant to your work to keep the install lean.

Or install only what you need explicitly:
```bash
npx ecc --modules rules-core,agents-core,commands-core,hooks-runtime,platform-configs,workflow-quality,agentic-patterns,orchestration
# or: ./install.sh --modules rules-core,agents-core,commands-core,hooks-runtime,platform-configs,workflow-quality,agentic-patterns,orchestration
```

---

### Scenario 8: Multiple Harnesses (Claude Code + Codex)

You switch between Claude Code and OpenAI Codex and want consistent standards across both.

```bash
# Install for Claude Code
npx ecc --profile developer typescript
# or: ./install.sh --profile developer typescript

# Install for Codex
npx ecc --target codex --profile developer typescript
# or: ./install.sh --target codex --profile developer typescript
```

Each target gets its own install state file, so they are tracked independently. Both pull from the same source modules.

---

### Quick Reference — Scenario to Command

| Situation | Recommended Command |
|-----------|-------------------|
| New project, solo dev | `npx ecc --profile developer <language>` or `./install.sh --profile developer <language>` |
| Team onboarding | `npx ecc --config ecc-install.json` or `./install.sh --config ecc-install.json` |
| Existing project, minimal disruption | `npx ecc --profile core <language>` or `./install.sh --profile core <language>` |
| Cursor editor, project-scoped | `npx ecc --target cursor --profile developer <language>` or `./install.sh --target cursor --profile developer <language>` |
| Security-critical codebase | `npx ecc --profile security <language>` or `./install.sh --profile security <language>` |
| Research / content work | `npx ecc --profile research` or `./install.sh --profile research` |
| Building AI agents / LLM pipelines | `npx ecc --profile full --without supply-chain-domain --without document-processing` or `./install.sh --profile full ...` |
| Multiple harnesses | Run once per target: `--target claude`, `--target codex` |

---

## Next Steps

- Read `the-shortform-guide.md` for skills, hooks, subagents, and daily workflow
- Read `the-longform-guide.md` for the full system design and philosophy
- See `CONTRIBUTING.md` for how to add your own agents, skills, and commands
- Run `npx ecc --help` or `./install.sh --help` to see all available options and languages
