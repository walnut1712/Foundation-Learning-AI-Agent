# Agent Context Analysis: Pre-Built Agents Context Loading

## Overview

This document analyzes how pre-built agents in the Specify framework load context. It covers 14 specialized agents across three categories: specification, planning/analysis, and git operations. Context loading follows a structured priority order based on agent lifecycle phases.

---

## Context Loading Phases (Universal Pattern)

All agents follow this loading sequence:

### Phase 0: User Input
1. Parse user arguments (`$ARGUMENTS`)
2. Check if empty; some agents error if no input provided

### Phase 1: Pre-Execution Checks (Hooks)
1. Check for `.specify/extensions.yml` existence
2. Load hook configuration for current agent
3. Execute enabled hooks (before_AGENT_NAME hooks)
4. Parse hook output (e.g., branch name, feature directory)

### Phase 2: Core Context Loading
1. Run setup scripts (e.g., `setup-specify.ps1`, `setup-plan.ps1`)
2. Load paths and configuration from script JSON output
3. Load required artifacts (spec.md, plan.md, etc.)
4. Load optional artifacts based on availability
5. Load system configuration (constitution.md, templates, etc.)

### Phase 3: Artifact Context Building
1. Parse loaded artifacts into semantic models
2. Cross-reference relationships
3. Extract decision context from prior phases

---

## Agent Context Loading Details

### Category 1: Specification & Design Workflow

#### 1. `speckit.specify` — Create/Update Feature Specification

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input (feature description) | `$ARGUMENTS` | Yes | User-provided before execution |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Spec directory configuration | `.specify/init-options.json` | Yes | Before directory creation |
| 4 | Specification template | `.specify/templates/spec-template.md` | Yes | Before writing spec |
| 5 | Feature directory config | `.specify/feature.json` | No | After directory created |

**What Loads When:**

```
User Input
    ↓
[Pre-Execution Hooks Check]
    ↓
Load init-options.json → Determine numbering (sequential/timestamp)
    ↓
Generate short name from feature description
    ↓
Create feature directory (specs/NNN-short-name or specs/TIMESTAMP-short-name)
    ↓
Load spec template
    ↓
Generate specification (filling template sections)
    ↓
Validate against quality checklist criteria
    ↓
Save to feature.json for downstream agents
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **No external context required**: This agent starts from scratch with just user input
- **Configuration-driven**: Directory naming depends on initialization options
- **Template-based generation**: Uses template structure for consistency
- **Quality gates**: Validates completeness before proceeding

---

#### 2. `speckit.plan` — Execute Implementation Planning Workflow

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input (planning notes) | `$ARGUMENTS` | No | Optional |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Feature specification | Setup script output → feature.json | Yes | Before loading |
| 4 | Specification content | `FEATURE_DIR/spec.md` | Yes | Phase 0 context |
| 5 | Constitution principles | `.specify/memory/constitution.md` | Yes | Phase 0 validation |
| 6 | Plan template | `.specify/templates/plan-template.md` | Yes | Before generation |
| 7 | Research findings | `FEATURE_DIR/research.md` (if exists) | No | Phase 1 input |
| 8 | Data model template | `.specify/templates/data-model-template.md` | No | Phase 1 generation |
| 9 | Agent context file | `.github/copilot-instructions.md` | Yes | Phase 1 update |

**What Loads When:**

```
User Input (optional planning notes)
    ↓
[Pre-Execution Hooks Check]
    ↓
Run setup-plan.ps1 → Parse FEATURE_DIR, SPECS_DIR, BRANCH
    ↓
Load Feature Specification (spec.md) → Extract requirements & success criteria
    ↓
Load Constitution (constitution.md) → Extract principles for validation
    ↓
Load Plan Template
    ↓
Phase 0: Research & Clarification
    ├─ Extract NEEDS CLARIFICATION from spec
    ├─ Load/reference research.md (if exists)
    └─ Generate research tasks for unknowns
    ↓
Phase 1: Design & Contracts
    ├─ Extract entities from spec → Generate data-model.md
    ├─ Validate against constitution principles
    ├─ Generate contracts/ if applicable
    ├─ Generate quickstart.md
    └─ Update agent context in copilot-instructions.md
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Prerequisite**: Must run after `speckit.specify` completes
- **Constitution validation**: Constitution is loaded early to guide all design decisions
- **Progressive disclosure**: Loads only necessary sections from spec initially
- **Agent context update**: Updates `.github/copilot-instructions.md` to point to new plan

---

#### 3. `speckit.clarify` — Identify Underspecified Areas

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input | `$ARGUMENTS` | No | Optional |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Current spec | `FEATURE_DIR/spec.md` | Yes | Before analysis |
| 4 | Plan (if exists) | `FEATURE_DIR/plan.md` | No | Additional context |
| 5 | Constitution | `.specify/memory/constitution.md` | Yes | Validation criteria |
| 6 | Clarification template | `.specify/templates/clarification-template.md` | No | For output format |

**What Loads When:**

```
User Input (optional context)
    ↓
[Pre-Execution Hooks Check]
    ↓
Load Specification (spec.md)
    ↓
Identify underspecified areas:
    ├─ Extract [NEEDS CLARIFICATION] markers (max 5 questions)
    ├─ Load plan.md for additional context (if exists)
    ├─ Prioritize by impact: scope > security/privacy > UX > technical
    └─ Generate clarification options (multiple choice where possible)
    ↓
Ask user up to 5 clarification questions
    ↓
Update spec.md with clarifications
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Question generation**: Identifies and prioritizes clarification needs
- **Limited questions**: Max 5 questions; further details use informed guesses
- **Contextual**: Uses plan context if available to provide better clarifications
- **In-place updates**: Directly updates spec.md after user provides answers

---

#### 4. `speckit.constitution` — Create/Update Project Constitution

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input (principles) | `$ARGUMENTS` | Yes or interactive | User-provided |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Existing constitution | `.specify/memory/constitution.md` | No | If updating |
| 4 | Constitution template | `.specify/templates/constitution-template.md` | Yes | For new constitution |

**What Loads When:**

```
User Input (principles as text or interactive session)
    ↓
[Pre-Execution Hooks Check]
    ↓
Check if constitution.md exists
    ├─ If YES: Load existing constitution (for updates)
    └─ If NO: Load constitution template
    ↓
Parse user-provided principles OR conduct interactive session
    ↓
Generate/Update constitution.md
    ↓
Update dependent templates to align with constitution
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Interactive or declarative**: Supports both user input and interactive Q&A
- **Template-synchronized**: Updates all downstream templates when principles change
- **Constitutional authority**: Non-negotiable principles that influence spec/plan validation

---

#### 5. `speckit.checklist` — Generate Custom Checklist

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input (domain context) | `$ARGUMENTS` | No | Optional domain-specific context |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Feature specification | `FEATURE_DIR/spec.md` | Yes | Extract feature details |
| 4 | Feature plan | `FEATURE_DIR/plan.md` | No | Additional context |
| 5 | Checklist template | `.specify/templates/checklist-template.md` | Yes | For structure |
| 6 | Existing checklists | `FEATURE_DIR/checklists/` | No | Reference for patterns |

**What Loads When:**

```
User Input (domain or feature context)
    ↓
[Pre-Execution Hooks Check]
    ↓
Load Feature Specification (spec.md)
    ↓
Load Feature Plan (plan.md) if exists
    ↓
Load Existing Checklists (for pattern reference)
    ↓
Load Checklist Template
    ↓
Generate domain-specific checklist:
    ├─ Extract key concepts from spec/plan
    ├─ Determine domain (e.g., security, performance, testing)
    └─ Customize template for specific domain
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Domain-driven**: Generates checklists tailored to feature domain
- **Flexible input**: Works with or without explicit user domain specification
- **Pattern reference**: Uses existing checklists to maintain consistency

---

### Category 2: Task & Analysis Workflow

#### 6. `speckit.tasks` — Generate Actionable Task List

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input | `$ARGUMENTS` | No | Optional context |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Setup context | Setup script output (setup-tasks.ps1) | Yes | Get FEATURE_DIR, paths |
| 4 | Plan template | Script output → `TASKS_TEMPLATE` | Yes | Task structure |
| 5 | Feature specification | `FEATURE_DIR/spec.md` | Yes | Extract user stories & priorities |
| 6 | Implementation plan | `FEATURE_DIR/plan.md` | Yes | Tech stack, architecture, structure |
| 7 | Data model | `FEATURE_DIR/data-model.md` | No | Entity mapping to stories |
| 8 | Interface contracts | `FEATURE_DIR/contracts/` | No | Contract-to-story mapping |
| 9 | Research findings | `FEATURE_DIR/research.md` | No | Technical decisions for setup |
| 10 | Quickstart guide | `FEATURE_DIR/quickstart.md` | No | Test scenarios reference |
| 11 | Available documents | Setup script output → `AVAILABLE_DOCS` | Yes | Know what exists to load |

**What Loads When:**

```
User Input (optional context)
    ↓
[Pre-Execution Hooks Check]
    ↓
Run setup-tasks.ps1 → Parse FEATURE_DIR, AVAILABLE_DOCS, TASKS_TEMPLATE
    ↓
Load Required Artifacts:
    ├─ Feature Specification (spec.md)
    │   ├─ Extract user stories with priorities (P1, P2, P3)
    │   └─ Extract acceptance criteria
    └─ Implementation Plan (plan.md)
        ├─ Extract tech stack
        ├─ Extract libraries/dependencies
        └─ Extract project structure
    ↓
Load Optional Artifacts (based on AVAILABLE_DOCS):
    ├─ Data Model (data-model.md) → Map entities to stories
    ├─ Contracts (contracts/) → Map interfaces to stories
    ├─ Research (research.md) → Extract technical decisions
    └─ Quickstart (quickstart.md) → Reference test scenarios
    ↓
Generate Tasks Organization:
    ├─ Phase 1: Setup (project initialization)
    ├─ Phase 2: Foundational (blocking prerequisites for all stories)
    └─ Phase 3+: One phase per user story (in priority order)
    ↓
Generate tasks.md with:
    ├─ Correct feature name
    ├─ All phase tasks in checklist format
    ├─ Dependencies & parallel execution examples
    ├─ Independent test criteria per story
    └─ Suggested MVP scope
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Prerequisite chain**: Requires both spec.md and plan.md to exist
- **Progressive loading**: Loads only available documents; gracefully handles missing optional artifacts
- **Story-organized**: Tasks grouped by user story for independent implementation
- **Dependency mapping**: Identifies parallel opportunities and blocking relationships
- **MVP-focused**: Suggests MVP scope (typically User Story 1)

---

#### 7. `speckit.analyze` — Cross-Artifact Consistency Analysis

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input | `$ARGUMENTS` | No | Optional |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Setup context | setup-prerequisites script output | Yes | Get FEATURE_DIR, paths |
| 4 | Feature specification | `FEATURE_DIR/spec.md` | Yes | Requirements, scenarios, edge cases |
| 5 | Implementation plan | `FEATURE_DIR/plan.md` | Yes | Architecture, stack, constraints |
| 6 | Task list | `FEATURE_DIR/tasks.md` | Yes | Required (runs only after tasks generated) |
| 7 | Constitution | `.specify/memory/constitution.md` | Yes | Principle validation |
| 8 | Data model (optional context) | `FEATURE_DIR/data-model.md` | No | Entity reference for consistency |
| 9 | Contracts (optional context) | `FEATURE_DIR/contracts/` | No | Interface consistency check |

**What Loads When:**

```
User Input (optional)
    ↓
[Pre-Execution Hooks Check]
    ↓
Run check-prerequisites script → Get FEATURE_DIR, AVAILABLE_DOCS
    ↓
Verify All Required Files Exist (error if missing):
    ├─ spec.md
    ├─ plan.md
    └─ tasks.md
    ↓
Load & Parse Specification (spec.md):
    ├─ Overview & Context
    ├─ Functional Requirements (FR-### IDs)
    ├─ Success Criteria (SC-### IDs & measurable outcomes)
    ├─ User Stories with acceptance criteria
    └─ Edge Cases
    ↓
Load & Parse Implementation Plan (plan.md):
    ├─ Architecture/stack choices
    ├─ Data Model references
    ├─ Phases
    └─ Technical constraints
    ↓
Load & Parse Task List (tasks.md):
    ├─ Task IDs (T-###)
    ├─ Descriptions & file paths
    ├─ Phase grouping
    └─ Parallel markers [P]
    ↓
Load Constitution (constitution.md):
    ├─ Principle names
    └─ MUST/SHOULD normative statements
    ↓
Build Semantic Models (internal analysis):
    ├─ Requirements inventory (FR-###, SC-### → requirements map)
    ├─ User story/action inventory
    ├─ Task coverage mapping (task → requirement/story)
    └─ Constitution rule set (principles)
    ↓
Execute Detection Passes (Analysis):
    ├─ Duplication Detection
    ├─ Ambiguity Detection (vague adjectives, unresolved placeholders)
    ├─ Underspecification Detection (missing objects, outcomes)
    ├─ Constitution Alignment Checks
    ├─ Coverage Gaps (requirements with 0 tasks)
    └─ Inconsistency Detection (terminology drift, conflicts)
    ↓
Severity Assignment & Report Generation (READ-ONLY output)
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Prerequisite requirement**: Must run ONLY after `speckit.tasks` generates tasks.md
- **Read-only**: Produces analysis report; does NOT modify files
- **Constitution authority**: Constitution violations are automatic CRITICAL findings
- **Token-efficient**: Limits findings to 50 total; aggregates overflow
- **Multi-artifact validation**: Checks consistency across spec, plan, and tasks
- **NO remediation**: Offers suggestions but requires explicit user approval before any edits

---

### Category 3: Git Workflow Agents

#### 8. `speckit.git.initialize` — Initialize Git Repository

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input | `$ARGUMENTS` | No | Optional |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Git script | `.specify/extensions/git/scripts/powershell/initialize-repo.ps1` | Yes (or fallback) | For initialization |
| 4 | Git availability check | System PATH | Yes | Prerequisite check |

**What Loads When:**

```
User Input (optional)
    ↓
[Pre-Execution Hooks Check]
    ↓
Check Git Installation (system check)
    ↓
Check if already in Git repo (git status)
    ↓
Load Git Initialization Script:
    ├─ Try: .specify/extensions/git/scripts/powershell/initialize-repo.ps1
    └─ Fallback: Built-in git init/add/commit commands
    ↓
Execute:
    ├─ git init
    ├─ git add .
    └─ git commit -m "Initial commit from Specify template"
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Graceful degradation**: Skips if Git not installed; works without it
- **Idempotent**: Skips if already a Git repo
- **Minimal context**: No artifact dependencies
- **Script-extensible**: Custom .gitignore, branches, hooks via script customization

---

#### 9. `speckit.git.feature` — Create Feature Branch

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input (branch name/number) | `$ARGUMENTS` | No or via config | Optional override |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Init options | `.specify/init-options.json` | Yes | Numbering scheme (sequential/timestamp) |
| 4 | Git script | `.specify/extensions/git/scripts/powershell/feature-branch.ps1` | Yes (or fallback) | For branch creation |
| 5 | Git repository state | Current branch, existing branches | Yes | Context check |

**What Loads When:**

```
User Input (optional branch name/number)
    ↓
[Pre-Execution Hooks Check]
    ↓
Load init-options.json → Determine numbering scheme
    ↓
Load Git Feature Branch Script:
    ├─ Try: .specify/extensions/git/scripts/powershell/feature-branch.ps1
    └─ Fallback: Built-in git branch commands
    ↓
Determine Branch Name:
    ├─ If user provided: Use explicit GIT_BRANCH_NAME
    ├─ If sequential numbering: Generate NNN-feature-name
    └─ If timestamp numbering: Generate YYYYMMDD-HHMMSS-feature-name
    ↓
Execute:
    ├─ git checkout -b <branch-name>
    └─ Optional: Link to feature directory via .specify/feature.json
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Configuration-driven**: Numbering follows project init-options.json
- **Optional enforcement**: Can accept explicit branch names
- **Lightweight**: Minimal context dependency; works independently

---

#### 10. `speckit.git.validate` — Validate Branch Naming

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input | `$ARGUMENTS` | No | Optional |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Git branch name | Current git HEAD | Yes | What to validate |
| 4 | Init options | `.specify/init-options.json` | Yes | Expected naming convention |
| 5 | Validation script | `.specify/extensions/git/scripts/powershell/validate-branch.ps1` | Yes (or fallback) | Validation logic |

**What Loads When:**

```
User Input (optional)
    ↓
[Pre-Execution Hooks Check]
    ↓
Get current Git branch name (git rev-parse --abbrev-ref HEAD)
    ↓
Load init-options.json → Expected naming convention
    ↓
Load Validation Script:
    ├─ Try: .specify/extensions/git/scripts/powershell/validate-branch.ps1
    └─ Fallback: Built-in pattern matching
    ↓
Validate:
    ├─ Current branch matches expected pattern
    ├─ If sequential: Matches NNN-feature-name format
    └─ If timestamp: Matches YYYYMMDD-HHMMSS-feature-name format
    ↓
Report: PASS or FAIL with remediation suggestions
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Validation-only**: Read-only; does not modify branch
- **Configuration-aware**: Validates against project's chosen convention
- **Feedback-focused**: Provides clear remediation guidance on failure

---

#### 11. `speckit.git.remote` — Detect Git Remote URL

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input (remote name) | `$ARGUMENTS` | No | Optional override |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Git remote config | `git remote -v` | Yes | Current remotes |
| 4 | Git script | `.specify/extensions/git/scripts/powershell/detect-remote.ps1` | Yes (or fallback) | Detection logic |

**What Loads When:**

```
User Input (optional remote name, default "origin")
    ↓
[Pre-Execution Hooks Check]
    ↓
Get Git Remotes (git remote -v)
    ↓
Load Remote Detection Script:
    ├─ Try: .specify/extensions/git/scripts/powershell/detect-remote.ps1
    └─ Fallback: Parse git remote -v output
    ↓
Detect Remote URL:
    ├─ If explicit remote provided: Query that remote
    └─ Else: Use "origin" by default
    ↓
Parse URL for GitHub Integration:
    ├─ Extract repository owner/name
    ├─ Validate GitHub URL pattern
    └─ Format for GitHub API usage
    ↓
Store Detected Remote (for downstream GitHub integration)
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **GitHub-aware**: Specifically detects and parses GitHub URLs
- **Minimal context**: Just needs git remote info
- **Foundational**: Prerequisite for GitHub-integrated features

---

#### 12. `speckit.git.commit` — Auto-commit Changes

**Priority Order of Context Loading:**

| Priority | Component | Source | Mandatory | Timing |
|----------|-----------|--------|-----------|--------|
| 1 | User input (commit message) | `$ARGUMENTS` | Yes | Explicit message or generated |
| 2 | Hook configuration | `.specify/extensions.yml` | No | Before execution |
| 3 | Git staged changes | `git status --short` | Yes | What to commit |
| 4 | Git script | `.specify/extensions/git/scripts/powershell/commit.ps1` | Yes (or fallback) | Commit logic |
| 5 | Feature context | `.specify/feature.json` (optional) | No | Generate commit message |

**What Loads When:**

```
User Input (commit message or context)
    ↓
[Pre-Execution Hooks Check]
    ↓
Check Git Status (git status)
    ↓
Load Git Commit Script:
    ├─ Try: .specify/extensions/git/scripts/powershell/commit.ps1
    └─ Fallback: Built-in git commit command
    ↓
Determine Commit Message:
    ├─ If user provided explicit message: Use it
    ├─ If feature context available: Generate from feature.json
    └─ Else: Use default "Update specification artifacts"
    ↓
Execute:
    ├─ git add . (or select files)
    └─ git commit -m "<message>"
    ↓
Report: Commit hash, branch, files committed
    ↓
[Post-Execution Hooks Check]
```

**Key Observations:**
- **Context-aware**: Can generate meaningful messages from feature context
- **Flexible**: Accepts explicit or auto-generated commit messages
- **End-of-workflow**: Typically run after specify/plan/tasks agents complete
- **Minimal pre-requisites**: Works independently of artifact state

---

### Summary: Git Agents Context Dependencies

| Agent | Prerequisite | Context Type | Output Context |
|-------|-------------|-------------|-----------------|
| git.initialize | None | System (Git install check) | Git repo initialized |
| git.feature | init-options.json | Config + Git state | Feature branch created |
| git.validate | init-options.json | Config + Git state | Validation report |
| git.remote | None | Git state | Detected remote URL |
| git.commit | feature.json (optional) | Feature context + Git state | Commit hash |

---

## Cross-Agent Context Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    SPECIFICATION PHASE                       │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  speckit.specify ──→ spec.md                                 │
│       ↓              ├─ Creates: feature.json                │
│       ↓              └─ Stores: Feature directory path       │
│       ↓                                                       │
│  [OPTIONAL] speckit.clarify (refine spec.md)                 │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                    PLANNING PHASE                            │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  speckit.plan  (reads: spec.md, constitution.md)             │
│       ↓         (creates: plan.md, data-model.md,            │
│       ↓                   contracts/, research.md)           │
│       ↓         (updates: .github/copilot-instructions.md)   │
│       ↓                                                       │
│  [OPTIONAL] speckit.checklist (refs: plan.md, spec.md)       │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                     TASKS PHASE                              │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  speckit.tasks (reads: spec.md, plan.md,                     │
│       ↓                  data-model.md, contracts/,          │
│       ↓                  research.md, quickstart.md)         │
│       ↓         (creates: tasks.md)                          │
│       ↓                                                       │
│  [OPTIONAL] speckit.analyze (reads: spec.md, plan.md,        │
│                                     tasks.md,               │
│                                     constitution.md)        │
│                              (outputs: Analysis report ONLY) │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                 IMPLEMENTATION PHASE                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  speckit.implement (reads: tasks.md, plan.md)                │
│       ↓             (executes tasks in phases)               │
│       ↓                                                       │
│  [ASYNC] speckit.git.commit (optional commits)               │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Context Loading Priority Rules

### Rule 1: Early Hook Execution
- **When**: Immediately before agent core logic
- **Why**: Allows pre-flight setup (e.g., git branch creation, custom configuration)
- **Agents**: All agents check hooks first

### Rule 2: Setup Script Execution (Plan/Tasks/Analyze)
- **When**: Before loading any artifacts
- **Why**: Scripts resolve dynamic paths and check prerequisites
- **Agents**: `speckit.plan`, `speckit.tasks`, `speckit.analyze`, `speckit.implement`

### Rule 3: Mandatory Before Optional
- **When**: Within artifact loading phase
- **Why**: Ensures core context is always available
- **Example**: `plan.md` must load before optional `data-model.md`

### Rule 4: Progressive Disclosure
- **When**: Only load sections needed for current phase
- **Why**: Minimizes context bloat; improves token efficiency
- **Example**: `speckit.plan` loads only "Functional Requirements" and "Success Criteria" from spec, not full narrative

### Rule 5: Constitution Validation Gateway
- **When**: After all feature-specific context loads
- **Why**: Constitution is project-level policy; must validate ALL decisions against it
- **Agents**: `speckit.plan` (Phase 0), `speckit.analyze` (all detection passes)

### Rule 6: Artifact Interdependencies
- **When**: Design-phase agents establish strict prerequisite chains
- **Priority Order**:
  1. `speckit.specify` must run first (creates spec.md)
  2. `speckit.plan` requires spec.md (creates plan.md, data-model.md, etc.)
  3. `speckit.tasks` requires both spec.md + plan.md (creates tasks.md)
  4. `speckit.analyze` requires all three + constitution (read-only analysis)
  5. `speckit.implement` requires tasks.md (executes tasks)

### Rule 7: Read-Only Analysis Phase
- **When**: `speckit.analyze` runs
- **Why**: Quality gate before implementation; no destructive changes allowed
- **Consequence**: User must explicitly approve any remediation edits

---

## Context Persistence & Reuse

### State Persistence Mechanisms

| Storage | Agent(s) | Content | Lifecycle |
|---------|----------|---------|-----------|
| `.specify/feature.json` | All design agents | Feature directory path, resolved from feature arguments | Created by `speckit.specify`, read by downstream agents |
| `.specify/extensions.yml` | All agents (hooks) | Hook configurations, enabled/disabled flags | Created by user; persists across agent invocations |
| `.specify/init-options.json` | Git agents, `speckit.specify` | Numbering scheme (sequential/timestamp), templates | Created at project init; rarely changed |
| `.specify/memory/constitution.md` | `speckit.plan`, `speckit.analyze` | Project principles (MUST/SHOULD rules) | Created by `speckit.constitution`; referenced by all validation |
| `FEATURE_DIR/spec.md` | All downstream agents | Feature specification; requirements, stories, edge cases | Created by `speckit.specify`; read by all downstream agents |
| `FEATURE_DIR/plan.md` | Task & implementation agents | Architecture, data model references, phases | Created by `speckit.plan`; read by `speckit.tasks` and `speckit.implement` |
| `FEATURE_DIR/tasks.md` | Implementation & analysis agents | Actionable tasks, phases, dependencies | Created by `speckit.tasks`; read by `speckit.analyze` and `speckit.implement` |
| `.github/copilot-instructions.md` | Agent context | Plan file reference (updated by `speckit.plan`) | Auto-updated by design agents; manually configured initially |

### Context Reuse Pattern

```
Session Execution Path:
    /speckit.specify
        └─ Writes: spec.md, feature.json
    
    ↓ (downstream agent reads)
    
    /speckit.plan
        ├─ Reads: spec.md, feature.json, constitution.md
        └─ Writes: plan.md, data-model.md, contracts/, research.md
    
    ↓ (downstream agent reads)
    
    /speckit.tasks
        ├─ Reads: spec.md, plan.md, data-model.md, contracts/, research.md
        └─ Writes: tasks.md
    
    ↓ (optional validation)
    
    /speckit.analyze
        ├─ Reads: spec.md, plan.md, tasks.md, constitution.md (READ-ONLY)
        └─ Outputs: Analysis report (no file writes)
    
    ↓ (implementation)
    
    /speckit.implement
        ├─ Reads: tasks.md, plan.md
        └─ Executes: Tasks in phases (writes user code artifacts)
```

---

## Key Insights & Patterns

### 1. **Multi-Phase Context Loading**
All agents follow four phases: user input → hooks → setup → core loading. This ensures consistent initialization and extensibility.

### 2. **Hook-Based Extensibility**
Every agent allows pre/post-execution hooks via `.specify/extensions.yml`. Hooks run before core logic, enabling custom initialization without modifying agent code.

### 3. **Constitution as Guardrail**
Constitution principles are loaded early and guide all downstream decisions. Violations are treated as CRITICAL findings, not suggestions.

### 4. **Read-Only Analysis**
`speckit.analyze` is intentionally read-only. It identifies issues but requires explicit user approval before any remediation edits. This prevents accidental data loss.

### 5. **Progressive Artifact Chain**
- Specification → Blueprint (what user wants)
- Plan → Architecture (how to build it)
- Tasks → Implementation (do it step-by-step)
- Analysis → Quality gate (verify it all fits)
- Implementation → Execution (build it)

Each artifact depends on prior stages; breaking the chain requires re-running from the break point.

### 6. **Graceful Degradation**
Optional artifacts (data-model.md, contracts/, research.md) are loaded if they exist; agents continue if they don't. This enables incremental specification without forcing all artifacts upfront.

### 7. **Git Integration Independence**
Git agents operate independently of feature context. They can initialize repos, create branches, and commit changes without reading spec/plan/tasks. This allows parallel git workflow setup.

### 8. **Configuration-Driven Behavior**
Numbering schemes, templates, and hooks are all configurable via `.specify/init-options.json` and `.specify/extensions.yml`. Agents adapt behavior based on project configuration rather than hardcoding conventions.

---

## Agent Context Loading Matrix

| Agent | Required Context | Optional Context | Produces | Read-Only? |
|-------|------------------|------------------|----------|-----------|
| specify | User input + init-options.json + spec-template | extensions.yml | spec.md, feature.json | No |
| plan | spec.md + plan-template + constitution | extensions.yml, research.md | plan.md, data-model.md, contracts/, research.md | No |
| clarify | spec.md + constitution | extensions.yml, plan.md | Updated spec.md | No |
| constitution | constitution-template OR existing constitution.md | extensions.yml | constitution.md + updated templates | No |
| checklist | spec.md + checklist-template | extensions.yml, plan.md, existing checklists | Checklist file | No |
| tasks | spec.md + plan.md + tasks-template | extensions.yml, data-model.md, contracts/, research.md, quickstart.md | tasks.md | No |
| analyze | spec.md + plan.md + tasks.md + constitution | extensions.yml, data-model.md, contracts/ | Analysis report (stdout) | **YES** |
| git.initialize | Git install check | extensions.yml | Git repo | No |
| git.feature | init-options.json | extensions.yml, feature.json | Feature branch | No |
| git.validate | init-options.json | extensions.yml | Validation report (stdout) | **YES** |
| git.remote | Git remotes config | extensions.yml | Remote URL (stdout) | **YES** |
| git.commit | Git status | extensions.yml, feature.json | Git commit | No |

---

## Summary: Context Loading Priority Order (Universal)

1. **User Input** — Explicit arguments provided by user
2. **Pre-Execution Hooks** — Custom initialization from extensions
3. **Configuration Files** — init-options.json, extensions.yml
4. **Setup Scripts** — Dynamic path resolution & prerequisite checks
5. **Required Artifacts** — Core context (spec.md, plan.md, tasks.md)
6. **System Files** — Templates, constitution, guidelines
7. **Optional Artifacts** — Conditional context (data-model, contracts, research)
8. **Semantic Models** — Internal representations for analysis
9. **Validation & Output** — Perform work; generate results

**Key Principle**: Load minimum necessary context to proceed safely, validate early and often, extend via hooks, and persist state for downstream reuse.

