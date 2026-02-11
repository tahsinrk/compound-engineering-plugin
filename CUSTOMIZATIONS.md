# Customizations to Compound Engineering Plugin

This fork adapts Every's compound-engineering plugin for a Python/TypeScript/GCP stack. The original plugin was built for Every's Rails/Ruby stack and references Rails-specific tools, patterns, and file conventions throughout.

**Fork owner:** tahsinrk
**Upstream:** [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin)
**Date of initial customization:** 2026-02-11

---

## Branch Strategy

| Branch | Purpose | When to use |
|--------|---------|-------------|
| `main` | Customized version with all adaptations below | Default. Install from this. |
| `every-original` | Every's unmodified code, frozen at time of fork | Reference only. Compare against this to see what changed. Never push to this. |

**Remotes:**

| Remote | Points to | Purpose |
|--------|-----------|---------|
| `origin` | `github.com/tahsinrk/compound-engineering-plugin` | Your fork. Push here. |
| `upstream` | `github.com/EveryInc/compound-engineering-plugin` | Every's original. Pull selective updates from here. |

**Why this structure exists:** Every will continue improving their plugin. The `upstream` remote lets you pull their changes selectively. The `every-original` branch lets you compare what you changed vs. what they shipped. If Every adds a new agent or improves a workflow, you can `git fetch upstream` and cherry-pick specific commits without losing your customizations.

**How to sync upstream changes:**

```bash
git fetch upstream
git diff upstream/main main -- path/to/specific/file  # See what they changed
git checkout upstream/main -- path/to/specific/file     # Pull a specific file
# Or cherry-pick a specific commit:
git cherry-pick <commit-hash>
```

---

## Path Convention Changes

Every's plugin stores documentation artifacts under a `docs/` prefix. This fork removes that prefix so artifacts sit at the project root.

| Original path | Our path | Why |
|---------------|----------|-----|
| `docs/solutions/[category]/` | `solutions/[category]/` | We don't use a `docs/` directory. Solutions, plans, and brainstorms are top-level project artifacts, not documentation subfolders. Shorter paths, less nesting. |
| `docs/plans/` | `plans/` | Same reason. |
| `docs/brainstorms/` | `brainstorms/` | Same reason. |

**Scope of this change:** This path replacement was applied across **all 28 modified files** in the plugin. Every agent, command, and skill that referenced `docs/solutions/`, `docs/plans/`, or `docs/brainstorms/` was updated.

**Verification:** After the bulk replacement, `grep -r "docs/solutions\|docs/plans\|docs/brainstorms"` returned zero matches across the entire plugin directory.

---

## Agent Reference Swaps

Every's plugin references three Rails-specific reviewer agents throughout its workflow commands. Since we don't use Rails, these were swapped to language-appropriate equivalents:

| Original agent | Replaced with | Reason |
|----------------|---------------|--------|
| `kieran-rails-reviewer` | `kieran-python-reviewer` | Python is our primary backend language. The Python reviewer agent already exists in the plugin. |
| `dhh-rails-reviewer` | `kieran-typescript-reviewer` | TypeScript is our frontend/secondary language. The TypeScript reviewer agent already exists in the plugin. |
| `rails-turbo-expert` | Removed (no replacement) | Turbo is a Rails-specific frontend framework. We don't use it and there's no equivalent needed. |

**Where these swaps occurred:**
- `commands/workflows/review.md` (the main review orchestrator)
- `commands/workflows/work.md` (the work execution workflow)
- `commands/workflows/compound.md` (the documentation workflow)
- `commands/technical_review.md` (standalone technical review command)

---

## Modified Agents (8 files)

These agents had Rails-specific content that was adapted to be stack-agnostic or Python/TypeScript-oriented. Changes were minimal and targeted: only Rails-specific references were modified. The agent's core logic, review philosophy, and structure were preserved.

### 1. `agents/design/figma-design-sync.md`

**What it does:** Synchronizes Figma designs to code, enforcing component width/padding conventions.

**What changed:**
- All `ERB` references → `HTML/template`. ERB is Rails' HTML templating language. Since we don't use Rails, examples now use plain HTML.
- `<%= render SomeComponent.new(...) %>` (Rails component syntax) → `<!-- Component content here -->` (generic HTML comment)
- File extension references changed from `.html.erb` to generic `.html`

**Why:** The design-to-code patterns are universal (full-width components, wrapper-level padding). Only the template syntax was Rails-specific.

### 2. `agents/research/best-practices-researcher.md`

**What it does:** Researches industry best practices for a given technical topic, maps them to your stack.

**What changed:**
- Framework/tool skill mappings updated: Rails/Ruby references → Python/TypeScript equivalents
- Example skill lookups changed from `rails`, `hotwire`, `turbo` → `python`, `fastapi`, `typescript`

**Why:** The agent needs to know what frameworks to research. Pointing it at Rails frameworks when you use Python/TypeScript would produce irrelevant results.

### 3. `agents/research/framework-docs-researcher.md`

**What it does:** Looks up framework documentation for the specific versions you're running.

**What changed:**
- `bundle show` (Ruby's dependency version checker) → `pip show` (Python) and `npm list` (TypeScript/Node)
- `Gemfile.lock` (Ruby dependency lockfile) → generic "dependency files" / `requirements.txt` / `package-lock.json`
- Framework documentation sources updated to reference Python/TypeScript ecosystems

**Why:** The agent checks your installed dependency versions and fetches docs for those specific versions. It needs to know your package manager commands to do that.

### 4. `agents/review/deployment-verification-agent.md`

**What it does:** Creates Go/No-Go deployment checklists with verification queries and rollback procedures.

**What changed:**
- `rails db:migrate` → generic "run database migrations" / framework-agnostic migration commands
- Rails-specific deployment steps → generic deployment verification steps

**Why:** Database migrations exist in every stack, but the commands differ. The verification logic (check before deploy, check after deploy, have a rollback plan) is universal.

### 5. `agents/review/julik-frontend-races-reviewer.md`

**What it does:** Reviews JavaScript code for race conditions, timing issues, and DOM lifecycle problems. Written as the persona "Julik," a senior developer with expertise in UI data races.

**What changed:**
- Removed Turbo/Stimulus/Hotwire-specific references (these are Rails' frontend frameworks)
- Changed to framework-agnostic language: "SPA routers, HTMX, or similar navigation systems"
- Kept all race condition patterns, cancellation token examples, and state machine recommendations intact

**Why:** The race condition expertise is universal and excellent. Only the framework name-drops needed updating. The core advice (cancel tokens, state machines, event listener cleanup, animation frame management) applies to any JavaScript frontend.

### 6. `agents/review/performance-oracle.md`

**What it does:** Analyzes code for performance issues (N+1 queries, missing indexes, unnecessary allocations).

**What changed:**
- `ActiveRecord` (Rails' ORM / database query layer) → generic "ORM" references
- Rails-specific query patterns → ORM-agnostic patterns

**Why:** N+1 queries and missing indexes exist in every ORM. The detection patterns are the same; only the class names differ.

### 7. `agents/review/security-sentinel.md`

**What it does:** Security-focused code review agent that checks for vulnerabilities.

**What changed:**
- Rails-specific grep patterns (searching for `html_safe`, `raw`, `permit`) → generic patterns plus Python-specific ones (`eval(`, `exec(`, `subprocess.shell=True`, `pickle.loads`)
- Rails CSRF/session patterns → framework-agnostic security checks

**Why:** Security review needs to know what dangerous patterns look like in your language. Rails' `html_safe` is irrelevant; Python's `eval()` is relevant.

### 8. `agents/workflow/bug-reproduction-validator.md`

**What it does:** Helps reproduce bugs systematically by setting up the environment and verifying symptoms.

**What changed:**
- `"For Rails apps"` section header → generic language
- Removed assumption that bug reproduction always involves `rails console` or `rails server`

**Why:** Minor change. The reproduction methodology is universal; only the "start the app" commands were Rails-specific.

---

## Modified Workflow Commands (4 files)

These are the four core workflow commands that form the Plan → Work → Review → Compound loop.

### 1. `commands/workflows/plan.md`

**What changed:**
- All `docs/plans/` paths → `plans/`
- Rails file structure examples (e.g., `app/models/user.rb`, `app/controllers/`) → Python equivalents (`src/models/user.py`, `src/api/`)
- `bin/rails test` → "project's test command"

**Why:** The plan command generates a plan file and references file paths as examples. Those examples need to match your actual project structure.

### 2. `commands/workflows/work.md`

**What changed:**
- `bin/rails test` → "project's test command" (the work command runs tests as verification)
- `kieran-rails-reviewer` → `kieran-python-reviewer` (the work command invokes a reviewer after code changes)
- All `docs/` path prefixes removed

**Why:** The work command executes a plan and verifies the result. It needs to know how to run your tests and which reviewer to invoke.

### 3. `commands/workflows/review.md`

**What changed:**
- Agent list updated: `kieran-rails-reviewer` → `kieran-python-reviewer`, `dhh-rails-reviewer` → `kieran-typescript-reviewer`
- Removed `rails-turbo-expert` from the parallel agent list entirely
- All `docs/` path prefixes removed

**Why:** The review command launches 10-15 agents in parallel to review code. The agent roster needs to match your stack.

### 4. `commands/workflows/compound.md`

**What changed:**
- All `docs/solutions/` paths → `solutions/`
- Added `project:` field to YAML frontmatter instructions (see Solution Documentation section below)
- `kieran-rails-reviewer` → `kieran-python-reviewer` in the optional enhancement agents list
- Solution doc category examples preserved as-is (they're language-agnostic)

**Why:** The compound command documents solutions. Path changes and the `project:` field addition are the main changes.

---

## Other Modified Commands (4 files)

### `commands/generate_command.md`
- `bin/rails test` → "use the project's test command"

### `commands/technical_review.md`
- `dhh-rails-reviewer` + `kieran-rails-reviewer` → `kieran-python-reviewer` + `kieran-typescript-reviewer`

### `commands/deepen-plan.md`
- `docs/plans/` → `plans/` path updates

### `commands/resolve_todo_parallel.md`
- `docs/` path prefix removed from solution references

---

## Modified Skills (6 files)

These are all path-only changes (`docs/solutions/` → `solutions/`, etc.). No logic changes.

| File | Change |
|------|--------|
| `skills/compound-docs/SKILL.md` | Path updates throughout |
| `skills/compound-docs/assets/critical-pattern-template.md` | Path updates |
| `skills/compound-docs/assets/resolution-template.md` | Path updates |
| `skills/compound-docs/references/yaml-schema.md` | Path updates, solution template paths |
| `skills/brainstorming/SKILL.md` | `docs/brainstorms/` → `brainstorms/` |
| `skills/document-review/SKILL.md` | Path updates |
| `skills/orchestrating-swarms/SKILL.md` | Path updates |

---

## Solution Documentation Design

The plugin's compound workflow creates solution documents when problems are solved. Here's how we configured ours:

**Location:** `solutions/[category]/` at the project root (or at `~/Dropbox/Claude Code/solutions/` for cross-project solutions).

**YAML frontmatter format:**
```yaml
---
category: runtime-errors
problem: Brief description of the problem
project: billbuddy          # or 'cross-project' if generally applicable
date: 2026-02-11
tags: [python, gcp, cloud-run]
---
```

**The `project:` field** was added to Every's original format. Every is a single-product company, so they didn't need it. We work across multiple projects (BillBuddy, Priority Engine, Vital Shift, etc.), so solution docs need to indicate which project they originated from.

**Categories:**
- `gcp/`, `claude-code/`, `python/`, `typescript/`, `deployment/`, `mcp/`, `skills/`, `database/`, `security/`, `ui-bugs/`, `integration/`, `performance/`
- Create new categories as needed.

**Body sections:** Problem, Cause, Fix, Prevention.

---

## Agents NOT Modified

The following agents were left unchanged because they are either language-agnostic or their domain expertise applies regardless of stack:

**Fully language-agnostic (no changes needed):**
- `code-simplicity-reviewer.md` (path-only update)
- `code-philosopher.md`
- `architecture-strategist.md`
- `pattern-recognition-specialist.md`
- `agent-native-reviewer.md`
- `dependency-detective.md`

**Domain-specific but not Rails-specific:**
- `data-integrity-guardian.md` (database patterns, SQL-focused)
- `data-migration-expert.md` (migration safety)
- `devops-harmony-analyst.md` (CI/CD, infrastructure)

**Rails-heavy agents left as-is (on `every-original` branch if needed):**
- `kieran-rails-reviewer.md` - Exists in the plugin but won't be invoked since workflow commands now reference `kieran-python-reviewer` instead
- `dhh-rails-reviewer.md` - Same situation
- `rails-turbo-expert.md` - Same situation

These Rails agents were **not deleted** from the plugin files. They still exist in the `agents/` directory. They simply aren't referenced by any workflow command anymore. If you ever work on a Rails project, you could re-add them to the review command's agent list.

---

## Installation on a Fresh Machine

```bash
# 1. Clone the fork (not Every's original)
git clone https://github.com/tahsinrk/compound-engineering-plugin.git /tmp/compound-engineering-plugin

# 2. Set up the upstream remote (for future sync)
cd /tmp/compound-engineering-plugin
git remote add upstream https://github.com/EveryInc/compound-engineering-plugin.git

# 3. Install in Claude Code
#    (run these inside Claude Code, not in a regular terminal)
/plugin marketplace add https://github.com/tahsinrk/compound-engineering-plugin
/plugin install compound-engineering

# 4. Restart Claude Code to load the plugin
```

After installation, you'll have access to:
- `/workflows:plan` - Structured planning
- `/workflows:work` - Plan execution with task tracking
- `/workflows:review` - Multi-agent code review
- `/workflows:compound` - Solution documentation

---

## Summary of All Changes

**28 files modified** across the plugin. Breakdown:

| Category | Files | Type of change |
|----------|-------|----------------|
| Agents (substantive) | 8 | Rails code/patterns → Python/TypeScript/generic |
| Workflow commands | 4 | Paths, agent references, test commands |
| Other commands | 4 | Paths, agent references |
| Skills | 7 | Path-only updates |
| Plugin CLAUDE.md | 1 | Path update |
| **Total** | **28** | **206 insertions, 202 deletions** |

No files were added or deleted. All changes were modifications to existing files. The original unmodified versions are preserved on the `every-original` branch.
