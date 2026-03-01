---
name: subagent-creator
description: Helps users create, configure, and manage custom subagents for Gemini CLI. Use this skill when a user asks to create or configure a subagent, an agent, or a specialized agent.
---
# Subagent Creator

This skill guides the process of creating custom subagents for Gemini CLI.

## About Subagents

Subagents are specialized expert agents defined as Markdown files with YAML frontmatter. They operate as tools for the main Gemini CLI agent.

### 1. Prerequisites (Enable Agents)

First, verify or ask the user to verify that subagents are enabled in their `settings.json`. It must contain:

```json
{
  "experimental": {
    "enableAgents": true
  }
}
```

### 2. Location

Subagents can be defined in two locations:
- **Project-level:** `.gemini/agents/*.md` (Shared with the project team)
- **User-level:** `~/.gemini/agents/*.md` (Available across all projects)

### 3. File Structure

A subagent file requires YAML frontmatter followed by a Markdown body that serves as the System Prompt.

**Example File (`.gemini/agents/security-auditor.md`):**

```markdown
---
name: security-auditor
description: Specialized in finding security vulnerabilities in code.
tools:
  - read_file
  - grep_search
model: gemini-2.5-pro
temperature: 0.2
max_turns: 10
---

You are a Security Auditor. Your job is to analyze code for potential vulnerabilities.
Focus on SQL Injection, XSS, and hardcoded credentials.
```

### 4. Configuration Fields (YAML)

- **`name`** (Required): Unique identifier used as the tool name. Use lowercase and hyphens.
- **`description`** (Required): Describes the agent's expertise so the main agent knows when to "hire" it. Be clear and specific.
- **`tools`** (Optional): List of tools this subagent can access (e.g., `read_file`, `grep_search`, `run_shell_command`, `write_file`).
- **`model`** (Optional): Specific model to use (e.g., `gemini-2.5-pro`).
- **`max_turns`** (Optional): Maximum conversation turns (default: 15).

### Workflow for Creating a Subagent

**Important Rule:** All communication and interactions with the user MUST be conducted in Japanese.

1. **Understand Requirements:** Ask the user what the subagent should do, what tools it needs, and where it should be stored (project or user level).
2. **Ensure Settings are enabled:** Remind them that `settings.json` needs `"experimental": {"enableAgents": true}`.
3. **Generate Definition:** Propose the YAML and system prompt based on their requirements.
4. **Create File:** Ask for permission to create the `.md` file in the chosen location using the `write_file` tool.
