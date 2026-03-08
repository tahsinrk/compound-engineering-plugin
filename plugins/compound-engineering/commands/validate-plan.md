---
name: validate-plan
description: Validate a plan against reality before building. Catches wrong assumptions about user workflow, existing infrastructure, requirement-architecture mismatches, and unstated assumptions. Run after /workflows:plan or after any plan revision.
argument-hint: "[path to plan file]"
---

# Validate Plan — External Reality Check

## Introduction

**Note: The current year is 2026.** Use this when dating validations and searching for recent documentation.

This command validates a plan against external reality — not whether the plan is internally consistent (SpecFlow does that), but whether it matches how the user actually lives, what infrastructure already exists, and whether the architecture delivers what the requirements promise.

**Why this exists:** Plans can pass technical review while being fundamentally wrong for the user. A plan that's architecturally coherent but assumes the wrong device, interface, or infrastructure is worse than no plan — it wastes build time and creates sunk-cost momentum that makes the errors harder to fix.

## Plan File

<plan_path> #$ARGUMENTS </plan_path>

**If the plan path above is empty:**
1. Check for recent plans: `ls -la plans/`
2. Ask the user: "Which plan would you like to validate? Please provide the path (e.g., `plans/2026-01-15-feat-my-feature-plan.md`)."

Do not proceed until you have a valid plan file path.

## Step 0: Read the Plan

Read the entire plan file. Identify:
- All acceptance criteria / requirements
- The proposed architecture (components, data flow, interfaces)
- Any infrastructure the plan proposes to build or use
- The intended user and their interaction pattern

## Step 1: Requirements-Architecture Cross-Check

For every acceptance criterion or requirement in the plan, trace the **specific mechanism** in the architecture that delivers it.

**Format:**

| Requirement | Mechanism in Architecture | Status |
|---|---|---|
| [Acceptance criterion text] | [Component/flow that delivers it] | TRACED / MISSING / CONTRADICTED |

**Rules:**
- If a requirement says "automatically," the mechanism must not involve manual human action
- If a requirement says "tracks" or "records," there must be a write path to persistent storage
- If a requirement says "notifies" or "alerts," there must be a delivery channel specified
- "CONTRADICTED" means the architecture actively prevents the requirement (e.g., read-only agent when requirement says "writes automatically")

**If any requirement is MISSING or CONTRADICTED, stop and report. The plan needs revision before proceeding.**

## Step 2: User Context Model

Answer these five questions about the plan's intended user:

1. **Where is the user physically** when they interact with this system? (desk, commuting, between appointments, in bed, walking)
2. **What device are they on?** (laptop with terminal, phone with apps, tablet)
3. **How much time do they have** per interaction? (30 seconds, 2 minutes, 10 minutes, unlimited)
4. **What app is already open?** (Slack, email, browser, Claude Code, terminal)
5. **What's the minimum interaction to get value?** (one tap, a short reply, a 5-minute review session)

**Check CLAUDE.md and any user profile files** (e.g., `self-map/tahsin-profile.md`) for context about the user's daily routine, work patterns, and preferences.

### Day in the Life Narrative

Write a short narrative (5-10 sentences) walking through the plan from the user's actual daily experience. Use specific times, locations, and devices.

Example format:
> "It's 8:15 AM Tuesday. [User] is [location]. Their phone buzzes — [notification]. They open [app], see [content]. They [action]. [Time passes]. They get [response]. They [next action]."

**Review the narrative for breaks:**
- Does any step require switching to a different device?
- Does any step require opening a development tool (IDE, terminal, Claude Code)?
- Does any step take longer than the user has?
- Does any step require knowledge the user won't have in context?

**If any step breaks, report the specific failure and what the architecture should do instead.**

## Step 3: Infrastructure Inventory

For every capability the plan proposes to **build**, check if it already exists.

**Files to check (read these before filling the table):**
- Project-specific CLAUDE.md files
- `terminal-use/OPERATIONS.md` (if the plan involves TU agents — env vars, credentials, tools available)
- `memory-topics/credentials-registry.md` (if the plan involves any API keys or tokens)
- `memory-topics/installed-tools.md` (if the plan involves MCPs, skills, or plugins)
- `memory-topics/dependency-map.md` (if the plan involves cross-system dependencies)
- The existing codebase for the target platform (e.g., events.js, agent.py, existing workflows)

**Format:**

| Capability Needed | Already Exists? | Where? | Gap to Fill |
|---|---|---|---|
| [What the plan says to build] | Yes / No / Partial | [File path or service] | [Only the delta needed] |

**Rules:**
- If "Already Exists?" is Yes for most rows, the design is an **extension**, not a new build. The plan should describe extension points, not new components.
- If the plan proposes a new credential, check the credentials registry first. Existing credentials with sufficient scopes don't need duplicates.
- If the plan proposes a new listener/webhook/handler, check if the existing platform already receives those events.

**If any row shows unnecessary new infrastructure, report it with the existing alternative.**

## Step 4: Assumption Audit

List every assumption the plan makes — about the user, the platform, the environment, the data, the timing. Assumptions are often unstated. Look for:

- **Interface assumptions:** "The user will invoke X from Y" — is Y actually where the user is?
- **Capability assumptions:** "The agent will do X" — can the agent actually do X with its current tools/permissions?
- **State assumptions:** "The system will remember X" — where is X stored? Is there session persistence?
- **Timing assumptions:** "This runs at X time" — is that when the user is available to respond?
- **Permission assumptions:** "The agent writes to X" — does it have write access?
- **Platform assumptions:** "The platform supports X" — has this been verified in docs?

**Format:**

| Assumption | Stated in Plan? | Verified? | How Verified |
|---|---|---|---|
| [Assumption text] | Yes / No | Yes / No | [Doc, code, or test that confirms it] |

**Rules:**
- Unstated assumptions are the most dangerous — surface them explicitly
- "Verified" means checked against documentation, code, or a real test. "I think so" is not verified.
- For unverified assumptions, either verify them now (read the doc, check the code) or flag them as risks

## Step 5: Validation Report

Present the consolidated findings:

### Summary
- **Requirements-Architecture:** X traced, Y missing, Z contradicted
- **User Context:** [One sentence — does the plan match the user's reality?]
- **Infrastructure:** X capabilities already exist, Y are genuinely new
- **Assumptions:** X verified, Y unverified, Z wrong

### Critical Issues (must fix before building)
[Numbered list of issues that would cause the plan to fail. Each includes: what's wrong, why it matters, and what the fix is.]

### Warnings (should fix, may not block)
[Numbered list of issues that could cause problems but won't prevent the system from working.]

### Validated (confirmed correct)
[Brief list of things the plan got right — builds confidence that the validation was thorough.]

**After presenting the report, ask:**
"Would you like me to update the plan to address the critical issues?"

## When to Run This

- **After `/workflows:plan`** — validate before building
- **After plan revisions** — re-validate to catch new errors introduced by fixes
- **Before any Phase N build** — validate the specific phase about to be built
- **When switching from planning to building** — final check before code is written
