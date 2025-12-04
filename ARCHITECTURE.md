# Agent OS Architecture

> **Quick Answer:** Prompt definitions are located in `~/agent-os/profiles/default/` (source templates) and get compiled into your project's `.claude/` or `agent-os/` directories. See [Where Are the Prompt Definitions?](#where-are-the-prompt-definitions) for details.

## Table of Contents

- [Overview](#overview)
- [How Agent OS Works](#how-agent-os-works)
- [Directory Structure](#directory-structure)
- [Core Components](#core-components)
- [Where Are the Prompt Definitions?](#where-are-the-prompt-definitions)
- [The Templating System](#the-templating-system)
- [The Compilation Process](#the-compilation-process)
- [Key Scripts](#key-scripts)
- [Customization Guide](#customization-guide)
- [Best Practices](#best-practices)
- [Workflow Examples](#workflow-examples)
- [Troubleshooting](#troubleshooting)

## Overview

Agent OS is a spec-driven agentic development framework that transforms AI coding agents into productive developers. It provides structured workflows, standards, and prompts that guide AI agents to produce quality code consistently.

## How Agent OS Works

Agent OS operates on a **profile-based template system** that gets **compiled and installed** into your project. Here's the workflow:

1. **Installation**: Run `~/agent-os/scripts/project-install.sh` from your project directory
2. **Compilation**: Templates from `~/agent-os/profiles/[profile-name]/` are compiled with your configuration settings
3. **Installation**: Compiled prompts are installed into your project (`.claude/` or `agent-os/` directories)
4. **Usage**: AI agents use the installed commands and reference compiled workflows/standards

## Directory Structure

```
agent-os/
├── config.yml                    # Global configuration
├── scripts/                      # Installation and compilation scripts
│   ├── project-install.sh       # Main installation script
│   ├── project-update.sh        # Update existing installations
│   ├── common-functions.sh      # Core compilation logic
│   ├── base-install.sh          # Base installation script
│   └── create-profile.sh        # Create new profiles
└── profiles/                     # Profile templates
    └── default/                  # Default profile
        ├── agents/              # AI agent definitions (for Claude Code subagents)
        ├── commands/            # Command definitions (main entry points)
        ├── workflows/           # Reusable workflow components
        ├── standards/           # Coding standards and conventions
        └── claude-code-skill-template.md
```

## Core Components

### 1. Profiles (`profiles/[profile-name]/`)

Profiles are template collections that define how Agent OS behaves. The `default` profile includes:

- **Agents** (`agents/`) - Specialized AI agent definitions for Claude Code's subagent system
- **Commands** (`commands/`) - Top-level commands that users invoke
- **Workflows** (`workflows/`) - Reusable workflow components embedded into commands/agents
- **Standards** (`standards/`) - Project-specific coding standards and conventions

### 2. Agents (`profiles/default/agents/`)

Agent definitions are markdown files with YAML frontmatter that define specialized AI agents:

```markdown
---
name: implementer
description: Use proactively to implement a feature
tools: Write, Read, Bash, WebFetch, Playwright
color: red
model: inherit
---

[Agent prompt instructions here]
{{workflows/implementation/implement-tasks}}
{{standards/*}}
```

**Available agents:**
- `implementer.md` - Implements features following task lists
- `product-planner.md` - Creates product documentation
- `spec-writer.md` - Writes detailed specifications
- `spec-shaper.md` - Shapes high-level spec outlines
- `spec-initializer.md` - Initializes new specifications
- `spec-verifier.md` - Verifies specification quality
- `tasks-list-creator.md` - Creates implementation task lists
- `implementation-verifier.md` - Verifies implementations

### 3. Commands (`profiles/default/commands/`)

Commands are the main entry points that users invoke. Each command can have:
- **single-agent/** - A single workflow with optional numbered phases
- **multi-agent/** - Delegates to specialized subagents

```
commands/
├── implement-tasks/
│   ├── single-agent/
│   │   ├── implement-tasks.md          # Main command file
│   │   ├── 1-determine-tasks.md        # Phase 1
│   │   ├── 2-implement-tasks.md        # Phase 2
│   │   └── 3-verify-implementation.md  # Phase 3
│   └── multi-agent/
│       └── implement-tasks.md          # Delegates to subagents
├── write-spec/
├── shape-spec/
├── create-tasks/
├── plan-product/
├── orchestrate-tasks/
└── improve-skills/
```

### 4. Workflows (`profiles/default/workflows/`)

Workflows are reusable prompt components that get embedded into commands and agents:

```
workflows/
├── implementation/
│   ├── implement-tasks.md
│   ├── compile-implementation-standards.md
│   └── verification/
├── specification/
│   ├── write-spec.md
│   ├── initialize-spec.md
│   └── verify-spec.md
└── planning/
    ├── gather-product-info.md
    ├── create-product-mission.md
    └── create-product-roadmap.md
```

### 5. Standards (`profiles/default/standards/`)

Standards define project-specific coding conventions that get injected into prompts:

```
standards/
├── global/
│   ├── tech-stack.md        # Technology stack definitions
│   ├── coding-style.md      # Code style guidelines
│   ├── conventions.md       # Naming and structure conventions
│   ├── error-handling.md    # Error handling patterns
│   ├── commenting.md        # Code documentation standards
│   └── validation.md        # Input validation standards
├── frontend/
│   ├── components.md        # Component patterns
│   ├── css.md              # CSS/styling conventions
│   ├── accessibility.md     # A11y requirements
│   └── responsive.md       # Responsive design patterns
├── backend/
│   ├── api.md              # API design standards
│   ├── models.md           # Data model patterns
│   ├── queries.md          # Database query patterns
│   └── migrations.md       # Database migration guidelines
└── testing/
    └── test-writing.md     # Testing standards
```

## Where Are the Prompt Definitions?

### Source Templates (Read-Only)
Located in `~/agent-os/profiles/[profile-name]/`:
- **Agents**: `~/agent-os/profiles/default/agents/*.md`
- **Commands**: `~/agent-os/profiles/default/commands/*/[single|multi]-agent/*.md`
- **Workflows**: `~/agent-os/profiles/default/workflows/**/*.md`
- **Standards**: `~/agent-os/profiles/default/standards/**/*.md`

### Compiled & Installed (In Your Project)
After running `project-install.sh`, prompts are installed in your project:

**For Claude Code** (when `claude_code_commands: true`):
- Commands: `.claude/commands/agent-os/[command-name]/`
- Agents: `.claude/agents/agent-os/` (when `use_claude_code_subagents: true`)
- Skills: `.claude/skills/agent-os/` (when `standards_as_claude_code_skills: true`)

**For Other Tools** (when `agent_os_commands: true`):
- Commands: `agent-os/commands/`
- Standards: `agent-os/standards/`

## The Templating System

Agent OS uses a custom templating syntax for composing prompts:

### Template Syntax

#### 1. Workflow Embedding (`{{workflows/path}}`)
Embeds content from workflow files:
```markdown
{{workflows/implementation/implement-tasks}}
```

#### 2. Standards Injection (`{{standards/pattern}}`)
Injects standards files matching a pattern:
```markdown
{{standards/*}}                    # All standards
{{standards/global/*}}             # All global standards
{{standards/backend/api}}          # Specific standard
```

#### 3. Phase References (`{{@agent-os/path}}`)
References command phase files (in single-agent commands):
```markdown
{{PHASE 1: @agent-os/commands/implement-tasks/1-determine-tasks.md}}
```

#### 4. Conditional Blocks
Control content based on configuration:

```markdown
{{IF use_claude_code_subagents}}
Content shown only when subagents are enabled
{{ENDIF use_claude_code_subagents}}

{{UNLESS standards_as_claude_code_skills}}
Content shown when NOT using Claude Code Skills
{{ENDUNLESS standards_as_claude_code_skills}}
```

**Available conditions:**
- `use_claude_code_subagents` - Whether Claude Code subagents are enabled
- `standards_as_claude_code_skills` - Whether standards are provided as Skills
- `compiled_single_command` - Used internally for phase file compilation

### Example: How a Command Gets Compiled

**Source** (`profiles/default/commands/implement-tasks/single-agent/implement-tasks.md`):
```markdown
Follow this process:

{{PHASE 1: @agent-os/commands/implement-tasks/1-determine-tasks.md}}
{{PHASE 2: @agent-os/commands/implement-tasks/2-implement-tasks.md}}
```

**Phase 2** (`2-implement-tasks.md`):
```markdown
Implement following this workflow:
{{workflows/implementation/implement-tasks}}

{{UNLESS standards_as_claude_code_skills}}
Follow these standards:
{{standards/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
```

**Compiled Result** (`.claude/commands/agent-os/implement-tasks/implement-tasks.md`):
```markdown
Follow this process:

[Contents of 1-determine-tasks.md inserted here]

[Contents of 2-implement-tasks.md with all templates expanded:]
Implement following this workflow:
[Contents of workflows/implementation/implement-tasks.md inserted]

Follow these standards:
[Contents of all standards files inserted]
```

## The Compilation Process

### 1. Configuration Loading (`config.yml`)

```yaml
claude_code_commands: true              # Install Claude Code commands?
agent_os_commands: false                # Install generic commands?
use_claude_code_subagents: true         # Use Claude Code subagents?
standards_as_claude_code_skills: false  # Use Skills for standards?
profile: default                        # Which profile to use
```

### 2. Template Processing (`scripts/common-functions.sh`)

The `compile_agent()` and `compile_command()` functions:

1. **Load source file** from profile
2. **Process conditionals** (IF/UNLESS blocks) based on config
3. **Expand workflows** - Recursively replace `{{workflows/...}}` references
4. **Inject standards** - Replace `{{standards/...}}` with file contents
5. **Process PHASE tags** - Embed or reference phase files
6. **Write compiled output** to project directory

### 3. Installation Locations

**Claude Code Structure:**
```
.claude/
├── commands/
│   └── agent-os/
│       ├── implement-tasks/
│       │   └── implement-tasks.md
│       ├── write-spec/
│       └── ...
├── agents/
│   └── agent-os/
│       ├── implementer.md
│       ├── spec-writer.md
│       └── ...
└── skills/                    # If standards_as_claude_code_skills: true
    └── agent-os/
        ├── coding-style.md
        └── ...
```

**Generic Structure:**
```
agent-os/
├── commands/
│   ├── implement-tasks.md
│   ├── write-spec.md
│   └── ...
└── standards/                 # If NOT using Claude Code Skills
    ├── global/
    ├── frontend/
    └── ...
```

## Key Scripts

### `project-install.sh`
Main installation script - compiles and installs Agent OS into your project.

**Usage:**
```bash
cd /path/to/your/project
~/agent-os/scripts/project-install.sh [options]
```

**Key options:**
- `--profile [name]` - Use specific profile
- `--claude-code-commands true/false` - Enable/disable Claude Code commands
- `--use-claude-code-subagents true/false` - Enable/disable subagents
- `--agent-os-commands true/false` - Enable/disable generic commands
- `--standards-as-claude-code-skills true/false` - Use Skills for standards
- `--dry-run` - Preview without making changes

### `project-update.sh`
Updates existing installation - recompiles with new settings or profile changes.

### `common-functions.sh`
Core compilation logic including:
- `compile_agent()` - Compiles agent definitions
- `compile_command()` - Compiles command definitions
- `process_workflows()` - Recursively expands workflow references
- `process_conditionals()` - Evaluates IF/UNLESS blocks
- `process_standards()` - Injects standards files

### `base-install.sh`
Installs Agent OS base files to `~/agent-os/` (first-time setup).

### `create-profile.sh`
Creates new custom profiles based on the default profile.

## Customization Guide

### 1. Customize Standards

Edit files in your **project's** installed standards:
```bash
# If using Claude Code Skills:
.claude/skills/agent-os/[standard-name].md

# If NOT using Skills:
agent-os/standards/[category]/[standard-name].md
```

Or edit the source templates in `~/agent-os/profiles/default/standards/` and run `project-update.sh`.

### 2. Create Custom Profile

```bash
~/agent-os/scripts/create-profile.sh --profile my-custom-profile
```

This copies the default profile to `~/agent-os/profiles/my-custom-profile/` for customization.

### 3. Modify Commands/Agents

Edit templates in `~/agent-os/profiles/[profile]/` then run:
```bash
cd /path/to/project
~/agent-os/scripts/project-update.sh --overwrite-all
```

## Best Practices

1. **Start with defaults** - Use the default profile before customizing
2. **Customize standards first** - Standards are the easiest to customize and most impactful
3. **Keep standards current** - Update as your project evolves
4. **Use project-update** - Run after modifying templates to recompile
5. **Version control** - Commit `.claude/` or `agent-os/` directories to share with team
6. **Document changes** - Add notes in your standards explaining project-specific decisions

## Workflow Examples

### Implementing a Feature

1. User runs: `.claude/commands/agent-os/write-spec/write-spec.md` (Claude Code)
2. Agent reads prompt containing:
   - Instructions from the command file
   - Embedded workflow steps from `workflows/specification/`
   - Project standards from `standards/`
3. Agent creates `agent-os/specs/[feature]/spec.md`

4. User runs: `.claude/commands/agent-os/create-tasks/create-tasks.md`
5. Agent creates `agent-os/specs/[feature]/tasks.md`

6. User runs: `.claude/commands/agent-os/implement-tasks/implement-tasks.md`
7. Agent (or subagent) implements following:
   - Task list in `tasks.md`
   - Specifications in `spec.md`
   - Workflows from `workflows/implementation/`
   - Standards from `standards/`

### Product Planning

1. User runs: `.claude/commands/agent-os/plan-product/plan-product.md`
2. Agent follows multi-phase workflow:
   - Gathers product requirements (interactive)
   - Creates `agent-os/product/mission.md`
   - Creates `agent-os/product/roadmap.md`
   - Documents tech stack

## Troubleshooting

### Prompts not updated after template changes
Run `project-update.sh --overwrite-all` to force recompilation.

### Standards not appearing in prompts
Check `config.yml` - if `standards_as_claude_code_skills: true`, standards go in `.claude/skills/` not embedded in prompts.

### Subagents not working
Ensure `use_claude_code_subagents: true` and Claude Code supports subagents.

### Missing workflow content
Check for circular references or missing workflow files in profile.

## Further Resources

- **Documentation**: https://buildermethods.com/agent-os
- **Changelog**: [CHANGELOG.md](CHANGELOG.md)
- **Configuration Reference**: [config.yml](config.yml)
- **Default Profile**: [profiles/default/](profiles/default/)

---

## Summary: Where to Find Prompt Definitions

| Component | Source Templates | Compiled Output (Claude Code) | Compiled Output (Generic) |
|-----------|-----------------|-------------------------------|---------------------------|
| **Agents** | `~/agent-os/profiles/default/agents/*.md` | `.claude/agents/agent-os/*.md` | N/A |
| **Commands** | `~/agent-os/profiles/default/commands/*/` | `.claude/commands/agent-os/*/` | `agent-os/commands/*.md` |
| **Workflows** | `~/agent-os/profiles/default/workflows/*/` | Embedded in commands/agents | Embedded in commands |
| **Standards** | `~/agent-os/profiles/default/standards/*/` | `.claude/skills/agent-os/*.md` (if Skills enabled) or embedded | `agent-os/standards/*/` or embedded |

**Key Insight**: Prompts are **templates** in `~/agent-os/profiles/` that get **compiled and installed** into your project's `.claude/` or `agent-os/` directories with all references expanded and conditionals evaluated.
