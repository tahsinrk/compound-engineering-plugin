# Compound Engineering Plugin — Fork Customizations

## Fork Info
- **Origin:** `tahsinrk/compound-engineering-plugin` (forked from Every Inc.)
- **Original code preserved on:** `every-original` branch
- **Customization commit:** `3ea53b8` ("Adapt plugin for Python/TypeScript stack")

## What Changed

### Path Changes
- `docs/solutions/`, `docs/plans/`, `docs/brainstorms/` → `solutions/`, `plans/`, `brainstorms/`
- Output directories moved to project root instead of nested under `docs/`

### Agent Swaps
- Rails-specific reviewers (kieran-rails-reviewer, dhh-rails-reviewer) replaced with broadly-useful alternatives (kieran-python-reviewer, kieran-typescript-reviewer) as defaults in workflows
- Rails-specific examples in 8 agents generalized (ERB templates → generic templates, ActiveRecord → generic ORM, Turbo/Stimulus/Hotwire → generic frontend)

### Command/Test References
- `bin/rails test` and `bundle exec standardrb` replaced with generic test/linter references
- Works with Python (pytest), TypeScript (vitest/jest), or any stack

### Solution Doc Template
- Added `project:` field to YAML frontmatter (routes solutions by project name)

### What Was Preserved
- All agent structure, methodology, and non-Rails content
- All 29 agents, 5 workflows, 25 commands, 16 skills
- Context7 MCP bundling
- All review agent logic and scoring criteria

## How to Merge Upstream Updates
1. `git fetch upstream` (if upstream remote set) or download new release
2. Review changes — never auto-merge
3. Check if any customized files were modified upstream
4. Apply upstream changes manually, preserving our customizations
5. Test affected workflows
6. Commit with clear message about what was merged
