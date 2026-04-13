# Classification Guide

Heuristics for classifying CLAUDE.md sections as invariant, workflow, or router.

## Section 1: The Three Types

### Invariant

Universal constraints that apply to every Claude Code interaction in this project.

**Criteria:**
- Applies to every Claude Code interaction in this project, not just specific tasks
- Would be confusing to omit from any session (style rules, security constraints, identity)
- Short: usually 1-5 lines per rule

**Examples:**
- "Always use TypeScript strict mode"
- "Never commit .env files"
- "My name is Luke, I'm a solo developer"
- "Prefer composition over inheritance"

### Workflow

A named, repeatable, multi-step procedure for a specific task type.

**Criteria:**
- Describes a named, repeatable, multi-step procedure for a specific task type
- Has a natural trigger ("when building a component," "when writing tests," "when debugging")
- Contains at least 3 steps in sequence
- Could be invoked as a slash command without changing any behavior

**Examples:**
- "When creating a new API route, do 1... 2... 3..."
- "When writing a PR description, include..."
- "When debugging, first check logs, then..."

### Router

Decision logic that controls which behavior, skill, or subagent to invoke.

**Criteria:**
- Decision logic: "if X, do Y"
- Controls which behavior, skill, or subagent to invoke
- Often appears as conditional statements or escalation rules
- The `UserPromptSubmit` hook is the right home for routing logic

**Examples:**
- "If the user asks to deploy, run tests first"
- "For ML tasks, use the Python subagent"
- "Before starting any task, review available skills"

---

## Section 2: Gray-Area Heuristics

- **Long role descriptions (>10 lines explaining Claude's persona)** → split: the core role definition is invariant, but specific behavioral instructions within the role description may be workflows if they're task-specific

- **Lists of "things to always do"** → invariant if truly universal (≤5 lines per item); workflow if the list is actually a procedure in disguise

- **Tool-use instructions** → invariant if they're constraints ("never use grep, use Grep"), workflow if they're procedures ("to search the codebase, first run Glob, then...")

- **"Key principles" sections** → invariant if abstract, workflow if procedural

---

## Section 3: Token Savings Estimation

**Level 1 (metadata):** ~100 tokens — always in context regardless

**Level 2 (SKILL.md body):** ~1000-5000 tokens — loaded only when skill invoked

**Section extracted to skill:** those tokens never appear in the main session context

### Savings Formula

```
(lines_extracted / total_lines) × 0.85
```

The 0.85 factor accounts for the fact that metadata and references remain in context.

### Rule of Thumb

A 500-line CLAUDE.md with 300 extractable lines → ~50% token reduction in sessions that don't invoke those skills.

---

## Section 4: Examples

Each example below is synthetic, not from any actual user config.

### Example A — Invariant

```markdown
## Core Principles
- Simplicity first. Impact minimal code.
- No laziness. Find root causes.
- Never commit to main without tests passing.
```

**Classification:** invariant

**Reason:** Three lines, each a universal constraint, no task specificity.

---

### Example B — Workflow

```markdown
## Writing a New Feature
1. Read the existing code in the relevant module.
2. Write the test first.
3. Implement the minimum code to pass the test.
4. Run the full test suite.
5. Update CHANGELOG.md.
6. Open a PR with the template from .github/pull_request_template.md.
```

**Classification:** workflow

**Reason:** Six ordered steps, specific to one task type ("writing a new feature"), natural trigger, suitable as `/new-feature` skill.

---

### Example C — Router

```markdown
## Task Routing
For any task involving database migrations: pause and ask for confirmation
before writing any migration files. For tasks involving external API calls:
check for existing MCP servers first. For documentation tasks: use the
portfolio-readme-writer skill.
```

**Classification:** router

**Reason:** Three conditional branches, no procedural steps — decision logic for routing to other behaviors or skills. Belongs in `UserPromptSubmit` hook.
