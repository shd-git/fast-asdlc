# Skill: Tool Onboarding & Provisioning - Kimi Code CLI

## 1. Objective
- **Purpose:** Act as a provisioning adapter that maps the Fast-ASDLC agent workforce (rules, workflows, memory-bank) into Kimi Code CLI's native architecture (`AGENTS.md`, `SKILL.md`, and optional custom agent YAML files).
- **Context:** Invoked by the Meta-Agent immediately after fine-tuning framework manifests or when the human explicitly selects Kimi Code CLI as their primary AI IDE.

## 2. Kimi Code CLI Architecture Overview

Unlike Cursor (`.cursorrules`) or Cline (`.clinerules`), Kimi Code CLI uses a **three-layer context system**:

| Fast-ASDLC Asset | Kimi Code Native Equivalent | Scope |
|---|---|---|
| `/.agents/rules/[role].md` | `.kimi/skills/fast-asdlc-[role]/SKILL.md` | Project-level skill |
| `/.agents/fast-asdlc/METHODOLOGY.md` | `.kimi/AGENTS.md` (or root `AGENTS.md`) | Project context |
| `/.agents/memory-bank/` | Referenced via relative paths inside `AGENTS.md` | Persistent state |
| `/.agents/workflows/` | Embedded into skills or referenced in `AGENTS.md` | Execution sequences |
| Custom agent orchestration | Optional: `.kimi/agents/[role].yaml` | Agent identity & tools |

### Key Differences from Cursor/Cline
- **No dotfile rules:** Kimi Code does not read `.cursorrules` or `.clinerules`. It reads `AGENTS.md` (project context) and discovers `SKILL.md` files.
- **Skills are first-class:** The AI decides on its own whether to read a `SKILL.md` based on the task. Skills must have clear `description` frontmatter.
- **Subagents:** Kimi Code has built-in subagent types (`coder`, `explore`, `plan`). Fast-ASDLC roles can map to these or to custom agent YAML files.
- **Built-in variables:** System prompts can reference `${KIMI_WORK_DIR}`, `${KIMI_AGENTS_MD}`, `${KIMI_SKILLS}`, `${KIMI_NOW}`.

## 3. Direction of Truth & Write Boundaries
- **Immutable Rule:** `/.agents/` remains the absolute Source of Truth (SSOT).
- **Target Runtime Environment:** `.kimi/` directory at the project root.
- **Execution Constraint:** This skill performs content adaptation (not raw copy). It MUST restructure Fast-ASDLC markdown into Kimi Code-native `SKILL.md` format with YAML frontmatter.

## 4. Provisioning Execution Steps

When synchronization is triggered for Kimi Code CLI, execute these atomic operations:

### Phase 1: Assert Environment
1. **Initialize `.kimi/` structure:**
   ```
   .kimi/
   ├── AGENTS.md                 # Merged project context
   ├── skills/
   │   ├── fast-asdlc-architect/
   │   │   └── SKILL.md
   │   ├── fast-asdlc-programmer/
   │   │   └── SKILL.md
   │   ├── fast-asdlc-qa/
   │   │   └── SKILL.md
   │   └── fast-asdlc-meta/
   │       └── SKILL.md
   └── agents/                   # Optional: custom agent YAMLs
       ├── architect.yaml
       ├── programmer.yaml
       └── qa.yaml
   ```
2. **Verify `.kimi/` is gitignored or tracked** based on team policy. (Recommended: track `AGENTS.md` and `skills/*/SKILL.md` in Git; ignore runtime agent configs if they contain local paths.)

### Phase 2: Generate `.kimi/AGENTS.md`
3. **Compile Project Context:** Read `/.agents/fast-asdlc/METHODOLOGY.md`, `/.agents/memory-bank/project-brief.md`, and `/.agents/memory-bank/process-overview.md`. Generate `.kimi/AGENTS.md` with:
   - High-level Fast-ASDLC methodology summary
   - Active project brief (sanitized: no secrets)
   - Agent interaction standards (who calls whom)
   - Language policy boundaries
   - Absolute paths to memory-bank and backlog for `${KIMI_WORK_DIR}` resolution

   **Template:**
   ```markdown
   # Fast-ASDLC Project Context

   This project follows the Fast-ASDLC agentic SDLC framework.

   ## Methodology
   - AI-Native & Agentic: 80-90% automation, human-in-the-loop
   - Spec-Driven Development: No code without verified specs in `/docs/specs/`
   - Hexagonal Architecture: Domain → Application → Infrastructure
   - Everything-as-Code: All artifacts in Markdown/Mermaid

   ## Agent Workforce
   Available skills (invoke via `/skill:<name>` or automatic context):
   - `fast-asdlc-meta` — Process architect, workforce initialization
   - `fast-asdlc-architect` — C4 modeling, ADRs, technical specs
   - `fast-asdlc-programmer` — Hexagonal code generation, test scaffolding
   - `fast-asdlc-qa` — Test suite generation, dynamic validation

   ## Memory Bank
   - Project brief: `/.agents/memory-bank/project-brief.md`
   - Process overview: `/.agents/memory-bank/process-overview.md`
   - Active context: `/.agents/memory-bank/active-context.md`
   - Decision log: `/.agents/memory-bank/decision-log.md`
   - Progress tracker: `/.agents/memory-bank/progress.md`

   ## Language Policy
   [Dynamically populated from project-brief.md]
   ```

### Phase 3: Adapt Rules into Skills
4. **Per-Role Skill Generation:** For each file in `/.agents/rules/[role].md`, generate `.kimi/skills/fast-asdlc-[role]/SKILL.md` with YAML frontmatter:

   ```markdown
   ---
   name: fast-asdlc-[role]
   description: Fast-ASDLC [Role] Agent — [brief description from rule file]
   ---

   ## Role: [Role Name]

   [Adapted content from /agents/rules/[role].md, preserving all constraints]

   ## Relevant Workflows
   - [Link to /.agents/workflows/[role]-*.md]

   ## Memory Bank Integration
   - Read `/.agents/memory-bank/project-brief.md` for context
   - Read `/.agents/memory-bank/active-context.md` for current focus
   ```

5. **Content Adaptation Rules:**
   - Strip Cursor-specific path references (`/.cursor/rules/`, `.mdc` frontmatter)
   - Strip Cline-specific path references (`/.clinerules/`)
   - Replace with Kimi Code-native relative paths (`/.agents/...`)
   - Preserve all behavioral constraints, quality gates, and forbidden actions
   - Keep `MUST` / `MUST NOT` language intact

### Phase 4: Optional Custom Agent YAMLs (Advanced)
6. **Generate Agent Definitions:** If the human requests strict tool boundaries per role, create `.kimi/agents/[role].yaml`:

   ```yaml
   version: 1
   agent:
     name: fast-asdlc-[role]
     extend: default
     system_prompt_path: ../skills/fast-asdlc-[role]/SKILL.md
     # Tool restrictions mapped from role write boundaries:
     exclude_tools:
       - "kimi_cli.tools.file:WriteFile"        # If role is read-only (e.g., QA reviewing)
       - "kimi_cli.tools.file:StrReplaceFile"   # If role is read-only
   ```

   **Note:** Kimi Code's custom agent YAMLs are loaded via `--agent-file`, which replaces the default agent entirely. For Fast-ASDLC's multi-agent workflow, it is usually **preferable to use skills** (loaded automatically) and let the human or Meta-Agent decide which skill to invoke, rather than switching agent files.

### Phase 5: Validation & Sync Lock
7. **Verify Skill Discovery:** Ensure all generated skills appear in `${KIMI_SKILLS}` list when Kimi Code starts.
8. **Sync Lock:** Any deletion or renaming of an agent manifest inside `/.agents/` must trigger an immediate mirror deletion in `.kimi/skills/` to eliminate zombie skills.
9. **Fail-Fast Trigger:** If `.kimi/` is inside a directory where the file system blocks writes (e.g., read-only mount), abort the bootstrap sequence and alert the human supervisor.

## 5. Operational Guardrails
- **No Git Side-Effects:** NEVER execute `git commit` or `git push` autonomously during provisioning.
- **Path Sanitization:** All paths in generated skills must be relative to the project root. No absolute local paths.
- **SSOT Preservation:** `.kimi/` is a derived view. Humans may edit `/.agents/` directly; re-running this skill must overwrite `.kimi/` to match.

## 6. Human Onboarding Quickstart

After provisioning completes, instruct the human:

1. **Launch Kimi Code CLI** in the project root.
2. **Verify context loading:** The welcome panel should show the Fast-ASDLC skills under the Project scope.
3. **Invoke a skill manually:**
   ```
   /skill:fast-asdlc-meta
   ```
   This loads the Meta-Agent skill and initiates workforce initialization.
4. **Automatic invocation:** During normal work, Kimi Code will automatically read relevant skills based on the task context (e.g., when discussing architecture, it will load `fast-asdlc-architect`).

## 7. Mapping Reference

| Fast-ASDLC Concept | Kimi Code CLI Implementation |
|---|---|
| Meta-Agent bootstrap | `fast-asdlc-meta` skill + `.kimi/AGENTS.md` |
| Slash command (`/architect`) | `/skill:fast-asdlc-architect` or automatic context |
| `.cursorrules` role envelope | `SKILL.md` YAML frontmatter + `AGENTS.md` context |
| `.clinerules/workflows/` | Embedded workflow links inside `SKILL.md` |
| MCP settings | `~/.kimi/config.toml` `[mcp.client]` section |
| Plan Mode | Native `EnterPlanMode` / `ExitPlanMode` tools |
| Subagent orchestration | Built-in `coder`/`explore`/`plan` subagents or custom YAML |
