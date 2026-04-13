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
