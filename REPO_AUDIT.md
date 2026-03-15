# gstack Technical Onboarding Report

This document provides a comprehensive technical walkthrough of the gstack repository for new contributors and developers looking to customize the system.

---

## Table of Contents

1. [Folder Structure](#1-folder-structure)
2. [Install Flow](#2-install-flow)
3. [Skill Implementation](#3-skill-implementation)
4. [Slash Command Registration](#4-slash-command-registration)
5. [How /browse, /qa, and /setup-browser-cookies Work](#5-how-browse-qa-and-setup-browser-cookies-work)
6. [Build Tools, Binaries, Scripts, and Dependencies](#6-build-tools-binaries-scripts-and-dependencies)
7. [Important Files for Customization](#7-important-files-for-customization)
8. [Risky Areas to Edit](#8-risky-areas-to-edit)
9. [How to Adapt This for Your Own Workflow](#9-how-to-adapt-this-for-your-own-workflow)

---

## 1. Folder Structure

```
gstack/                          # Root directory
├── browse/                      # Core headless browser tool (main feature)
│   ├── src/                     # TypeScript source files
│   │   ├── browser-manager.ts   # Chromium lifecycle, tabs, refs, dialogs
│   │   ├── buffers.ts           # Circular buffers for console/network/dialog logs
│   │   ├── cli.ts               # CLI wrapper that talks to persistent server
│   │   ├── commands.ts          # SINGLE SOURCE OF TRUTH for all commands
│   │   ├── config.ts            # Path resolution and state configuration
│   │   ├── cookie-import-browser.ts  # Cookie decryption from Chromium browsers
│   │   ├── cookie-picker-routes.ts   # HTTP routes for cookie picker UI
│   │   ├── cookie-picker-ui.ts       # React-like HTML UI for cookie picker
│   │   ├── find-browse.ts       # Helper to locate browse binary
│   │   ├── meta-commands.ts     # Meta commands (snapshot, screenshot, tabs)
│   │   ├── read-commands.ts     # Read-only commands (text, html, console)
│   │   ├── server.ts            # Bun HTTP server (daemon)
│   │   ├── snapshot.ts          # Accessibility tree + @ref system
│   │   └── write-commands.ts    # State-mutating commands (click, fill, goto)
│   ├── test/                    # Integration tests for browse tool
│   ├── dist/                    # Compiled binary (browse) - gitignored
│   ├── bin/                     # Utility scripts
│   ├── SKILL.md                 # Generated documentation (DO NOT EDIT)
│   └── SKILL.md.tmpl            # Template for SKILL.md (EDIT THIS)
│
├── qa/                          # /qa skill - systematic QA testing
│   ├── SKILL.md                 # Generated documentation
│   ├── SKILL.md.tmpl            # Template
│   ├── references/              # Issue taxonomy, guidelines
│   └── templates/               # QA report templates
│
├── setup-browser-cookies/       # /setup-browser-cookies skill
│   ├── SKILL.md                 # Generated documentation
│   └── SKILL.md.tmpl            # Template
│
├── ship/                        # /ship skill - automated PR workflow
│   ├── SKILL.md                 # Generated documentation
│   └── SKILL.md.tmpl            # Template
│
├── review/                      # /review skill - pre-landing review
│   ├── SKILL.md                 # Generated documentation
│   ├── SKILL.md.tmpl            # Template
│   ├── checklist.md             # Review checklist
│   ├── greptile-triage.md       # Greptile integration instructions
│   └── TODOS-format.md          # TODO format specification
│
├── plan-ceo-review/             # /plan-ceo-review skill
│   ├── SKILL.md                 # Generated documentation
│   └── SKILL.md.tmpl            # Template
│
├── plan-eng-review/             # /plan-eng-review skill
│   ├── SKILL.md                 # Generated documentation
│   └── SKILL.md.tmpl            # Template
│
├── retro/                       # /retro skill - retrospectives
│   ├── SKILL.md                 # Generated documentation
│   └── SKILL.md.tmpl            # Template
│
├── gstack-upgrade/              # /gstack-upgrade skill - auto-update
│   ├── SKILL.md                 # Generated documentation
│   └── SKILL.md.tmpl            # Template
│
├── scripts/                     # Build and development tooling
│   ├── gen-skill-docs.ts        # Template → SKILL.md generator
│   ├── skill-check.ts           # Health dashboard for all skills
│   ├── dev-skill.ts             # Watch mode for skill development
│   ├── eval-compare.ts          # Compare two eval runs
│   ├── eval-list.ts             # List all eval runs
│   ├── eval-summary.ts          # Aggregate stats across evals
│   └── eval-watch.ts            # Real-time eval monitoring
│
├── test/                        # Test infrastructure
│   ├── helpers/                 # Test utilities
│   │   ├── skill-parser.ts      # Parse $B commands from SKILL.md
│   │   ├── session-runner.ts    # Spawn claude -p for E2E tests
│   │   ├── llm-judge.ts         # LLM-as-judge evaluation
│   │   └── eval-store.ts        # Eval persistence and comparison
│   ├── fixtures/                # Test fixtures and baselines
│   ├── skill-validation.test.ts # Tier 1: Static validation (free)
│   ├── gen-skill-docs.test.ts   # Tier 1: Generator quality tests
│   ├── skill-e2e.test.ts        # Tier 2: E2E via claude -p (~$3.85)
│   └── skill-llm-eval.test.ts   # Tier 3: LLM judge (~$0.15)
│
├── bin/                         # Executable scripts
│   ├── dev-setup                # Set up development environment
│   ├── dev-teardown             # Clean up development environment
│   └── gstack-update-check      # Daily version check
│
├── setup                        # Main setup script (entry point)
├── package.json                 # NPM/Bun configuration
├── conductor.json               # Conductor workspace scripts
├── VERSION                      # Current version (0.3.8)
├── SKILL.md.tmpl                # Root skill template
├── SKILL.md                     # Root skill documentation
├── ARCHITECTURE.md              # Why gstack is built this way
├── BROWSER.md                   # Browser documentation
├── CHANGELOG.md                 # Version history
├── CLAUDE.md                    # Development instructions for Claude
├── CONTRIBUTING.md              # Contribution guidelines
└── .gitignore                   # Ignored files
```

### What Each Major Folder Does

| Folder | Purpose |
|--------|---------|
| `browse/` | Core headless browser - daemon server, CLI wrapper, all browser commands |
| `qa/` | Systematic QA testing skill with report templates |
| `ship/` | Automated ship workflow (test, review, version, changelog, PR) |
| `review/` | Pre-landing PR review with checklists |
| `plan-ceo-review/` | CEO-style plan review (strategy, priorities) |
| `plan-eng-review/` | Engineering plan review (architecture, tests, performance) |
| `retro/` | Retrospective workflow |
| `setup-browser-cookies/` | Cookie import from real browsers |
| `scripts/` | Build tools, doc generators, eval infrastructure |
| `test/` | Multi-tier test infrastructure |
| `bin/` | Utility scripts |

---

## 2. Install Flow

The install flow is orchestrated by the `setup` script at the repository root. Here's the complete flow from start to finish:

### Step 1: User Runs `./setup`

The setup script is a Bash script that:

```bash
#!/usr/bin/env bash
# gstack setup — build browser binary + register all skills with Claude Code
```

### Step 2: Detect gstack Directory

```bash
GSTACK_DIR="$(cd "$(dirname "$0")" && pwd)"
SKILLS_DIR="$(dirname "$GSTACK_DIR")"
BROWSE_BIN="$GSTACK_DIR/browse/dist/browse"
```

### Step 3: Smart Rebuild Detection

The script checks if a build is needed:

```bash
NEEDS_BUILD=0
if [ ! -x "$BROWSE_BIN" ]; then
  NEEDS_BUILD=1
elif [ -n "$(find "$GSTACK_DIR/browse/src" -type f -newer "$BROWSE_BIN" -print -quit 2>/dev/null)" ]; then
  NEEDS_BUILD=1
elif [ "$GSTACK_DIR/package.json" -nt "$BROWSE_BIN" ]; then
  NEEDS_BUILD=1
elif [ -f "$GSTACK_DIR/bun.lock" ] && [ "$GSTACK_DIR/bun.lock" -nt "$BROWSE_BIN" ]; then
  NEEDS_BUILD=1
fi
```

Build triggers:
- Binary doesn't exist
- Any source file in `browse/src` is newer than binary
- `package.json` is newer than binary
- `bun.lock` is newer than binary

### Step 4: Build the Binary

If needed, the script runs:

```bash
bun install
bun run build
```

The `bun run build` command (from `package.json`) does:

1. Generate SKILL.md files from templates: `bun run gen:skill-docs`
2. Compile CLI binary: `bun build --compile browse/src/cli.ts --outfile browse/dist/browse`
3. Compile find-browse helper: `bun build --compile browse/src/find-browse.ts --outfile browse/dist/find-browse`
4. Write version hash: `git rev-parse HEAD > browse/dist/.version`

### Step 5: Ensure Playwright Chromium

```bash
ensure_playwright_browser() {
  bun --eval 'import { chromium } from "playwright"; const browser = await chromium.launch(); await browser.close();'
}
```

If this fails, Playwright installs Chromium:

```bash
bunx playwright install chromium
```

### Step 6: Create Global State Directory

```bash
mkdir -p "$HOME/.gstack/projects"
```

### Step 7: Create Skill Symlinks

If running from inside `~/.claude/skills/`, the script creates symlinks for each skill:

```bash
for skill_dir in "$GSTACK_DIR"/*/; do
  if [ -f "$skill_dir/SKILL.md" ]; then
    skill_name="$(basename "$skill_dir")"
    ln -snf "gstack/$skill_name" "$SKILLS_DIR/$skill_name"
  fi
done
```

This creates symlinks like:
- `~/.claude/skills/browse` → `gstack/browse`
- `~/.claude/skills/qa` → `gstack/qa`
- etc.

### Step 8: Output Summary

```bash
echo "gstack ready."
echo "  browse: $BROWSE_BIN"
echo "  linked skills: ${linked[*]}"
```

### Development Setup (`bin/dev-setup`)

For local development, there's a separate script that:

1. Copies `.env` from main worktree if this is a git worktree
2. Runs `bun install`
3. Creates `.claude/skills/` inside the repo
4. Symlinks `.claude/skills/gstack` → repo root
5. Runs the regular `setup` script

---

## 3. Skill Implementation

Each gstack skill follows a consistent pattern:

### File Structure

```
skill-name/
├── SKILL.md           # Generated - DO NOT EDIT DIRECTLY
└── SKILL.md.tmpl      # Template - EDIT THIS
```

### SKILL.md.tmpl Format

Every skill template has:

1. **YAML Frontmatter** - Metadata Claude Code uses to discover the skill:

```yaml
---
name: skill-name
version: 1.0.0
description: |
  Multi-line description of what the skill does.
  Used by Claude Code for skill discovery.
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---
```

2. **Placeholders** - Auto-generated sections:

```markdown
{{UPDATE_CHECK}}     # Update check code block
{{BROWSE_SETUP}}     # Browse binary detection code
{{COMMAND_REFERENCE}} # Table of all commands
{{SNAPSHOT_FLAGS}}    # Snapshot flag documentation
```

3. **Skill-Specific Content** - Instructions, workflows, examples

### How Skills Are Generated

The `scripts/gen-skill-docs.ts` script:

1. Finds all `.tmpl` files in candidate paths
2. Resolves placeholders from source code metadata:
   - `COMMAND_REFERENCE` from `browse/src/commands.ts`
   - `SNAPSHOT_FLAGS` from `browse/src/snapshot.ts`
3. Injects a generated header: `<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->`
4. Writes the final `.md` file

### Wiring into Claude Code

Claude Code discovers skills by:

1. Looking in `~/.claude/skills/` for directories with `SKILL.md` files
2. Parsing the YAML frontmatter for metadata
3. Making the skill available as a slash command (e.g., `/browse`, `/qa`)

The symlinks created by `setup` point Claude Code to the actual skill directories inside gstack.

---

## 4. Slash Command Registration

### How Slash Commands Are Defined

Slash commands are **NOT hardcoded anywhere**. They are auto-discovered by Claude Code from:

1. **SKILL.md files** in `~/.claude/skills/*/SKILL.md`
2. **YAML frontmatter** with `name:` field

The `name:` in the frontmatter becomes the slash command:

```yaml
---
name: qa          # Creates /qa command
version: 1.0.0
---
```

### Current Slash Commands

| Command | Source | Purpose |
|---------|--------|---------|
| `/gstack` or `/browse` | `SKILL.md` + `browse/SKILL.md` | Root skill / browser commands |
| `/qa` | `qa/SKILL.md` | Systematic QA testing |
| `/setup-browser-cookies` | `setup-browser-cookies/SKILL.md` | Import browser cookies |
| `/ship` | `ship/SKILL.md` | Automated ship workflow |
| `/review` | `review/SKILL.md` | Pre-landing review |
| `/plan-ceo-review` | `plan-ceo-review/SKILL.md` | CEO-style plan review |
| `/plan-eng-review` | `plan-eng-review/SKILL.md` | Engineering plan review |
| `/retro` | `retro/SKILL.md` | Retrospectives |
| `/gstack-upgrade` | `gstack-upgrade/SKILL.md` | Update gstack |

### Adding a New Slash Command

1. Create a new directory: `mkdir my-skill`
2. Create `my-skill/SKILL.md.tmpl` with YAML frontmatter
3. Run `bun run gen:skill-docs` to generate `SKILL.md`
4. Run `./setup` to create the symlink

---

## 5. How /browse, /qa, and /setup-browser-cookies Work

### /browse - Core Browser Interaction

#### Architecture Overview

```
Claude Code                     gstack
─────────                      ──────
                               ┌──────────────────────┐
  Tool call: $B snapshot -i    │  CLI (compiled binary)│
  ─────────────────────────→   │  • reads state file   │
                               │  • POST /command      │
                               │    to localhost:PORT   │
                               └──────────┬───────────┘
                                          │ HTTP
                               ┌──────────▼───────────┐
                               │  Server (Bun.serve)   │
                               │  • dispatches command  │
                               │  • talks to Chromium   │
                               │  • returns plain text  │
                               └──────────┬───────────┘
                                          │ CDP
                               ┌──────────▼───────────┐
                               │  Chromium (headless)   │
                               │  • persistent tabs     │
                               │  • cookies carry over  │
                               │  • 30min idle timeout  │
                               └───────────────────────┘
```

#### Flow: First Command

1. **Claude runs**: `$B goto https://example.com`
2. **CLI** (`cli.ts`) reads `.gstack/browse.json` for server state
3. **State file missing** → CLI spawns `bun run server.ts` in background
4. **Server** (`server.ts`) starts:
   - Finds available port (10000-60000)
   - Launches headless Chromium via Playwright
   - Writes state file with `{ pid, port, token, binaryVersion }`
5. **CLI** sends HTTP POST to `http://localhost:PORT/command`
6. **Server** dispatches to `handleWriteCommand("goto", ["https://example.com"])`
7. **BrowserManager** calls `page.goto(url)`
8. **Response** returned as plain text

#### Flow: Subsequent Commands (~100ms)

1. **CLI** reads state file (already exists)
2. **Health check** → server is alive
3. **Send command** → get response

#### Command Dispatch

Commands are categorized in `commands.ts`:

```typescript
export const READ_COMMANDS = new Set([
  'text', 'html', 'links', 'forms', 'accessibility',
  'js', 'eval', 'css', 'attrs', 'console', 'network', ...
]);

export const WRITE_COMMANDS = new Set([
  'goto', 'back', 'forward', 'reload',
  'click', 'fill', 'select', 'hover', 'type', ...
]);

export const META_COMMANDS = new Set([
  'tabs', 'tab', 'newtab', 'closetab',
  'status', 'stop', 'restart',
  'screenshot', 'pdf', 'responsive', 'snapshot', ...
]);
```

Server dispatches to appropriate handler:

```typescript
if (READ_COMMANDS.has(command)) → handleReadCommand()
if (WRITE_COMMANDS.has(command)) → handleWriteCommand()
if (META_COMMANDS.has(command)) → handleMetaCommand()
```

#### The @ref System

The `snapshot` command builds clickable element references:

1. Call `page.locator('body').ariaSnapshot()` → accessibility tree
2. Parse tree, assign refs: `@e1`, `@e2`, `@e3`...
3. Build Playwright Locator for each ref: `getByRole(role, { name }).nth(index)`
4. Store `Map<string, Locator>` on BrowserManager
5. Return annotated tree as plain text

Later commands resolve refs:

```typescript
resolveRef("@e3") → returns stored Locator → locator.click()
```

### /qa - Systematic QA Testing

#### What It Does

1. **Diff-aware mode** (default on feature branches):
   - Analyzes `git diff main...HEAD` to find changed files
   - Identifies affected pages/routes
   - Tests each affected page

2. **Full mode**: Systematic exploration of entire application

3. **Quick mode**: 30-second smoke test

4. **Regression mode**: Compare against baseline

#### Internal Flow

1. **Setup**: Find browse binary, create output directories
2. **Authenticate** (if needed): Login via browser or import cookies
3. **Orient**: Map the application (`$B links`, `$B snapshot -i`)
4. **Explore**: Visit pages systematically, run checks
5. **Document**: Take screenshots, log console errors
6. **Wrap Up**: Compute health score, write report

#### Key Patterns

```bash
$B goto https://app.com
$B snapshot -i -a -o "$REPORT_DIR/screenshots/page.png"
$B console --errors
$B is visible ".main-content"
```

### /setup-browser-cookies - Cookie Import

#### What It Does

Imports logged-in sessions from real Chromium browsers (Comet, Chrome, Arc, Brave, Edge) into the headless Playwright session.

#### Internal Flow

1. **Detect browsers**: Check known paths for installed browsers
2. **Open picker UI**: Serve interactive HTML on browse server port
3. **User selects domains**: Click "+" in the picker
4. **Decrypt cookies**:
   - Read SQLite cookie database (copied to avoid locks)
   - Get decryption key from macOS Keychain
   - Decrypt with PBKDF2 + AES-128-CBC
5. **Load into Playwright**: `context.addCookies(decrypted)`

#### Security Model

- Keychain access requires user approval (first time per browser)
- Cookie values never written to disk in plaintext
- Database opened read-only
- Key cached only in memory for server lifetime

---

## 6. Build Tools, Binaries, Scripts, and Dependencies

### Runtime Dependencies

From `package.json`:

```json
{
  "dependencies": {
    "playwright": "^1.58.2",   // Browser automation
    "diff": "^7.0.0"           // Text diffing for snapshots
  }
}
```

### Dev Dependencies

```json
{
  "devDependencies": {
    "@anthropic-ai/sdk": "^0.78.0"  // For LLM evaluation tests
  }
}
```

### Required Tools

| Tool | Purpose | Install |
|------|---------|---------|
| **Bun** | JS runtime, bundler, test runner | `curl -fsSL https://bun.sh/install \| bash` |
| **Playwright** | Browser automation | Installed via `bun install` |
| **Chromium** | Headless browser | Installed via `bunx playwright install chromium` |
| **Git** | Version control, diff analysis | Pre-installed on most systems |

### Compiled Binaries

Located in `browse/dist/` (gitignored):

| Binary | Source | Purpose |
|--------|--------|---------|
| `browse` | `browse/src/cli.ts` | Main CLI wrapper |
| `find-browse` | `browse/src/find-browse.ts` | Helper to locate binary |
| `.version` | `git rev-parse HEAD` | Version hash for auto-restart |

### NPM Scripts

From `package.json`:

```json
{
  "scripts": {
    "build": "bun run gen:skill-docs && bun build --compile ...",
    "gen:skill-docs": "bun run scripts/gen-skill-docs.ts",
    "dev": "bun run browse/src/cli.ts",
    "server": "bun run browse/src/server.ts",
    "test": "bun test browse/test/ test/ --ignore ...",
    "test:evals": "EVALS=1 bun test test/skill-llm-eval.test.ts test/skill-e2e.test.ts",
    "test:e2e": "EVALS=1 bun test test/skill-e2e.test.ts",
    "skill:check": "bun run scripts/skill-check.ts",
    "dev:skill": "bun run scripts/dev-skill.ts",
    "eval:list": "bun run scripts/eval-list.ts",
    "eval:compare": "bun run scripts/eval-compare.ts",
    "eval:summary": "bun run scripts/eval-summary.ts"
  }
}
```

### Utility Scripts

| Script | Location | Purpose |
|--------|----------|---------|
| `setup` | Root | Main install script |
| `dev-setup` | `bin/` | Development environment setup |
| `dev-teardown` | `bin/` | Clean up development environment |
| `gstack-update-check` | `bin/` | Daily version check |

---

## 7. Important Files for Customization

### High Priority - Skill Behavior

| File | What to Customize |
|------|-------------------|
| `*/SKILL.md.tmpl` | Skill instructions, workflows, examples |
| `browse/src/commands.ts` | Add new browse commands |
| `browse/src/snapshot.ts` | Modify snapshot flags or @ref behavior |
| `review/checklist.md` | Pre-landing review criteria |
| `qa/templates/qa-report-template.md` | QA report format |
| `qa/references/issue-taxonomy.md` | Issue categorization |

### Medium Priority - System Configuration

| File | What to Customize |
|------|-------------------|
| `package.json` | Dependencies, scripts |
| `VERSION` | Current version number |
| `browse/src/config.ts` | State file paths, defaults |
| `browse/src/server.ts` | Server behavior, timeouts |
| `scripts/gen-skill-docs.ts` | Add new placeholders |

### Low Priority - Reference Only

| File | Purpose |
|------|---------|
| `ARCHITECTURE.md` | Why decisions were made |
| `CLAUDE.md` | Development guidelines |
| `CONTRIBUTING.md` | Contribution process |

### Adding a New Browse Command

1. **Define in `commands.ts`**:

```typescript
// Add to appropriate set
export const READ_COMMANDS = new Set([..., 'mycommand']);

// Add description
export const COMMAND_DESCRIPTIONS = {
  'mycommand': {
    category: 'Inspection',
    description: 'What it does',
    usage: 'mycommand <arg>'
  },
  ...
};
```

2. **Implement in appropriate handler** (`read-commands.ts`, `write-commands.ts`, or `meta-commands.ts`):

```typescript
case 'mycommand':
  const [arg] = args;
  const result = await page.evaluate(...);
  return result;
```

3. **Rebuild**: `bun run build`

### Adding a New Snapshot Flag

1. **Add to `SNAPSHOT_FLAGS` in `snapshot.ts`**:

```typescript
export const SNAPSHOT_FLAGS = [
  ...
  {
    short: '-x',
    long: '--myflag',
    description: 'What it does',
    optionKey: 'myFlag'
  },
];
```

2. **Add to `SnapshotOptions` interface**:

```typescript
interface SnapshotOptions {
  ...
  myFlag?: boolean;
}
```

3. **Handle in `handleSnapshot()`**

4. **Rebuild**: `bun run build`

---

## 8. Risky Areas to Edit

### ⚠️ High Risk - Can Break Everything

| File/Area | Risk | Why |
|-----------|------|-----|
| `browse/src/server.ts` | High | Core server - bugs crash all commands |
| `browse/src/browser-manager.ts` | High | Chromium lifecycle, ref system |
| `browse/src/cli.ts` | High | CLI wrapper - bugs prevent all usage |
| `setup` script | High | Install flow - bugs prevent setup |
| `commands.ts` validation | High | Load-time checks - errors prevent server start |

### ⚠️ Medium Risk - Can Break Features

| File/Area | Risk | Why |
|-----------|------|-----|
| `browse/src/snapshot.ts` | Medium | @ref system - bugs break element selection |
| `browse/src/cookie-import-browser.ts` | Medium | Cookie decryption - security sensitive |
| `scripts/gen-skill-docs.ts` | Medium | Doc generation - bugs cause stale docs |
| `SKILL.md.tmpl` files | Medium | AI instructions - bad docs = confused agent |

### ✅ Low Risk - Safe to Edit

| File/Area | Risk | Why |
|-----------|------|-----|
| `*/SKILL.md.tmpl` content | Low | Instructions only - won't break system |
| `review/checklist.md` | Low | Review criteria - no code impact |
| `qa/templates/*` | Low | Report templates - no code impact |
| `ARCHITECTURE.md`, `CONTRIBUTING.md` | Low | Documentation only |
| Test files | Low | Tests are isolated |

### Common Pitfalls

1. **Editing `SKILL.md` instead of `SKILL.md.tmpl`** - Changes will be overwritten on next build

2. **Adding command without description** - Server crashes on load due to validation:

```typescript
for (const cmd of allCmds) {
  if (!descKeys.has(cmd)) throw new Error(`COMMAND_DESCRIPTIONS missing entry for: ${cmd}`);
}
```

3. **Breaking state file format** - CLI can't find server, creates orphan processes

4. **Changing port selection** - Multi-workspace setups may conflict

5. **Modifying Keychain access** - Cookie import may fail silently

---

## 9. How to Adapt This for Your Own Workflow

### Option 1: Add a Custom Skill

Easiest path for adding new workflows:

1. **Create skill directory**:
```bash
mkdir ~/.claude/skills/gstack/my-workflow
```

2. **Create SKILL.md.tmpl**:
```yaml
---
name: my-workflow
version: 1.0.0
description: |
  What your workflow does.
allowed-tools:
  - Bash
  - Read
  - Write
---

# My Workflow

Instructions for Claude...
```

3. **Generate and link**:
```bash
cd ~/.claude/skills/gstack
bun run gen:skill-docs
./setup
```

4. **Use**: `/my-workflow` in Claude Code

### Option 2: Customize Existing Skills

For tweaking behavior:

1. **Fork the repo** or edit your local clone

2. **Edit `.tmpl` files** in the skill directories

3. **Rebuild**:
```bash
bun run build
```

4. **Test locally** before deploying

### Option 3: Add Custom Browse Commands

For new browser interactions:

1. **Add to `browse/src/commands.ts`**

2. **Implement in appropriate handler**

3. **Rebuild**: `bun run build`

4. **Templates auto-update** via `{{COMMAND_REFERENCE}}`

### Option 4: Create a Parallel Stack

For major customization:

1. **Fork the entire repo**

2. **Rename from "gstack" to your name**

3. **Update `setup` script** with new paths

4. **Replace skill content** entirely

5. **Deploy to your `~/.claude/skills/`**

### Tips for Customization

1. **Start with SKILL.md.tmpl changes** - Lowest risk, immediate impact

2. **Use the test tiers**:
   - `bun test` - Fast, catches most issues
   - `bun run test:e2e` - Full integration test (costs ~$4)

3. **Watch mode for iteration**:
```bash
bun run dev:skill  # Auto-regen + validate on change
```

4. **Check skill health**:
```bash
bun run skill:check  # Dashboard of all skills
```

5. **Keep VERSION updated** - Triggers auto-restart on update

6. **Document your changes** - Future you will thank present you

---

## Quick Reference

### Common Commands

```bash
# Development
bun install                    # Install dependencies
bun run build                  # Full build (docs + binaries)
bun run gen:skill-docs         # Regenerate SKILL.md files only
bun run dev goto https://...   # Run CLI in dev mode

# Testing
bun test                       # Run free tests
bun run test:e2e               # Run E2E tests (~$4)
bun run skill:check            # Skill health dashboard

# Debugging
$B status                      # Server health check
$B console                     # View console logs
$B network                     # View network requests
$B stop                        # Stop server
$B restart                     # Restart server
```

### Key File Locations

| Purpose | Location |
|---------|----------|
| State file | `.gstack/browse.json` |
| Console log | `.gstack/browse-console.log` |
| Network log | `.gstack/browse-network.log` |
| Global state | `~/.gstack/` |
| Eval results | `~/.gstack-dev/evals/` |

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `BROWSE_PORT` | Override server port |
| `BROWSE_IDLE_TIMEOUT` | Override idle timeout (ms) |
| `BROWSE_STATE_FILE` | Override state file path |
| `ANTHROPIC_API_KEY` | Required for eval tests |
| `EVALS` | Set to `1` to run paid eval tests |

---

*Generated for gstack v0.3.8*
