# VS Code Copilot Customization Layers

A comprehensive guide to understanding the five customization layers available in VS Code Copilot, how they work, and how they interact with each other.

---

## Overview

VS Code Copilot provides five distinct customization mechanisms that work together to extend and configure agent behavior:

1. **Instructions** - Always-on guidelines
2. **Agents** - Custom personas with specific capabilities
3. **Skills** - On-demand workflows with bundled resources
4. **Prompts** - Single focused tasks
5. **MCP (Model Context Protocol)** - External system integration

Each layer serves a different purpose and operates at a different scope. Together, they enable comprehensive customization of Copilot's behavior for your specific project or workflow.

---

## 1. Instructions (.instructions.md files)

### Purpose

Guidelines that apply to specific file types or tasks, loaded on-demand when relevant or automatically when files match patterns.

### When to Use

- **File-specific patterns**: Language or framework standards (e.g., Python conventions)
- **Task-based guidance**: Triggered when agent detects task relevance (e.g., database migrations, API design)
- **Explicit attachment**: When working with files matching specific patterns

### File Structure & Format

**Locations:**

```
Workspace scope:  .github/instructions/*.instructions.md
User scope:       <profile>/instructions/*.instructions.md
```

**Frontmatter (YAML):**

```yaml
---
description: "<required>" # Keyword-rich, for on-demand discovery
name: "Instruction Name" # Optional, defaults to filename
applyTo: "**/*.py" # Optional, auto-attach for matching files
---
```

**Body Template:**

````markdown
# Instruction Title

## When Applied

- Context where this instruction is used

## Key Guidelines

- Rule 1
- Rule 2

## Examples

```codeblock
example code
```
````

````

### Discovery Modes

| Mode | Trigger | Use Case |
|------|---------|----------|
| **On-demand** | Agent detects task relevance via keywords in `description` | Task-based: migrations, refactoring |
| **Explicit** (`applyTo`) | Files matching glob pattern in context | File-based: language standards, framework rules |
| **Manual** | User via `Add Context` → `Instructions` | Ad-hoc attachment for specific work |

### Scope
- **Workspace-level**: Shared across team via version control (`.github/instructions/`)
- **User-level**: Personal, synced across devices via VS Code settings sync (`~/.copilot/instructions/`)

### Key Characteristics
- **Light-weight**: Loaded only when relevant
- **Keyword-rich descriptions**: Enable agent discovery via trigger phrases like "Use when..."
- **File matching**: Can auto-attach to specific file types via `applyTo` glob patterns
- **Example in workspace**: [coding-conventions.instructions.md](.github/instructions/coding-conventions.instructions.md) - Applies to all `.py` files

### Example from Workspace

```yaml
---
description: "Use when writing Python code. Enforces naming conventions,
  error-handling patterns, logging standards, async/await best practices,
  and testing conventions for consistency and maintainability."
name: "Coding Conventions"
applyTo: "**/*.py"
---

# Coding Conventions & Standards

## Naming Conventions

### Variables & Functions
- **snake_case** for variables, functions, and method names
  - ✓ `process_data()`, `user_profile`, `max_retries`
  - ✗ `ProcessData()`, `userProfile`, `maxRetries`

## Error Handling

### Exception Strategy
- **Catch specific exceptions**, not bare `except:`
  - ✓ `except ValueError:`, `except (KeyError, TypeError):`
  - ✗ `except:`, `except Exception:`

## Testing Conventions

### Test Structure
- **Test file naming**: `test_<module>.py` (e.g., `test_api_client.py`)
- **Test function naming**: `test_<function>_<scenario>`
````

### Common Pitfalls

- ❌ Vague descriptions like "Helpful coding tips" don't enable discovery
- ❌ Overly broad `applyTo: "**"` burns context on every interaction
- ❌ Duplicating documentation instead of linking to existing docs
- ❌ Mixing unrelated concerns (testing + styling + API design in one file)

---

## 2. Agents (.agent.md files)

### Purpose

Custom personas with specific tools, instructions, and behaviors. Agents are specialized AI personas designed for orchestrated workflows with role-based tool restrictions and different capabilities per agent.

### When to Use

- **Specialized workflows**: Different stages need different capabilities
- **Tool isolation**: Restrict tools based on role (e.g., security reviewer can't edit code)
- **Context separation**: Tasks that need isolated context to avoid interference
- **Role-based behavior**: Auditor vs. Developer vs. Documentation Writer
- **Multi-stage processes**: Decompose complex tasks into agent handoffs

### File Structure & Format

**Locations:**

```
Workspace scope:  .github/agents/*.agent.md
User scope:       <profile>/agents/*.agent.md
```

**Frontmatter:**

```yaml
---
description: "<required>" # Use when... trigger phrases for subagent discovery
name: "Agent Name" # Optional, defaults to filename
tools: [read, search, execute] # Optional: built-in aliases, MCP servers, extension tools
model: "Claude Sonnet 4.5 (copilot)" # Optional, uses picker default; supports array for fallback
argument-hint: "Task..." # Optional, input guidance for users
agents: [agent1, agent2] # Optional, restrict allowed subagents by name
user-invocable: true # Optional, show in agent picker (default: true)
disable-model-invocation: false # Optional, prevent as subagent (default: false)
handoffs: [...] # Optional, transitions to other agents
hooks: # Optional, inline hooks for lifecycle events
  PreToolUse:
    - type: command
      command: "./scripts/validate.sh"
  PostToolUse:
    - type: command
      command: "./scripts/format.sh"
---
```

### Body Template

```markdown
---
description: "Use when {specific task description}. Specialized in {what this agent does}."
tools: [{ minimal set needed for role }]
user-invocable: false
---

You are a specialist at {specific task}. Your job is to {clear purpose}.

## Constraints

- DO NOT {thing this agent should never do}
- DO NOT {another restriction}
- ONLY {the one thing this agent does}

## Approach

1. {Step one of how this agent works}
2. {Step two}
3. {Step three}

## Output Format

{Exactly what this agent should return}
```

### Tool Aliases

| Alias     | Purpose                           |
| --------- | --------------------------------- |
| `execute` | Run shell commands                |
| `read`    | Read file contents                |
| `edit`    | Edit files                        |
| `search`  | Search files or text              |
| `agent`   | Invoke custom agents as subagents |
| `web`     | Fetch URLs and web search         |
| `todo`    | Manage task lists                 |

### Scope

- **Workspace-level**: Team-shared custom agents (`.github/agents/`)
- **User-level**: Personal agents synced via settings (`~/.agents/`)
- **Invocation**: Manual selection in agent picker or automatic delegation

### Invocation Control

| Setting                          | Effect                                         |
| -------------------------------- | ---------------------------------------------- |
| `user-invocable: false`          | Hide from picker, only accessible as subagent  |
| `disable-model-invocation: true` | Prevent other agents from invoking as subagent |
| `agents: []`                     | No subagent invocation allowed                 |
| `agents: [agent1, agent2]`       | Restrict subagent calls to specific agents     |

### Key Differences from Instructions

- **Tools**: Agents have restricted tool sets; instructions don't control tools
- **Personas**: Agents have defined roles/personalities; instructions are guidelines
- **Complexity**: Agents orchestrate workflows; instructions guide single tasks
- **Scope**: Agents control what can be done; instructions guide how to do it
- **Invocation**: Agents can be manually selected or auto-delegated; instructions are context-dependent

### Common Patterns

```yaml
# Read-only researcher
tools: [read, search, web]

# Secure code reviewer
tools: [read, search]
hooks:
  PreToolUse:
    - type: command
      command: "./scripts/block-dangerous-cmds.sh"

# No terminal access
tools: [read, edit, search]

# Conversational only
tools: []

# MCP server only
tools: [myserver/*]
```

### Common Pitfalls

- ❌ Swiss-army agents with too many tools (dilutes focus)
- ❌ Vague descriptions that don't guide delegation
- ❌ Role confusion between description and actual persona
- ❌ Circular handoffs without progress criteria (A → B → A)

---

## 3. Skills (SKILL.md files)

### Purpose

On-demand workflows with bundled resources (scripts, templates, reference docs). Skills are discovered and invoked by users or agents when specialized multi-step procedures are needed.

### When to Use

- **Repeatable multi-step workflows**: Testing, deployment, analysis
- **Bundled resources needed**: Scripts, templates, configuration examples
- **Task-specific knowledge**: Procedures that are too complex for inline instructions
- **User-invokable tasks**: Appear as slash commands (`/skill-name`)
- **Automation workflows**: Automation, refactoring, migrations

### File Structure & Format

**Locations:**

```
Workspace scope:  .github/skills/<skill-name>/
Alternative:      .agents/skills/<skill-name>/
                  .claude/skills/<skill-name>/
User scope:       ~/.copilot/skills/<skill-name>/
                  ~/.agents/skills/<skill-name>/
                  ~/.claude/skills/<skill-name>/
```

**Folder Structure:**

```
.github/skills/test-webapp/
├── SKILL.md              # Required (name must match folder)
├── scripts/              # Executable code
│   ├── test.js          # Playwright test script
│   └── setup.sh
├── references/           # Detailed documentation
│   ├── patterns.md      # Advanced patterns
│   └── troubleshooting.md
└── assets/               # Templates, boilerplate
    ├── template.html
    └── config.json
```

**SKILL.md Frontmatter:**

```yaml
---
name: skill-name # Required: 1-64 chars, lowercase alphanumeric + hyphens
description: "What and when. Max 1024 chars."
argument-hint: "Optional hint shown for slash invocation"
user-invocable: true # Optional: show as slash command (default: true)
disable-model-invocation: false # Optional: disable automatic loading (default: false)
---
```

**Body Template:**

```markdown
---
name: webapp-testing
description: "Test web applications using Playwright. Use for verifying
  frontend, debugging UI, capturing screenshots."
---

# Web Application Testing

## When to Use

- Verify frontend functionality
- Debug UI behavior
- Generate visual regression tests

## Procedure

1. Start the development web server
2. Review [test configuration](./references/config.md)
3. Run [test script](./scripts/test.js)
4. Review screenshots in `./screenshots/`

## References

- [Advanced Patterns](./references/patterns.md)
- [Troubleshooting Guide](./references/troubleshooting.md)

## Scripts

- [Playwright Test](./scripts/test.js)
- [Setup Environment](./scripts/setup.sh)
```

### Progressive Loading Strategy

1. **Discovery** (~100 tokens): Agent reads `name` and `description`
2. **Instructions** (<5000 tokens): Loads SKILL.md body when relevant
3. **Resources**: Additional files load only when referenced

_Keep SKILL.md under 500 lines; use reference files for detailed content._

### Slash Command Behavior

Both skills and prompts appear as slash commands in chat.

| Configuration                    | Slash Command | Auto-loaded |
| -------------------------------- | ------------- | ----------- |
| Default (both omitted)           | Yes           | Yes         |
| `user-invocable: false`          | No            | Yes         |
| `disable-model-invocation: true` | Yes           | No          |
| Both set                         | No            | No          |

### Scope

- **Workspace-level**: Team-shared skills (`.github/skills/`)
- **User-level**: Personal skills available across workspaces (`~/.copilot/skills/`)
- **Auto-loading**: Agent can automatically invoke skills when relevant
- **Manual invocation**: Users can type `/skill-name` in chat

### Example from Workspace

```yaml
---
name: agent-customization
description: 'Create, update, review, fix, or debug VS Code agent customization
  files (.instructions.md, .prompt.md, .agent.md, SKILL.md, copilot-instructions.md,
  AGENTS.md). USE FOR: saving coding preferences; troubleshooting why
  instructions/skills/agents are ignored or not invoked; configuring applyTo patterns;
  defining tool restrictions; creating custom agent modes or specialized workflows;
  packaging domain knowledge; fixing YAML frontmatter syntax.'
---

# Agent Customization

## Decision Flow

| Primitive | When to Use |
|-----------|-------------|
| agent instructions | Always-on, applies everywhere in the project |
| File Instructions | Explicit via `applyTo` patterns, or on-demand via `description` |
| MCP | Integrates external systems, APIs, or data |
| Custom Agents | Subagents for context isolation, or multi-stage workflows |
| Prompts | Single focused task with parameterized inputs |
| Skills | On-demand workflow with bundled assets |

## Quick Reference

[Consult reference docs...]

## Creation Process

[Detailed step-by-step creation instructions...]
```

### Common Pitfalls

- ❌ Vague descriptions ("A helpful skill") don't enable discovery
- ❌ Monolithic SKILL.md with everything instead of using reference files
- ❌ Folder name doesn't match `name` field in frontmatter
- ❌ Missing step-by-step procedures (descriptions without guidance)
- ❌ Non-relative paths for resources (use `./scripts/` not `/full/path/`)

---

## 4. Prompt Files (.prompt.md)

### Purpose

Reusable task templates triggered on-demand in chat. Single focused task with parameterized inputs.

### When to Use

- **One-off generation tasks**: Generate test cases, create READMEs, summarize metrics
- **Parameterized workflows**: Task that needs user input (arguments)
- **Quick templates**: Invoking via slash command (`/prompt-name`)
- **Specific output format**: Task where output structure matters
- **Consistent generation**: Reusable task invoked multiple times with different inputs

### File Structure & Format

**Locations:**

```
Workspace scope:  .github/prompts/*.prompt.md
User scope:       <profile>/prompts/*.prompt.md
```

**Frontmatter:**

```yaml
---
description: "Optional, improves discoverability"
name: "Prompt Name" # Optional, defaults to filename
argument-hint: "Task input..." # Optional: hint in chat input
agent: "agent" # Optional: ask, agent, plan, or custom agent
model: "GPT-5 (copilot)" # Optional: selected model, or fallback array
tools: [search, web, execute] # Optional: built-in, tool sets, MCP (<server>/*), extension
---
```

**Model Fallback Array:**

```yaml
model: ["GPT-5 (copilot)", "Claude Sonnet 4.5 (copilot)"] # First available is used
```

**Body Template:**

```markdown
---
description: "Generate test cases for selected code"
agent: "agent"
tools: [search, web]
---

Generate comprehensive test cases for the provided code:

- Include edge cases and error scenarios
- Follow existing test patterns in the codebase
- Use descriptive test names

## Output Format

- Test class: `test_<feature>.py`
- Test method: `test_<scenario>`
- Each test should be independent and self-contained
```

### Context References

- **File links**: Use Markdown links for files: `[config](./config.json)`
- **Tool references**: Use `#tool:<name>` for specific tools

### Invocation

- **Chat**: Type `/` → select from prompts and skills
- **Command**: `Chat: Run Prompt...`
- **Editor**: Open prompt file → play button
- **UI Actions**: Configure via `chat.promptFilesRecommendations`

### Tool Priority

When both prompt and custom agent define tools:

1. Tools from prompt file
2. Tools from referenced custom agent
3. Default tools for selected agent

### Scope

- **Workspace-level**: Team-shared prompts (`.github/prompts/`)
- **User-level**: Personal prompts synced via settings (`~/.copilot/prompts/`)
- **Invocation**: Always manual via slash command or command palette

### Example from Workspace

```yaml
---
description: "Analyze economic data by cleaning, calculating metrics,
  identifying trends, creating charts, and generating professional insights
  with risk assessment"
name: "Economic Data Analysis"
argument-hint: "Provide economic dataset or describe analysis goal"
agent: "agent"
---

# Economic Data Analysis

Analyze the provided economic data and produce a concise, professional output.

## Objective

Perform a complete economic data analysis workflow:
1. Clean and validate the data
2. Calculate key metrics
3. Identify trends and comparisons
4. Generate real chart images as PNG files
5. Produce insights, risks, limitations, and a concise report

## 1. Data Preparation

Clean and validate the dataset before analysis.

### Tasks
- Check for missing values, duplicates, invalid entries
- Validate data types and value ranges
- Ensure time-based data is in correct chronological order
- Document assumptions, fixes, or transformations
```

### Key Differences from Skills

- **Scope**: Prompts are single-task; skills are multi-step workflows
- **Complexity**: Prompts are simpler; skills bundle resources and scripts
- **Discovery**: Both appear as slash commands, but skills have broader scope
- **Bundled Resources**: Skills can include scripts/templates; prompts typically don't
- **Reusability**: Both are reusable, but skills are for complex workflows

### Common Pitfalls

- ❌ Multi-task prompts trying to do too much ("create and test and deploy")
- ❌ Vague descriptions that don't help users understand when to use
- ❌ Over-tooling (many tools when task only needs search or file access)
- ❌ Poor output format specification (result depends on structure)

---

## 5. MCP (Model Context Protocol)

### Purpose

An open-source standard protocol for connecting AI applications (like VS Code Copilot) to external systems, data sources, tools, and workflows. MCP acts as a bridge between AI agents and external integrations.

### What is MCP?

- **Standard Protocol**: Open specification for AI ↔ External System communication
- **USB-C for AI**: Standardized connection point like USB-C for electronic devices
- **Ecosystem Approach**: Build once, integrate everywhere (Claude, ChatGPT, VS Code, Cursor, etc.)

### When to Use MCP

- **Connect to external APIs**: GitHub, Jira, Slack, Google Workspace
- **Access databases**: PostgreSQL, MongoDB, cloud databases
- **Integrate services**: Google Calendar, Notion, Figma, Blender
- **Custom tools**: Business logic, legacy systems, proprietary services
- **Multi-provider**: Single implementation works across multiple AI platforms

### How MCP Fits in the Customization Layers

| Layer        | Scope           | Purpose                                 |
| ------------ | --------------- | --------------------------------------- |
| Instructions | Guidelines      | How to write/structure code             |
| Agents       | Personas        | Who does the work, what tools they have |
| Skills       | Workflows       | Multi-step procedures                   |
| Prompts      | Templates       | Single-task templates                   |
| **MCP**      | **Integration** | **What external systems to access**     |

**MCP complements other layers:**

```
Agent (persona) + Tools (including MCP servers) + Instructions (guidelines)
+ Skills/Prompts (workflows/templates) = Complete customization stack
```

### Architecture

**Components:**

- **MCP Client**: VS Code Copilot (requests resources, tools, sampling)
- **MCP Server**: External system integration (database, API, service)
- **Protocol**: Standardized request-response format over stdio or WebSocket

**Capabilities Provided by MCP Servers:**

1. **Resources**: Data access points
   - Files, database records, API responses
   - Example: "access user calendar events"

2. **Tools**: Operations the agent can call
   - Create, read, update, delete operations
   - Example: "create a new Jira ticket"

3. **Prompts**: Reusable prompt templates
   - Context injection, specialized workflows
   - Example: "analyze Git commit history"

### MCP Server Examples

| Service             | Capability                  | Use Case                              |
| ------------------- | --------------------------- | ------------------------------------- |
| **Calendar Server** | Read/write calendar events  | Schedule meetings, check availability |
| **Database Server** | Query/update database       | Data analysis, reporting              |
| **GitHub MCP**      | Access repos, issues, PRs   | Code review, issue tracking           |
| **Notion MCP**      | Read/write Notion databases | Documentation, wiki searches          |
| **Figma MCP**       | Access designs, components  | Design-to-code, asset extraction      |

### Configuration in VS Code

MCP servers are configured in VS Code's settings:

```json
{
  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "python",
        "args": ["-m", "mcp_server_github"],
        "env": {
          "GITHUB_TOKEN": "${env:GITHUB_TOKEN}"
        }
      },
      "postgres": {
        "command": "node",
        "args": ["./mcp-servers/postgres/index.js"],
        "env": {
          "DATABASE_URL": "${env:DATABASE_URL}"
        }
      }
    }
  }
}
```

### MCP in Prompts and Agents

**Reference MCP servers in tool definitions:**

```yaml
# In .agent.md or .prompt.md
tools:
  - github/* # All tools from GitHub MCP server
  - postgres/* # All tools from PostgreSQL server
  - read, search # Mixed with built-in tools
```

### Scope

- **Workspace-level**: Team-shared MCP server configurations (`.github/mcp/`, VS Code workspace settings)
- **User-level**: Personal MCP servers configured in user settings
- **Connection**: Always requires active MCP server implementation (running process or service)

### Key Characteristics

- **Non-deterministic**: Agent decides when/how to use MCP tools
- **External dependency**: Requires external system or service
- **Ecosystem approach**: Standard works across multiple AI platforms
- **Extensible**: Create custom MCP servers for any system
- **Security**: Uses environment variables for sensitive credentials

### Comparison with Instructions/Agents/Skills

| Aspect         | Instructions     | Agents           | Skills      | Prompts     | MCP                  |
| -------------- | ---------------- | ---------------- | ----------- | ----------- | -------------------- |
| **Purpose**    | Guidelines       | Personas         | Workflows   | Templates   | Integration          |
| **Scope**      | File/task        | Specialized role | Multi-step  | Single task | External system      |
| **Tools**      | None             | Restricted set   | Via scripts | Via config  | Unlimited (external) |
| **When**       | Always/on-demand | Manual/delegated | On-demand   | Manual      | Agent-triggered      |
| **Complexity** | Simple           | Medium           | Complex     | Medium      | High (external)      |

### Common Pitfalls

- ❌ Treating MCP as a replacement for agents/skills (different purposes)
- ❌ Forgetting to set up environment variables for credentials
- ❌ Assuming MCP eliminates need for other customization layers
- ❌ Building custom MCP servers without clear API specification

---

## Layer Interactions & Decision Flow

### How They Work Together

```
┌─────────────────────────────────────────────────────────┐
│  User Request / Slash Command                           │
└────────┬────────────────────────────────────────────────┘
         │
         ├─► PROMPTS (.prompt.md) ──► Single focused task
         │
         ├─► SKILLS (SKILL.md) ──────► Multi-step workflow
         │                             with bundled resources
         │
         └─► AGENT SELECTION ─────────┐
                                      │
         ┌────────────────────────────┘
         │
         ▼
    ┌─────────────────────────────────────────────┐
    │  Selected/Current Agent                     │
    │  - Specific tools set                       │
    │  - Possible subagent restrictions           │
    └─────┬───────────────────────────────────────┘
          │
          ├─► INSTRUCTIONS (.instructions.md)
          │   - Applied based on:
          │     • Current file matches `applyTo`
          │     • Task description matches keywords
          │
          ├─► AGENTS (subagent delegation)
          │   - When complex task needs specialist
          │   - Agent selection based on `description`
          │
          ├─► MCP TOOLS
          │   - External system integrations
          │   - Available based on agent config
          │
          └─► BUILT-IN TOOLS (read, edit, search, execute, etc.)
              - Availability based on agent configuration
```

### Decision Tree: Which Customization Layer?

```
Does this apply to MOST/ALL code in the project?
  └─► YES: Use AGENT INSTRUCTIONS (copilot-instructions.md)
  └─► NO:  Continue below

Is this specific to a file type or language?
  └─► YES: Use FILE INSTRUCTIONS (.instructions.md) with `applyTo`
  └─► NO:  Continue below

Is this a complex workflow that needs context isolation and
restricted tools based on role?
  └─► YES: Use CUSTOM AGENT (.agent.md)
  └─► NO:  Continue below

Is this a multi-step procedure with bundled scripts/templates?
  └─► YES: Use SKILL (SKILL.md folder with scripts/)
  └─► NO:  Continue below

Is this a single-task template with parameterized inputs?
  └─► YES: Use PROMPT (.prompt.md)
  └─► NO:  Continue below

Does this need external system integration (database, API, service)?
  └─► YES: Use MCP SERVER (mcp configuration + server implementation)
  └─► NO:  This might not be a customization need
```

### Example Workflow

**Scenario**: Team developing a Python web service with tests, security scanning, and GitHub integration

**Customization Stack:**

1. **Agent Instructions** (copilot-instructions.md)

   ```markdown
   # Project Guidelines

   ## Code Style

   See .github/instructions/coding-conventions.instructions.md

   ## Architecture

   - FastAPI for API layer
   - PostgreSQL for persistence
   - See docs/architecture.md

   ## Testing

   - pytest for unit/integration tests
   - Run: `pytest tests/ -v --cov=src`
   ```

2. **File Instructions** (.github/instructions/coding-conventions.instructions.md)

   ```yaml
   applyTo: "**/*.py"
   description: "Use when writing Python code..."
   ```

   Enforces Python naming, error handling, logging, async patterns, testing

3. **Custom Agent** (.github/agents/security-reviewer.agent.md)

   ```yaml
   tools: [read, search] # No execute, no edit
   description: "Use when reviewing code for security issues"
   hooks:
     PreToolUse:
       - type: command
         command: "./scripts/block-dangerous-cmds.sh"
   ```

   Specialized secure code reviewer agent

4. **Skill** (.github/skills/test-deployment/)

   ```
   SKILL.md - Multi-step testing and deployment workflow
   scripts/
     - run-tests.sh
     - security-scan.sh
     - deploy.sh
   ```

   Reusable deployment workflow

5. **Prompt** (.github/prompts/generate-test-cases.prompt.md)

   ```yaml
   description: "Generate test cases for selected code"
   tools: [search, read]
   ```

   Quick template for generating tests

6. **MCP Server** (mcp configuration in VS Code settings)
   ```json
   {
     "github": {...},
     "postgres": {...}
   }
   ```
   GitHub and database access for agents

**How they interact:**

- User requests code generation
- Agent instructions guide general approach
- File instructions auto-apply to Python files
- User can invoke skill `/test-deployment` for testing
- Custom security-reviewer agent can be delegated for security review
- MCP provides GitHub issue/PR context
- Prompt can generate specific test cases

---

## Scope Levels

### Workspace Scope

- **Location**: `.github/` folder (or root for `AGENTS.md`)
- **Sharing**: Committed to version control, shared with team
- **Sync**: Not synced across user's machines
- **Use case**: Team standards, project-specific customization

**Workspace files:**

```
.github/
├── copilot-instructions.md        # Or AGENTS.md at root
├── instructions/
│   ├── coding-conventions.instructions.md
│   └── testing.instructions.md
├── prompts/
│   ├── generate-test-cases.prompt.md
│   └── economic-data-analysis.prompt.md
├── agents/
│   └── security-reviewer.agent.md
└── skills/
    ├── test-deployment/
    │   └── SKILL.md
    └── data-analysis/
        └── SKILL.md
```

### User Scope

- **Location**: User profile folder (platform-dependent)
- **Sharing**: Synced across all workspaces via VS Code settings sync
- **Sync**: Cross-machine, cross-workspace
- **Use case**: Personal preferences, cross-project standards

**User files (stored in VS Code user profile):**

```
~/.copilot/  (or platform equivalent)
├── instructions/
│   └── my-coding-style.instructions.md
├── prompts/
│   └── my-favorite-prompt.prompt.md
├── agents/
│   └── my-custom-agent.agent.md
└── skills/
    └── my-workflow/
        └── SKILL.md
```

**Equivalent locations on different platforms:**

- **macOS**: `~/.config/Code/User/globalStorage/GitHub.copilot/`
- **Linux**: `~/.config/Code/User/globalStorage/GitHub.copilot/`
- **Windows**: `%APPDATA%\Code\User\globalStorage\GitHub.copilot\`

Or via `{{VSCODE_USER_PROMPTS_FOLDER}}` variable: `c:\Users\ADMIN\AppData\Roaming\Code\User\prompts`

---

## Best Practices & Patterns

### Do's ✅

1. **Use clear, keyword-rich descriptions**

   ```yaml
   description: "Use when writing Python code. Enforces naming conventions,
     error handling, logging, async patterns, and testing."
   ```

2. **Keep instructions focused on one concern**
   - Separate: testing.instructions.md, styling.instructions.md
   - Avoid: everything-guide.instructions.md

3. **Link to existing documentation**

   ```markdown
   ## Testing

   For detailed test patterns, see [tests/README.md](../../tests/README.md)
   ```

4. **Use relative paths for skill resources**

   ```markdown
   [Setup script](./scripts/setup.sh)
   [Reference guide](./references/patterns.md)
   ```

5. **Make agents minimal in scope**
   - One role, one set of capabilities
   - Avoid Swiss-army agents with everything

6. **Test discovery keywords**
   - Will agent find instruction via "Use when" phrases?
   - Use specific trigger words

### Don'ts ❌

1. **Avoid vague descriptions**
   - ❌ "Helpful coding tips"
   - ✅ "Use when implementing API endpoints in Express.js"

2. **Don't use `applyTo: "**"` unless truly universal\*\*
   - Burns context on every interaction
   - Use specific globs: `**/*.py`, `src/api/**`

3. **Don't mix concerns in one instruction file**
   - ❌ Testing + Styling + API design in one file
   - ✅ Separate: testing.instructions.md, styling.instructions.md

4. **Don't duplicate documentation**
   - ❌ Copy README content into instructions
   - ✅ Link to existing docs

5. **Don't create circular agent handoffs**
   - ❌ Agent A → Agent B → Agent A
   - ✅ Define clear delegation hierarchy

6. **Don't over-tool agents**
   - ❌ tools: [read, edit, search, execute, web, agent, todo]
   - ✅ tools: [read, search] (for researcher role)

---

## Quick Reference Table

| Layer            | File Type          | Location                 | Scope               | Trigger                        | Use For                                   |
| ---------------- | ------------------ | ------------------------ | ------------------- | ------------------------------ | ----------------------------------------- |
| **Instructions** | `.instructions.md` | `.github/instructions/`  | File/Task           | `applyTo` or keyword match     | Language/framework standards              |
| **Agents**       | `.agent.md`        | `.github/agents/`        | Specialized role    | Manual selection or delegation | Restricted workflows, role-based behavior |
| **Skills**       | `SKILL.md` folder  | `.github/skills/<name>/` | Multi-step workflow | `/skill-name` or auto-load     | Complex workflows with resources          |
| **Prompts**      | `.prompt.md`       | `.github/prompts/`       | Single task         | `/prompt-name`                 | One-off templated tasks                   |
| **MCP**          | Server config      | VS Code settings         | External system     | Agent-triggered                | API/database/service integration          |

| Layer            | Discovery              | Invocation             | Best For                    |
| ---------------- | ---------------------- | ---------------------- | --------------------------- |
| **Instructions** | On-demand via keywords | Auto when files match  | Consistent code patterns    |
| **Agents**       | Via picker or keywords | Manual or delegation   | Complex, isolated workflows |
| **Skills**       | Slash command or auto  | `/skill-name` or agent | Bundled procedures          |
| **Prompts**      | Slash command          | `/prompt-name`         | Quick templates             |
| **MCP**          | Via tools config       | Agent tool invocation  | External integrations       |

---

## Troubleshooting

### Instruction Not Being Applied

**Problem**: File instruction isn't loading for matching files

**Solutions**:

1. Check `applyTo` glob pattern matches your file
   - ✅ `applyTo: "**/*.py"` matches `/src/api/routes.py`
   - ❌ `applyTo: "src/api/**"` doesn't match `/api/routes.py`

2. Verify YAML syntax in frontmatter
   - Colons must be quoted if in description: `"Use when: ..."`
   - Use spaces, not tabs
   - `name` field should match use case

3. Check file is in cache context
   - Instructions only apply to files in current context
   - Open the file to trigger instruction loading

### Skill Not Appearing in Slash Commands

**Problem**: SKILL.md not showing in `/` slash command list

**Solutions**:

1. Verify folder name matches `name` field
   - Folder: `.github/skills/test-deploy/`
   - SKILL.md: `name: test-deploy` ✅

2. Check `user-invocable` isn't false
   - Default is true, should appear unless explicitly set

3. Verify SKILL.md exists and has frontmatter
   - File must be named exactly `SKILL.md`
   - Must have YAML frontmatter with `name` and `description`

### Agent Not Being Delegated

**Problem**: Custom agent isn't being invoked as subagent

**Solutions**:

1. Check `disable-model-invocation: false`
   - If set to true, agent can't be auto-invoked

2. Verify description matches task
   - Agent needs keyword match in `description` for discovery
   - "Use when..." pattern helps discovery

3. Check parent agent allows subagent invocation
   - Parent may have `agents: [agent1, agent2]` restriction
   - Or `agents: []` which disallows all subagents

### MCP Tools Not Available

**Problem**: MCP server tools not showing in agent's tool list

**Solutions**:

1. Check MCP server is running
   - MCP requires active server process
   - Check VS Code settings for MCP configuration

2. Verify tool syntax in agent/prompt config
   - ✅ `tools: [github/*, execute, read]`
   - ❌ `tools: [github, execute, read]` (missing wildcard)

3. Check environment variables
   - MCP servers often need auth tokens
   - Verify `env` variables are set in mcp config

---

## Summary

The VS Code Copilot customization system provides five complementary layers:

1. **Instructions**: Guidelines for consistent behavior (file-type or task-based)
2. **Agents**: Specialized personas with restricted tools and capabilities
3. **Skills**: Multi-step workflows with bundled resources
4. **Prompts**: Single-task templates for quick generation
5. **MCP**: External system integration for APIs, databases, services

**Together they enable:**

- Team standards through version-controlled instructions
- Role-based tool restrictions through agents
- Complex automation through skills
- Parameterized templates through prompts
- External integrations through MCP

Use the decision tree and quick reference to choose the right layer for your customization needs.
