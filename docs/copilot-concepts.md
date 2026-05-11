# Copilot Customization Layers

VS Code Copilot provides five distinct customization mechanisms that work together to extend and configure agent behavior. This document explains each layer, their purposes, and how they interact.

---

## Quick Reference

| Layer            | Purpose                                                   | File Pattern              | Scope                     | Trigger                                               |
| ---------------- | --------------------------------------------------------- | ------------------------- | ------------------------- | ----------------------------------------------------- |
| **Instructions** | Guidelines for specific file types or tasks               | `.instructions.md`        | File/task-based           | Automatic (via `applyTo`) or on-demand (via keywords) |
| **Agents**       | Specialized personas with specific tools and capabilities | `.agent.md`               | Workflow/role-based       | Manual selection or auto-delegation                   |
| **Skills**       | Multi-step workflows with bundled scripts and resources   | `SKILL.md` (in folder)    | Repeatable procedures     | Slash command (`/skill-name`) or auto-invoke          |
| **Prompts**      | Single focused tasks with parameterized inputs            | `.prompt.md`              | Template/generation tasks | Slash command (`/prompt-name`)                        |
| **MCP**          | External system integration (APIs, databases, services)   | Configuration in settings | Integration layer         | Agent-triggered when configured                       |

---

## 1. Instructions (.instructions.md)

### What They Do

Guidelines and conventions that apply to specific file types, languages, or tasks. They're applied automatically or on-demand when relevant.

### When to Use

- **Language-specific standards** (Python naming conventions, error handling patterns)
- **Framework guidelines** (Express.js API patterns, React component structure)
- **Task-based guidance** (triggered when agent detects relevant keywords like "Use when")
- **File-specific rules** (applied automatically to matching file types via glob patterns)

### Structure

```yaml
---
description: "Use when [task]. [What this teaches]."  # Required; used for discovery
name: "Instruction Name"                              # Optional
applyTo: "**/*.py"                                    # Optional; auto-applies to matching files
---

# Title

## Key Guidelines
- Rule 1
- Rule 2

## Examples
...code examples...
```

### Discovery Modes

1. **Automatic** (`applyTo` pattern): Applied when files match glob pattern
2. **On-demand** (keywords): Agent discovers via "Use when" phrases in `description`
3. **Manual**: User explicitly adds via "Add Context" → "Instructions"

### Scope

- **Workspace**: Shared via `.github/instructions/*.instructions.md`
- **User**: Personal, synced across devices

### Example

```yaml
---
description: "Use when writing Python code. Enforces naming conventions,
  error-handling patterns, logging standards, async/await best practices,
  and testing conventions."
applyTo: "**/*.py"
---

# Python Coding Conventions

## Naming Conventions
- Variables/functions: `snake_case`
- Classes: `PascalCase`

## Error Handling
- Catch specific exceptions, not bare `except:`
```

### Key Differences from Other Layers

- **No tool control** (unlike agents)
- **No workflow** (unlike skills)
- **Lightweight and on-demand** (loaded only when relevant)

---

## 2. Agents (.agent.md)

### What They Do

Specialized AI personas with specific tools, instructions, and capabilities. Agents control **what** the AI can do for a specific role or workflow.

### When to Use

- **Role-based customization** (Code Reviewer, Security Auditor, Documentation Writer)
- **Tool isolation** (restrict dangerous operations, provide only needed tools)
- **Context separation** (complex tasks that benefit from isolated context)
- **Multi-stage workflows** (handoff between agents with different capabilities)

### Structure

```yaml
---
description: "Use when [specific task]. Specialized in [what this agent does]."
name: "Agent Name"
tools: [read, search, edit]                  # Control what agent can do
model: "Claude Sonnet 4.5 (copilot)"        # Optional model choice
user-invocable: true                        # Show in agent picker
disable-model-invocation: false             # Allow as subagent
agents: [agent1, agent2]                    # Restrict subagents
---

You are a specialist at [specific task]. Your job is to [purpose].

## Constraints
- DO: [what this agent should do]
- DON'T: [what this agent should never do]

## Approach
[How this agent works]
```

### Tool Control

Agents define what capabilities are available:

| Tool      | Purpose                |
| --------- | ---------------------- |
| `read`    | Read files             |
| `edit`    | Modify files           |
| `search`  | Search workspace       |
| `execute` | Run shell commands     |
| `web`     | Fetch URLs, web search |
| `mcp:*`   | External integrations  |

### Invocation

- **Manual**: User selects from agent picker
- **Automatic**: Agent delegates to specialized subagent when needed

### Scope

- **Workspace**: Shared via `.github/agents/*.agent.md`
- **User**: Personal agents synced via settings

### Example

```yaml
---
description: "Use when reviewing code for security vulnerabilities.
  Identifies dangerous patterns and recommends fixes."
name: "Security Reviewer"
tools: [read, search]  # No execute, no edit - safe read-only inspection
user-invocable: false  # Only invoked as subagent
---

You are a security code reviewer. Your role is to identify vulnerabilities...

## Constraints
- ONLY read and analyze code
- DO NOT execute code
- DO NOT modify files
```

### Key Differences from Other Layers

- **Tool control** (agents restrict capabilities; instructions don't)
- **Persona-based** (agents are roles; instructions are guidelines)
- **Delegation support** (agents can invoke other agents)

---

## 3. Skills (SKILL.md)

### What They Do

Multi-step workflows with bundled scripts, templates, and reference documentation. Skills are repeatable procedures that appear as slash commands.

### When to Use

- **Complex multi-step procedures** (testing, deployment, data analysis)
- **Bundled resources needed** (scripts, templates, configuration examples)
- **Reusable workflows** (invoked multiple times with different inputs)
- **User-triggered automation** (appears as `/skill-name` in chat)

### Structure

```
.github/skills/skill-name/
├── SKILL.md                    # Required; same name as folder
├── scripts/                    # Executable code
│   ├── setup.sh
│   └── deploy.sh
├── references/                 # Detailed documentation
│   ├── patterns.md
│   └── troubleshooting.md
└── assets/                     # Templates, boilerplate
    └── config.json
```

**SKILL.md frontmatter:**

```yaml
---
name: skill-name                    # Folder name must match
description: "What and when to use" # Max 1024 chars
argument-hint: "Optional input hint"
user-invocable: true               # Show as slash command
---

# Skill Title

## When to Use
- Scenario 1
- Scenario 2

## Procedure
1. [Step 1]
2. [Step 2]

## References
- [Detailed guide](./references/guide.md)
```

### Progressive Loading

1. **Discovery** (~100 tokens): Agent reads name and description
2. **Instruction** (<5000 tokens): Loads SKILL.md when relevant
3. **Resources**: Reference files load only when referenced

### Invocation

- **Manual**: Type `/skill-name` in chat
- **Automatic**: Agent invokes when task matches description

### Scope

- **Workspace**: `.github/skills/<name>/`
- **User**: `~/.copilot/skills/<name>/`

### Example

```yaml
---
name: test-deployment
description: "Run automated tests and deploy to production.
  Use for validation before release."
---

# Test & Deploy Workflow

## Procedure
1. Run tests: `./scripts/test.sh`
2. Security scan: `./scripts/security-scan.sh`
3. Deploy: `./scripts/deploy.sh`
```

### Key Differences from Other Layers

- **Bundled resources** (skills include scripts; prompts don't)
- **Multi-step** (skills orchestrate workflows; prompts are single-task)
- **Reusable procedures** (skills for complex automation)

---

## 4. Prompts (.prompt.md)

### What They Do

Single-task templates with parameterized inputs. Prompts are invoked on-demand via slash commands for specific generation tasks.

### When to Use

- **One-off generation tasks** (generate test cases, create README, write summaries)
- **Parameterized workflows** (same task, different inputs)
- **Quick templates** (simpler than skills; no bundled resources)
- **Consistent output** (tasks where output format matters)

### Structure

```yaml
---
description: "Optional, improves discovery"
name: "Prompt Name"
argument-hint: "Task input description"
agent: "agent"                     # Which agent to use
tools: [search, web, execute]     # Available tools
model: "Claude Sonnet 4.5 (copilot)"  # Model choice
---

Generate [what this prompt does]

## Output Format
- [Format details]
- [Constraints]
```

### Invocation

- **Manual**: Type `/` → select prompt from list
- **Command**: Run "Chat: Run Prompt..."
- **Keyboard**: Open prompt file and click play button

### Scope

- **Workspace**: `.github/prompts/<name>.prompt.md`
- **User**: `~/.copilot/prompts/<name>.prompt.md`

### Example

```yaml
---
description: "Generate test cases for selected code"
tools: [search, read]
---

Generate comprehensive test cases for the provided code:
- Include edge cases and error scenarios
- Follow existing test patterns
- Use descriptive test names

## Output Format
- Test class: `test_<feature>.py`
- Test method: `test_<scenario>`
```

### Key Differences from Other Layers

- **Single-task** (prompts are one thing; skills do multiple steps)
- **Simple** (prompts are templates; no bundled resources)
- **Quick generation** (good for repeated single tasks)

---

## 5. MCP (Model Context Protocol)

### What It Is

An open-source standard protocol for connecting AI applications to external systems, APIs, databases, and services. MCP acts as a bridge between Copilot agents and external integrations.

### When to Use

- **Connect to external APIs** (GitHub, Jira, Slack, Google Workspace)
- **Access databases** (PostgreSQL, MongoDB, cloud databases)
- **Integrate services** (Notion, Figma, Google Calendar, Blender)
- **Custom systems** (proprietary tools, legacy systems, business logic)
- **Cross-platform** (same implementation works with Claude, ChatGPT, VS Code, Cursor)

### How MCP Fits

MCP is the **integration layer** that provides agents with access to external systems:

```
Agent (persona) + Tools (including MCP servers) + Instructions (guidelines)
↓
Agent can now call external APIs, query databases, etc.
```

### Architecture

**Three components:**

1. **MCP Client**: VS Code Copilot (requests resources, tools, operations)
2. **MCP Server**: External system connection (database, API, service)
3. **Protocol**: Standardized request-response over stdio or WebSocket

**What MCP provides:**

- **Resources**: Data access (files, database records, calendar events)
- **Tools**: Operations agent can call (create, read, update, delete)
- **Prompts**: Reusable prompt templates with context injection

### Configuration

MCP servers are configured in VS Code settings:

```json
{
  "mcp": {
    "servers": {
      "github": {
        "command": "python",
        "args": ["-m", "mcp_server_github"],
        "env": { "GITHUB_TOKEN": "${env:GITHUB_TOKEN}" }
      },
      "postgres": {
        "command": "node",
        "args": ["./mcp-servers/postgres/index.js"],
        "env": { "DATABASE_URL": "${env:DATABASE_URL}" }
      }
    }
  }
}
```

### Using MCP in Agents and Prompts

Reference MCP servers in tool definitions:

```yaml
# In .agent.md or .prompt.md
tools:
  - github/* # All tools from GitHub server
  - postgres/* # All tools from PostgreSQL server
  - read, search # Mixed with built-in tools
```

### Examples of MCP Servers

| Service         | Capability                             | Use Case                               |
| --------------- | -------------------------------------- | -------------------------------------- |
| GitHub          | Access repos, issues, PRs, code review | CI/CD integration, code analysis       |
| PostgreSQL      | Query/update database                  | Data analysis, reporting               |
| Notion          | Read/write knowledge base              | Documentation search, wiki updates     |
| Figma           | Access designs, components             | Design-to-code, asset extraction       |
| Google Calendar | Read/write events                      | Meeting scheduling, availability check |

### Scope

- **Workspace**: Shared MCP configuration (`.github/mcp/` or workspace settings)
- **User**: Personal MCP servers configured in user settings

### Key Characteristics

- **External dependency**: Requires external system or running service
- **Non-deterministic**: Agent decides when/how to use available tools
- **Ecosystem standard**: Works across multiple AI platforms (Claude, ChatGPT, VS Code, Cursor)
- **Extensible**: Create custom MCP servers for any system
- **Security**: Uses environment variables for sensitive credentials

---

## How They Work Together

### Interaction Flow

```
User Request
    ↓
Prompt or Skill Selected
    ↓
Agent Assigned (or current agent)
    ↓
┌─ Instructions Applied (file/task matching)
├─ Tools Available (from agent config + MCP servers)
├─ Subagent Delegation (if needed)
└─ Built-in Tools (read, edit, search, execute)
    ↓
Task Executed
```

### Decision Tree

```
Is this project-wide guidance?
  ✓ YES → Agent Instructions (copilot-instructions.md)
  ✗ NO  → Continue

Is this specific to file types/languages?
  ✓ YES → File Instructions (.instructions.md with applyTo)
  ✗ NO  → Continue

Is this a role with restricted capabilities?
  ✓ YES → Custom Agent (.agent.md)
  ✗ NO  → Continue

Is this a multi-step workflow with resources?
  ✓ YES → Skill (SKILL.md folder with scripts/)
  ✗ NO  → Continue

Is this a single-task template?
  ✓ YES → Prompt (.prompt.md)
  ✗ NO  → Continue

Does this need external system access?
  ✓ YES → MCP Server (configure + implement)
```

### Example: Python Web Service Team

**The customization stack:**

1. **Agent Instructions** (`copilot-instructions.md`)
   - Project architecture, tool choices, deployment process

2. **File Instructions** (`.github/instructions/coding-conventions.instructions.md`)
   - Auto-applies to all `**/*.py` files
   - Python naming, error handling, testing standards

3. **Custom Agent** (`.github/agents/security-reviewer.agent.md`)
   - Read-only access only
   - Can delegate security audits without edit access

4. **Skill** (`.github/skills/test-deployment/`)
   - Multi-step: run tests → security scan → deploy
   - Includes scripts and troubleshooting guides

5. **Prompt** (`.github/prompts/generate-test-cases.prompt.md`)
   - Quick template for generating tests from code

6. **MCP** (GitHub MCP server in settings)
   - Access GitHub issues, PRs, and integration status

**Result**: When a developer writes Python code, they get consistent guidance, can run tests easily, delegate security reviews safely, and integrate with GitHub—all without context switching.

---

## Scope & File Organization

### Workspace Scope

Committed to version control, shared with team:

```
.github/
├── copilot-instructions.md
├── instructions/
│   ├── coding-conventions.instructions.md
│   └── testing.instructions.md
├── agents/
│   └── security-reviewer.agent.md
├── prompts/
│   └── generate-test-cases.prompt.md
└── skills/
    └── test-deployment/
        └── SKILL.md
```

### User Scope

Personal, synced across devices via VS Code settings:

```
~/.copilot/  (or via {{VSCODE_USER_PROMPTS_FOLDER}})
├── instructions/
│   └── my-style.instructions.md
├── agents/
│   └── my-agent.agent.md
├── prompts/
│   └── my-prompt.prompt.md
└── skills/
    └── my-skill/
        └── SKILL.md
```

**Platform paths:**

- **Windows**: `%APPDATA%\Code\User\globalStorage\GitHub.copilot\`
- **macOS**: `~/.config/Code/User/globalStorage/GitHub.copilot/`
- **Linux**: `~/.config/Code/User/globalStorage/GitHub.copilot/`

---

## Best Practices

### Do's ✅

1. **Use clear, specific descriptions** with "Use when..." triggers
   - ✓ "Use when writing Python async code"
   - ✗ "Helpful Python tips"

2. **Keep each layer focused on one concern**
   - Separate: `testing.instructions.md`, `styling.instructions.md`
   - Avoid: `everything.instructions.md`

3. **Use relative paths** in skills and prompts
   - ✓ `[script](./scripts/deploy.sh)`
   - ✗ `[script](/full/path/to/script.sh)`

4. **Test discovery keywords** by asking: Will agent find this via "Use when" phrases?

5. **Make agents minimal** in scope (one role, one capability set)

### Don'ts ❌

1. **Avoid vague descriptions** that don't enable discovery

2. **Don't use overly broad `applyTo` patterns** unless truly universal
   - ✓ `applyTo: "**/*.py"` or `src/api/**`
   - ✗ `applyTo: "**"`

3. **Don't duplicate documentation** (link instead)

4. **Don't create Swiss-army agents** with everything

5. **Don't forget environment variables** for MCP credentials

---

## Summary

| Layer            | Best For                        | Key Feature                              |
| ---------------- | ------------------------------- | ---------------------------------------- |
| **Instructions** | Code standards, guidelines      | Auto-apply via file patterns or keywords |
| **Agents**       | Specialized roles, tool control | Restrict capabilities for safety/focus   |
| **Skills**       | Multi-step workflows            | Bundled scripts and procedures           |
| **Prompts**      | Single focused tasks            | Quick templates with slash commands      |
| **MCP**          | External integrations           | API, database, service access            |

Use these layers together to create a comprehensive customization system that guides behavior, controls capabilities, and integrates external systems—all while keeping Copilot focused and effective.
