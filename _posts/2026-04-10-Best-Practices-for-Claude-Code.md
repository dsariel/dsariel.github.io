---
layout: post
title: Best Practices for Claude Code
date: 2026-04-10 09:50:00
description: Based on https://code.claude.com/docs/en/best-practices and https://habr.com/ru/articles/1012412/
tags: claude architecture skills context-management MCP plugins hooks subagents best-practices workflow-design prompt-engineering system-design tool-integration context-optimization ai-engineering
categories: Claude Code
---

# Best Practices for Claude Code


## Table of Contents

1. [Architecture Layers Overview](#architecture-layers-overview)
2. [System Balance and Layering](#system-balance-and-layering)
3. [Concept Boundaries](#concept-boundaries)
4. [Writing an Effective CLAUDE.md](#writing-an-effective-claudemd)
5. [Recommended Context Layering](#recommended-context-layering)
6. [Best Practices for Context Management](#best-practices-for-context-management)
7. [Connecting MCP Servers](#connecting-mcp-servers)
8. [Setting Up Hooks](#setting-up-hooks)
9. [Designing Skills](#designing-skills)
10. [Creating Custom Subagents](#creating-custom-subagents)
11. [Installing Plugins](#installing-plugins)
12. [Key Takeaways](#key-takeaways)


---

## Architecture Layers Overview

### The Six-Layer Framework

| Layer | Responsibility |
|-------|-----------------|
| **CLAUDE.md / rules / memory** | Long-term context: what Claude "knows" |
| **Tools / MCP** | Capabilities of action: what Claude "can do" |
| **Skills** | Methods on request: how Claude "should act" |
| **Hooks** | Obligatory behavior, without Claude's judgment |
| **Subagents** | Isolated workers, managed autonomy |
| **Verifiers** | Closed loop: we check, we respond, we audit |

---

## System Balance and Layering

### The Problem with Unbalanced Layers

If you strengthen only one layer, the system becomes unbalanced. CLAUDE.md is too long – the context pollutes itself; too many tools – choice becomes unclear; subagents launched everywhere – state begins to drift; verification skipped – when an error occurs, it's completely unclear where it broke.

---

## Concept Boundaries

### MCP / Plugin / Tools / Skills / Hooks / Subagents

| Concept | Role in Framework | What It Solves | Typical Misuse |
|---------|------------------|-----------------|-----------------|
| **CLAUDE.md** | Persistent project contract | Commands, boundaries, prohibitions for each session | Turn into command wiki |
| **.claude/rules/** | Path/language rules | Directories, file types, local norms | All rules fall into root CLAUDE.md |
| **Built-in Tools** | Built-in capabilities | Files, commands, search | All integrations drain to shell |
| **MCP** | External access protocol | GitHub, Sentry, Database | Too many servers, context overloaded |
| **Plugin** | Packet distribution | Skills/Hooks/MCP together | Plugin as runtime-primitive |
| **Skill** | Knowledge/workflow by request | Gives Claude a "method of work" | Skill as encyclopedia + deployed script |
| **Hook** | Rule override level | Check before/after events | Hook instead of all model decisions |
| **Subagent** | Isolated unit of work | Parallel research, scoped instruments | Unbounded fan-out, chaos management |

### Key Distinctions

- **New capabilities for Claude** – Tool/MCP
- **A set of working methods** – Skill
- **Isolated execution environment** – Subagent
- **Forced constraints and audit** – Hook
- **Distribution across projects** – Plugin

---

## Writing an Effective CLAUDE.md

### Overview

Run `/init` to generate a starter CLAUDE.md file based on your current project structure, then refine over time. CLAUDE.md is a special file that Claude reads at the start of every conversation. Include Bash commands, code style, and workflow rules. This gives Claude persistent context it can't infer from code alone.

### Keep It Short and Focused

Keep it concise. For each line, ask: "Would removing this cause Claude to make mistakes?" If not, cut it. Bloated CLAUDE.md files cause Claude to ignore your actual instructions!

**Target size:** Anthropic's official CLAUDE.md is approximately 2.5K tokens—use that as a reference point.

### What to Include vs. Exclude

| ✅ Include | ❌ Exclude |
|-----------|-----------|
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Code style rules that differ from defaults | Standard language conventions Claude already knows |
| Testing instructions and preferred test runners | Detailed API documentation (link to docs instead) |
| Repository etiquette (branch naming, PR conventions) | Information that changes frequently |
| Architectural decisions specific to your project | Long explanations or tutorials |
| Developer environment quirks (required env vars) | File-by-file descriptions of the codebase |
| Common gotchas or non-obvious behaviors | Self-evident practices like "write clean code" |

### Example CLAUDE.md

```markdown
# Code style
- Use ES modules (import/export) syntax, not CommonJS (require)
- Destructure imports when possible (eg. import { foo } from 'bar')

# Workflow
- Be sure to typecheck when you're done making a series of code changes
- Prefer running single tests, and not the whole test suite, for performance
```

### File Locations

You can place CLAUDE.md files in several locations:

- **Home folder (`~/.claude/CLAUDE.md`)**: applies to all Claude sessions
- **Project root (`./CLAUDE.md`)**: check into git to share with your team
- **Project root (`./CLAUDE.local.md`)**: personal project-specific notes; add this file to your `.gitignore` so it isn't shared with your team
- **Parent directories**: useful for monorepos where both `root/CLAUDE.md` and `root/foo/CLAUDE.md` are pulled in automatically
- **Child directories**: Claude pulls in child CLAUDE.md files on demand when working with files in those directories

### Importing Additional Files

CLAUDE.md files can import additional files using `@path/to/import` syntax:

```markdown
See @README.md for project overview and @package.json for available npm commands.

# Additional Instructions
- Git workflow: @docs/git-instructions.md
- Personal overrides: @~/.claude/my-project-instructions.md
```

---

## Recommended Context Layering

### The Layering Strategy

```
Always resident      → CLAUDE.md: project contract / build team / prohibitions
On load path         → rules: rules for languages / directories / file types
On demand            → Skills: workflows / domain knowledge
Isolated loading     → Subagents: mass research / parallel investigations
Outside context      → Hooks: deterministic scripts / audit / blocking
```

### Core Principle

In other words, what is used rarely should not be loaded every time.

---

## Best Practices for Context Management

### Essential Guidelines

1. **Keep CLAUDE.md short, strict, and executable** – Prioritize commands, constraints, architectural boundaries

2. **Move large reference documents to supporting files for Skills** – Don't stuff them into the body of SKILL.md

3. **Use `.claude/rules/` for path/language rules** – Don't overload the root CLAUDE.md

4. **In long sessions, actively monitor context usage** – Use `/context` command to track token consumption

### Context Practices

- **When switching tasks:** Use `/clear` to reset context between unrelated work
- **When moving to a new stage of the same task:** Use `/compact` to condense history while preserving key information
- **Write Compact Instructions in CLAUDE.md** – Control what survives compression, not the algorithm

### Managing Context Aggressively

During long sessions, Claude's context window can fill with irrelevant conversation, file contents, and commands. This can reduce performance and sometimes distract Claude.

- Use `/clear` frequently between tasks to reset the context window entirely
- When auto compaction triggers, Claude summarizes what matters most, including code patterns, file states, and key decisions
- For more control, run `/compact <instructions>`, like `/compact Focus on the API changes`
- To compact only part of the conversation, use `Esc + Esc` or `/rewind`, select a message checkpoint, and choose **Summarize from here**
- Customize compaction behavior in CLAUDE.md with instructions like `"When compacting, always preserve the full list of modified files and any test commands"` to ensure critical context survives summarization

---

## Connecting MCP Servers

### What Are MCP Servers?

With MCP servers, you can ask Claude to implement features from issue trackers, query databases, analyze monitoring data, integrate designs from Figma, and automate workflows.

### How to Connect

Run `claude mcp add` to connect external tools like Notion, Figma, or your database.

This enables Claude to interact with external services like:
- GitHub
- Sentry
- Databases
- Figma designs
- Issue trackers
- Monitoring systems

### Context Efficiency

CLI tools are the most context-efficient way to interact with external services. Use them whenever possible.

---

## Setting Up Hooks

### What Are Hooks?

Hooks run scripts automatically at specific points in Claude's workflow. Unlike CLAUDE.md instructions which are advisory, hooks are deterministic and guarantee the action happens.

### When to Use Hooks

Use hooks for actions that **must happen every time with zero exceptions**. They're appropriate for:
- Automated validation (e.g., linting)
- Blocking certain operations (e.g., preventing writes to migrations folder)
- Required side-effect actions that cannot be missed

### Example Hook Scenarios

Claude can write hooks for you. Try prompts like *"Write a hook that runs eslint after every file edit"* or *"Write a hook that blocks writes to the migrations folder."* Edit `.claude/settings.json` directly to configure hooks by hand, and run `/hooks` to browse what's configured.

### Hooks vs. CLAUDE.md Instructions

- **Hooks** = Mandatory, deterministic execution
- **CLAUDE.md** = Advisory guidelines that Claude may overlook if context is crowded

---

## Designing Skills

### Official Definition

A **Skill** is "knowledge and workflows loaded on request." The descriptor stays in context permanently, while full content loads on demand – this is very different from a "saved prompt."

### Why Skills Matter

CLAUDE.md is loaded every session, so only include things that apply broadly. For domain knowledge or workflows that are only relevant sometimes, use skills instead. Claude loads them on demand without bloating every conversation.

### What a Good Skill Should Meet

1. **Description tells the model "when to use me," not "what I do"** – The difference is huge
2. **Complete steps, inputs, outputs, and stopping conditions** – Not just a beginning without an end
3. **Main text only – navigation and key constraints** – Move voluminous materials to supporting files
4. **Side effects require explicit safeguards** – Use `disable-model-invocation: true` for workflows you want to trigger manually

### Skill Structure

Create a skill by adding a directory with a `SKILL.md` to `.claude/skills/`:

```
.claude/skills/
└── incident-triage/
    ├── SKILL.md
    ├── runbook.md
    ├── examples.md
    └── scripts/
        └── collect-context.sh
```

### Progressive Disclosure Pattern

The Claude Code team repeatedly emphasized the concept of "progressive disclosure" in their internal design – the idea is not for the model to see all information at once, but to first get an index and navigation, then pull in details as needed:

- SKILL.md defines task semantics, boundaries, and execution skeleton
- Supporting files provide domain details
- Scripts deterministically collect context or evidence

### Creating Skills

Skills extend Claude's knowledge with information specific to your project, team, or domain. Claude applies them automatically when relevant, or you can invoke them directly with `/skill-name`.

Example skill definition:

```yaml
---
name: api-conventions
description: REST API design conventions for our services
---
# API Conventions
- Use kebab-case for URL paths
- Use camelCase for JSON properties
- Always include pagination for list endpoints
- Version APIs in the URL path (/v1/, /v2/)
```

### Three Typical Skill Types

#### Type 1: Checklist (Quality Barrier)

A pre-release run to make sure nothing is missed:

```yaml
---
name: release-check
description: Use before cutting a release to verify build, version, and smoke test.
---

## Pre-flight (All must pass)
- `cargo build --release` passes
- `cargo clippy -D warnings` clean
- Version bumped in Cargo.toml
- CHANGELOG updated
- `kaku doctor` passes on clean env

## Output
Pass / Fail per item. Any Fail must be fixed before release.
```

#### Type 2: Workflow (Standardized Operation)

Config migration – high risk, explicit invocation + built-in rollback steps:

```yaml
---
name: config-migration
description: Migrate config schema. Run only when explicitly requested.
disable-model-invocation: true
---

## Steps
1. Backup: `cp ~/.config/kaku/config.toml ~/.config/kaku/config.toml.bak`
2. Dry run: `kaku config migrate --dry-run`
3. Apply: remove `--dry-run` after confirming output
4. Verify: `kaku doctor` all pass

## Rollback
`cp ~/.config/kaku/config.toml.bak ~/.config/kaku/config.toml`
```

#### Type 3: Domain Expert (Encapsulated Decision-Making)

When there are runtime issues, Claude collects evidence along a fixed path:

```yaml
---
name: runtime-diagnosis
description: Use when kaku crashes, hangs, or behaves unexpectedly at runtime.
---

## Evidence Collection
1. Run `kaku doctor` and capture full output
2. Last 50 lines of `~/.local/share/kaku/logs/`
3. Plugin state: `kaku --list-plugins`

## Decision Matrix
| Symptom             | First Check                         |
|---------------------|-------------------------------------|
| Crash on startup    | doctor output → Lua syntax error    |
| Rendering glitch    | GPU backend / terminal capability   |
| Config not applied  | Config path + schema version        |

## Output Format
Root cause / Blast radius / Fix steps / Verification command
```

### Descriptor Optimization

Keep descriptors short – each Skill eats up context space.

**Inefficient (~45 tokens):**
```
description: >
  This skill helps you review code changes in Rust projects.
  It checks for common issues like unsafe code, error handling...
  Use this when you want to ensure code quality before merging.
```

**Efficient (~9 tokens):**
```
description: Use for PR reviews with focus on correctness.
```

### Auto-Invocation Strategy

| Frequency | Strategy | Action |
|-----------|----------|--------|
| High (>1 per session) | Keep auto-invoke | Optimize descriptor |
| Low (<1 per session) | Disable auto-invoke | Manual invocation, exclude from context |
| Very low (<1 per month) | Delete Skill | Move to AGENTS.md documentation |

### Skills Antipatterns to Avoid

- **Description too short:** `description: help with backend` (triggers on any backend work)
- **Main text too long:** Hundreds of lines crammed into SKILL.md
- **One Skill, too many tasks:** Review + deploy + debug + docs + incident = chaos
- **Side effects without safeguards:** Allows automatic invocation by the model

---

## Creating Custom Subagents

### What Are Subagents?

Subagents run in their own context with their own set of allowed tools. They're useful for tasks that read many files or need specialized focus without cluttering your main conversation.

### Why Use Subagents?

Since context is your fundamental constraint, subagents are one of the most powerful tools available. When Claude researches a codebase it reads lots of files, all of which consume your context. Subagents run in separate context windows and report back summaries.

### Creating a Subagent

Define specialized assistants in `.claude/agents/` that Claude can delegate to for isolated tasks.

Example:

```yaml
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
model: opus
---
You are a senior security engineer. Review code for:
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication and authorization flaws
- Secrets or credentials in code
- Insecure data handling

Provide specific line references and suggested fixes.
```

### Invoking Subagents

Tell Claude to use subagents explicitly: *"Use a subagent to review this code for security issues."*

### Investigation Pattern

```
Use subagents to investigate how our authentication system handles token
refresh, and whether we have any existing OAuth utilities I should reuse.
```

The subagent explores the codebase, reads relevant files, and reports back with findings, all without cluttering your main conversation.

---

## Installing Plugins

### What Are Plugins?

Plugins bundle skills, hooks, subagents, and MCP servers into a single installable unit from the community and Anthropic. If you work with a typed language, install a code intelligence plugin to give Claude precise symbol navigation and automatic error detection after edits.

### How to Install

Run `/plugin` to browse the marketplace. Plugins add skills, tools, and integrations without configuration.

### Choosing Between Features

For guidance on choosing between skills, subagents, hooks, and MCP, see the Extend Claude Code documentation.

---

## Key Takeaways

### System Design

1. **Balance is critical** – Strengthen all layers proportionally, not just one
2. **Context is precious** – Load what's needed, when it's needed
3. **Layering strategy matters** – Always resident → on load path → on demand → isolated → outside context

### CLAUDE.md Guidelines

1. **Keep it short** (~2.5K tokens target)
2. **Make it executable** – Commands, constraints, boundaries
3. **Prune aggressively** – If Claude can figure it out, don't document it
4. **Check it into git** – Share with your team, it compounds in value

### Skills Design

1. **Skills are workflows, not documentation** – They enable doing, not just knowing
2. **Descriptors control invocation** – Make them tell the story of "when to use me"
3. **Progressive disclosure** – Index first, details on demand
4. **Side effects need safeguards** – Use `disable-model-invocation: true` for risky operations

### Context Management

1. **Use `/clear` between unrelated tasks** – Reset context to prevent pollution
2. **Use `/compact` for progress within a task** – Condense history while preserving decisions
3. **Monitor token usage** – Use `/context` command actively
4. **Delegate with subagents** – Keep main conversation clean for implementation

### Hook Strategy

1. **Hooks = Deterministic execution** – Use for actions that must happen every time
2. **CLAUDE.md = Advisory** – Use for guidelines Claude should follow
3. **Think fail-safe** – Hooks should block or enforce, not suggest

### MCP Integration

1. **Context-efficient** – Use CLI tools and MCP servers instead of APIs
2. **Flexible** – Connect databases, issue trackers, design tools, monitoring systems
3. **Scalable** – Start with one tool, add more as needed

---

## Summary: The Complete Picture

A well-designed Claude system has:

- **A lean CLAUDE.md** (~2.5K tokens) with essential commands and constraints
- **Clear rules in `.claude/rules/`** for language/directory-specific guidance
- **Domain skills in `.claude/skills/`** loaded on demand for specialized knowledge
- **Deterministic hooks** for actions that must happen without fail
- **Specialized subagents** for isolated research or verification
- **Connected MCP servers** for external tool integration
- **Plugins** bundling related functionality together

This architecture ensures that Claude has the information it needs, when it needs it, without context pollution or missed opportunities.

---

*This guide synthesizes best practices from [Anthropic's Claude Code](https://code.claude.com/docs/en/best-practices) documentation and [akimovpro's](https://habr.com/ru/users/akimovpro/) [article](https://habr.com/ru/articles/1012412/) on proven patterns for building scalable, maintainable Claude systems with sustainable context management wich is based on [tw93's](https://github.com/tw93) original [article](https://x.com/HiTw93/status/2032091246588518683).*

