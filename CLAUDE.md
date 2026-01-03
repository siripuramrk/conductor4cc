# Conductor Extension - AI Agent Context

## Project Overview

**Conductor** is a Gemini CLI extension that enables **Context-Driven Development**. It transforms the Gemini CLI into a proactive project manager that follows a strict protocol: **Context -> Spec & Plan -> Implement**.

- **Philosophy**: "Measure twice, code once" - treating context as a managed artifact alongside code
- **Version**: 0.1.1
- **License**: Apache License 2.0
- **Repository**: https://github.com/gemini-cli-extensions/conductor

---

## Architecture

### Declarative Command System

This project uses a **TOML-based declarative command system** for the Gemini CLI. Commands are defined as `.toml` files rather than imperative code:

```
commands/conductor/
├── setup.toml       # Project initialization (38KB)
├── newTrack.toml    # Track creation (9.6KB)
├── implement.toml   # Implementation execution (12.7KB)
├── revert.toml      # Git-aware revert (8.6KB)
└── status.toml      # Progress display (3.6KB)
```

### Command TOML Structure

Each command file contains:

```toml
description = "Human-readable command description"

prompt = """
## Multi-line prompt that defines:
## 1.0 SYSTEM DIRECTIVE - Agent role and purpose
## 2.0 PROTOCOL - Step-by-step execution instructions
"""
```

The `prompt` field contains detailed instructions for the AI agent, including:
- Setup validation checks
- Protocol steps with numbered sequences
- Critical directives (marked with **CRITICAL:**)
- Error handling instructions

---

## Directory Structure

```
conductor4cc/
├── commands/conductor/           # Command definitions (TOML)
│   ├── setup.toml
│   ├── newTrack.toml
│   ├── implement.toml
│   ├── revert.toml
│   └── status.toml
├── templates/                    # Templates for generated artifacts
│   ├── code_styleguides/        # Language-specific style guides
│   │   ├── general.md
│   │   ├── python.md
│   │   ├── javascript.md
│   │   ├── typescript.md
│   │   ├── go.md
│   │   ├── dart.md
│   │   ├── csharp.md
│   │   └── html-css.md
│   └── workflow.md              # TDD-based development workflow
├── .github/workflows/
│   └── release-please.yml       # Automated release management
├── gemini-extension.json         # Extension metadata
├── GEMINI.md                     # Extension context file
├── README.md                     # User documentation
└── CONTRIBUTING.md               # Contribution guidelines
```

---

## Core Concepts

### Tracks

A **track** is the highest-level unit of work in Conductor, representing either a feature or bug fix. Each track contains:

- `spec.md` - Detailed requirements and acceptance criteria
- `plan.md` - Actionable to-do list with phases, tasks, and sub-tasks
- `metadata.json` - Track metadata (ID, type, status, timestamps)

### Workflow Protocol

The default workflow (`templates/workflow.md`) enforces **Test-Driven Development**:

1. **Red Phase**: Write failing tests
2. **Green Phase**: Implement minimum code to pass tests
3. **Refactor Phase**: Improve code quality with passing tests as safety net
4. **Verification**: Manual verification at phase completion with checkpoint commits

### Task Status Markers

In `plan.md` files:
- `[ ]` - Pending
- `[~]` - In Progress
- `[x]` - Completed (with commit SHA appended)

### Git Integration

- **Task commits**: Individual task implementations
- **Git notes**: Attached to commits with detailed summaries
- **Checkpoint commits**: Created at phase completion with verification reports
- **SHA tracking**: Commit hashes recorded in `plan.md` for traceability

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `gemini-extension.json` | Extension metadata (name, version, contextFileName) |
| `GEMINI.md` | Context file loaded when extension is active |
| `commands/conductor/*.toml` | Command definitions with prompts |
| `templates/workflow.md` | Default development workflow template |
| `templates/code_styleguides/*.md` | Language-specific coding standards |

---

## Generated Artifacts (User Projects)

When users run Conductor commands, the following directory structure is created:

```
conductor/
├── product.md              # Project context (users, goals, features)
├── product-guidelines.md   # Brand/UX standards
├── tech-stack.md           # Technical choices and constraints
├── workflow.md             # Team development workflow
├── tracks.md               # Track registry and overview
├── code_styleguides/       # Copy of style guides for user's language(s)
└── tracks/
    └── <track_id>/
        ├── spec.md         # Track specification
        ├── plan.md         # Implementation plan
        └── metadata.json   # Track metadata
```

---

## Command Flow

### 1. `/conductor:setup`
- Validates project state (greenfield vs brownfield)
- Generates `conductor/` directory with context files
- Prompts for product context, tech stack, and workflow preferences
- Creates `tracks.md` registry

### 2. `/conductor:newTrack`
- Creates new track directory with unique ID
- Generates `spec.md` with requirements
- Creates `plan.md` with phased task breakdown
- Updates `tracks.md` registry

### 3. `/conductor:implement`
- Reads current track's `plan.md`
- Executes tasks sequentially following workflow
- Updates task status markers
- Creates commits with git notes
- Creates checkpoint commits at phase completion
- Guides user through manual verification

### 4. `/conductor:status`
- Reads `tracks.md` and all track `plan.md` files
- Displays progress summary with current phase/task
- Shows blockers and next actions

### 5. `/conductor:revert`
- Analyzes git history to identify logical work units
- Reverts by track, phase, or task
- Uses commit SHAs and git notes for precise rollback

---

## Build and Release

### No Build System

This is a **configuration-only extension** with no compilation step. The TOML files are interpreted directly by Gemini CLI.

### Release Automation

Uses **release-please** for automated releases:

- **Trigger**: Push to `main` branch
- **Detection**: Conventional commit messages trigger version bumps
- **Artifacts**: TAR archives uploaded to GitHub Releases
- **Configuration**: `.release-please-manifest.json` and `release-please-config.json`

### Conventional Commit Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting changes
- `refactor`: Code restructuring
- `test`: Adding tests
- ` chore`: Maintenance tasks

---

## Development Patterns

### Error Handling

Commands include setup validation:
```toml
## 1.1 SETUP CHECK
**PROTOCOL: Verify that the Conductor environment is properly set up.**

1. **Check for Required Files:** You MUST verify the existence of:
   - `conductor/tech-stack.md`
   - `conductor/workflow.md`
   - `conductor/product.md`

2. **Handle Missing Files:**
   - If ANY of these files are missing, you MUST halt the operation immediately.
   - Announce: "Conductor is not set up. Please run `/conductor:setup`"
```

### Protocol Sections

Each command follows a standard structure:
- `## 1.0 SYSTEM DIRECTIVE` - Agent role and purpose
- `## 1.1 SETUP CHECK` - Environment validation
- `## 2.0 PROTOCOL` - Main execution steps
- Subsections for each logical phase

### Critical Directives

Instructions marked with `**CRITICAL:**` must be followed exactly:
- Tool call validation
- File existence checks
- Error handling requirements

---

## Quality Gates

From the workflow template, before marking a task complete:

- [ ] All tests pass
- [ ] Code coverage >80%
- [ ] Code follows style guide
- [ ] All public functions documented
- [ ] Type safety enforced
- [ ] No linting errors
- [ ] Mobile testing complete (if applicable)
- [ ] No security vulnerabilities

---

## Important Notes for Agents

1. **Plan is Source of Truth**: All work must be tracked in `plan.md`
2. **Validate Every Tool Call**: Check tool call success before proceeding
3. **Follow Strict Workflow**: Red-Green-Refactor TDD cycle
4. **Commit Per Task**: Each task gets its own commit with git note
5. **Checkpoint at Phase End**: Create verification checkpoint after each phase
6. **Update Status Markers**: Always update `[ ]` -> `[~]` -> `[x]` in plan.md
7. **Read Context Files**: Always reference `product.md`, `tech-stack.md`, `workflow.md`

---

## Token Consumption Consideration

Conductor's context-driven approach reads multiple files (context, specs, plans) which can increase token usage. Users can check consumption with `/stats model` in their Gemini CLI session.

---

## Extension Metadata

```json
{
  "name": "conductor",
  "version": "0.1.1",
  "contextFileName": "GEMINI.md"
}
```

The `GEMINI.md` file is automatically loaded when the extension is active, providing context about the extension itself.

---

## Related Resources

- [Gemini CLI Extensions Documentation](https://geminicli.com/docs/extensions/)
- [GitHub Issues](https://github.com/gemini-cli-extensions/conductor/issues)
- [Contributing Guidelines](CONTRIBUTING.md)
