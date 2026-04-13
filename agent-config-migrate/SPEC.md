# agent-config-migrate: Build Spec

## File Structure

```
agent-config-migrate/
├── SKILL.md                          ← phases, quality rules
└── references/
    ├── classification-guide.md       ← heuristics for invariant/workflow/router
    └── templates.md                  ← report template, skill stub, hook JSON
```

The split follows the same pattern as `react-agent-scaffold`: instructions in `SKILL.md`, large reference content in `references/` loaded on demand. The classification guide is long enough (~150 lines) that inlining it would push `SKILL.md` well past 500 lines.

---

## SKILL.md

```markdown
---
description: Audits your CLAUDE.md and Claude Code configuration, then generates
  a migration plan to the skills+hooks architecture. Classifies each section as
  invariant (keep), workflow (extract to skill stub), or router (extract to
  UserPromptSubmit hook). Outputs token savings estimate, stub skill files, and
  hook boilerplate for reliable skill activation. Run /agent-config-migrate for
  a report-only audit, or /agent-config-migrate --apply to generate the output
  files. Use when CLAUDE.md exceeds 300 lines, skills aren't triggering reliably,
  or you want to adopt modular agent configuration. Triggers on "audit CLAUDE.md",
  "refactor my config", "slim down CLAUDE.md", "skills not triggering".
disable-model-invocation: true
allowed-tools:
  - Read
  - Glob
  - Bash(wc -l *)
  - Bash(ls -la .claude/)
  - Write
---

# agent-config-migrate

Audits CLAUDE.md and generates a migration plan to the skills+hooks architecture.

## Phase 1: Discovery

1. Find and read CLAUDE.md in the current project root.
   - If CLAUDE.md has @import directives, read all imported files too.
   - If no CLAUDE.md exists: report "No CLAUDE.md found" and offer to generate
     a starter template. Stop here.
2. Run: wc -l CLAUDE.md (and each imported file) — record total line count.
3. Glob for .claude/skills/**/*.md and skills/**/*.md — list any already-extracted
   skills. Note their names.
4. Run: ls -la .claude/ — note what directories and files already exist.

## Phase 2: Classification

Read the classification heuristics in
[references/classification-guide.md](references/classification-guide.md).

For each section in CLAUDE.md, assign a type:
- **invariant**: universal constraint applying to every Claude interaction.
  Keep in CLAUDE.md.
- **workflow**: a named, multi-step procedure for a specific task type.
  Extract as a skill stub.
- **router**: conditional/trigger logic deciding what action to take.
  Extract as a UserPromptSubmit hook.

List each section with its assigned type and a one-line reason.

## Phase 3: Report (always output this)

Print the structured audit report using the template in
[references/templates.md](references/templates.md) § Report Template.

Include:
- Total lines, lines to keep, lines to extract
- Estimated token savings percentage (lines extracted / total lines × 100)
- Section-by-section classification table
- For each workflow section: proposed skill slug and one-line description
- Hook recommendation: whether a UserPromptSubmit hook is needed and what
  it should evaluate

If running in audit-only mode (no --apply flag): stop here.
Print: "Review the plan above. Re-run with /agent-config-migrate --apply
to generate the output files."

## Phase 4: Generation (only with --apply)

Before writing any files: print the full list of files that will be created
or modified. Ask for confirmation.

On confirmation:
1. Write .claude/skills/{slug}/SKILL.md for each extracted workflow, using
   the skill stub template in references/templates.md.
2. Write .claude/hooks/user-prompt-submit.json using the hook template in
   references/templates.md. Populate {SKILL_LIST} with the actual skill
   names from step 1.
3. Rewrite CLAUDE.md: keep invariant sections verbatim, replace each
   extracted section with a one-line @reference comment, add an
   "Available Skills" section listing all skill slugs.
4. Create CLAUDE.md.bak (copy of original) before rewriting.
5. Output a summary: files created, lines before/after, token savings.

## Quality rules

- Never write the slimmed CLAUDE.md without first creating CLAUDE.md.bak.
- The invariant sections must be reproduced verbatim in the new CLAUDE.md —
  do not rephrase or summarize them.
- Each skill stub must include the extracted instructions exactly as written,
  not summarized — the user will refine them.
- The UserPromptSubmit hook prompt must include all skill slugs by name.
- If token savings estimate is below 20%, note this in the report:
  "Your CLAUDE.md is already relatively lean. Migration may not be worth
  the overhead."
```

---

## references/classification-guide.md

```markdown
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
```

---

## references/templates.md

```markdown
# Templates

Output templates for agent-config-migrate.

---

## Template 1: Report Output Format

```markdown
## agent-config-migrate: Audit Report

**Project:** {project_name}
**CLAUDE.md:** {total_lines} lines ({imported_lines} from @imports)
**Existing skills:** {existing_skill_count} already extracted

### Classification Summary

| Lines | Type | Recommendation |
|-------|------|---------------|
| {N} | Invariant | Keep in CLAUDE.md |
| {N} | Workflow | Extract to {K} skill stubs |
| {N} | Router | Extract to UserPromptSubmit hook |
| {N} | Unclassified | Review manually |

**Estimated token savings:** {X}% in sessions that don't invoke extracted skills

### Sections to Extract

#### Skills
{for each workflow section:}
- **{section_name}** → `.claude/skills/{slug}/SKILL.md`
  Description: "{one-line trigger description}"
  Rationale: {one sentence why this is a workflow not an invariant}

#### Hook
{if router sections found:}
UserPromptSubmit hook recommended. Will evaluate:
{list of router conditions to move into the hook prompt}

### Sections to Keep (Invariants)
{list section names}

---
Run `/agent-config-migrate --apply` to generate the output files listed above.
```

---

## Template 2: Skill Stub

```markdown
---
description: "TODO: {paste the trigger phrase from the extracted section title}.
  Use when {describe the task condition}. Triggers on {list 3-5 phrases}."
disable-model-invocation: true
allowed-tools: []
---

# {Skill Name}

{Extracted workflow instructions from CLAUDE.md — verbatim, not summarized.}

## Steps
{Numbered steps from the extracted section}

## Quality rules
{Any quality criteria from the extracted section}

## Notes
Extracted from CLAUDE.md by agent-config-migrate on {date}.
Review the description field above — it was auto-generated and may need refinement.
```

---

## Template 3: UserPromptSubmit Hook

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Before responding, check if any of the following skills in .claude/skills/ is relevant to this request: {SKILL_LIST}. If one matches, load and follow its SKILL.md. Extracted routing rules to apply: {ROUTER_RULES}. If no skill applies, proceed normally."
          }
        ]
      }
    ]
  }
}
```

**Note:** `{SKILL_LIST}` is populated with real skill slugs at generation time. `{ROUTER_RULES}` is populated with the extracted router-type sections. Both are substituted by the skill during Phase 4 generation.
```

---

## Test Cases

### Trigger Tests

| Query | Should trigger? |
|-------|----------------|
| `/agent-config-migrate` (in project with CLAUDE.md) | Yes — audit mode |
| `/agent-config-migrate --apply` | Yes — generates files |
| "my CLAUDE.md is 900 lines, help me slim it down" | Yes (if invocation mode default) |
| "my skills aren't triggering reliably" | Yes |
| "audit CLAUDE.md" | Yes |
| "how do I write a good CLAUDE.md?" | No — knowledge query, not audit action |
| "what is a UserPromptSubmit hook?" | No — knowledge query |
| "update my CLAUDE.md to add a new rule" | No — editing, not auditing |

### Edge Cases

- **No CLAUDE.md exists**: report this clearly, offer a starter template, do not error
- **CLAUDE.md has `@import`**: read all imported files and analyze as a unified document; note which sections came from which file in the report
- **Existing skills already present**: skip those in the report — only flag CLAUDE.md sections not yet extracted; acknowledge the already-extracted ones
- **Very short CLAUDE.md (<100 lines)**: output "Configuration is already lean. No migration recommended." Do not generate unnecessary stubs
- **All sections are invariants**: output "No extractable workflows found. Your CLAUDE.md structure is already optimized for the skills architecture" — don't force extraction when it isn't warranted
- **`--apply` without confirmation**: always print the file list and ask for explicit confirmation before writing anything; never write silently

---

## Open Decisions

1. **Interactive confirmation vs. `--apply` flag?**
   Current design uses `--apply` flag (explicit, predictable). Alternative: always prompt interactively ("Found 3 extractable sections, generate files?"). Both work since `disable-model-invocation: true` means this is always user-initiated. Keep `--apply` flag for v1 — more scriptable and predictable. Add interactive as a v2 option.

2. **Hook format portability**
   The `UserPromptSubmit` hook JSON format is Claude Code-specific. For cross-agent portability (Cursor, GitHub Copilot), hooks work differently or not at all. v1 should generate Claude Code-specific hooks with a comment noting this; a `--cursor` flag could generate Cursor Rules equivalents in v2.

3. **Skill stub description quality**
   Auto-generated descriptions are likely to be generic. Phase 4 should generate a description draft and explicitly flag it as needing human review. A follow-on `/description-optimize` skill (running the mellanon activation statistics insight) could help here — defer to v2.

4. **Scope of `--apply` destructiveness**
   Rewriting `CLAUDE.md` is destructive even with `.bak`. Consider generating `CLAUDE.migrated.md` instead, letting the user diff and rename. Safer but adds a manual step. Decision: generate `.bak` AND output `CLAUDE.migrated.md` to be extra safe; document both files in the summary.

5. **Does this cannibalize `portfolio-readme-writer`?**
   No — they serve completely different workflows. But build order matters: build `agent-config-migrate` first since it improves the environment you'll use to build the other two.
