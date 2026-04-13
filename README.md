# skills

A collection of Claude Code skills using the open [Agent Skills](https://agentskills.dev) standard, installable via the `npx skills@latest` CLI.

Skills in this repo are designed to extend Claude Code with focused, reusable workflows — installed once, invokable by name, and kept out of your main `CLAUDE.md`.

---

## Skills

### `agent-config-migrate`

#### What problem it solves

`CLAUDE.md` files tend to grow. What starts as a lean set of constraints gradually accumulates workflow documentation, routing logic, and stale instructions — until the file is hundreds of lines long, loaded into every session whether or not it's relevant. The symptoms:

- Skills don't trigger reliably (the model never "sees" the right instruction at the right time)
- Every session pays the token cost of instructions that only apply to 1-in-10 tasks
- Router logic like "if the user asks to deploy, do X first" lives inline in `CLAUDE.md` instead of in a `UserPromptSubmit` hook where it belongs

#### What it does

Runs a structured audit of your `CLAUDE.md` and generates a migration plan to the skills+hooks architecture:

1. **Audit** — reads your `CLAUDE.md` (including `@import` files), counts lines, notes already-extracted skills
2. **Classify** — labels each section as one of:
   - `invariant`: universal constraints to keep in `CLAUDE.md`
   - `workflow`: multi-step procedures to extract as skill stubs
   - `router`: conditional routing logic to extract to a `UserPromptSubmit` hook
   - `bloat/meta`: scaffolding, notes, or spec content that can be deleted
3. **Report** — outputs a section-by-section table with estimated token savings
4. **Generate** (with `--apply`) — writes the extracted skill stubs, the hook file, and a rewritten slim `CLAUDE.md`; backs up the original as `CLAUDE.md.bak`

#### What it does NOT do

- It does not improve bad instructions — if a workflow is vague, the extracted skill stub will be equally vague
- It does not touch your code, tests, or anything outside `CLAUDE.md` and `.claude/`
- It does not automatically commit anything — all output is local and reviewable before you commit

---

## Installation

Install into a Claude Code project using the `skills` CLI with the `--agent claude` flag:

```bash
npx skills@latest add lukethebuilder/skills/agent-config-migrate --agent claude
```

By default, `skills` installs into `.agents/skills/`. The `--agent claude` flag maps the install to `.claude/skills/` instead, which is where Claude Code looks for skills.

---

## Usage

From inside a Claude Code session in your project:

**Audit only (no files written):**
```
Run the agent-config-migrate skill on this project.
```
or
```
/agent-config-migrate
```

**Generate skills, hook, and rewritten CLAUDE.md:**
```
/agent-config-migrate --apply
```

The skill will ask for confirmation before writing anything.

#### Files created or changed by `--apply`

| File | What happened |
|------|--------------|
| `CLAUDE.md.bak` | Backup of your original `CLAUDE.md` |
| `CLAUDE.md` | Rewritten: invariants kept verbatim, workflows replaced with `@reference` comments, "Available Skills" index added |
| `.claude/skills/<slug>/SKILL.md` | One stub per extracted workflow |
| `.claude/hooks/user-prompt-submit.*` | Hook file wiring extracted routing logic to skill names |

---

## When to use this skill

- Your `CLAUDE.md` has grown past ~300 lines
- You keep scrolling to find the right workflow and can't remember what's in there
- Skills feel flaky — they work sometimes but not consistently
- You suspect the model is missing relevant instructions because the file is too noisy
- You want to move to a modular skills+hooks setup but don't want to manually refactor a large file

---

## Limitations & Safety

- **Always commit before running `--apply`.** The skill creates `CLAUDE.md.bak`, but a git snapshot is safer than a backup file.
- The classifier works best when your `CLAUDE.md` uses reasonably clear headings. Freeform prose or heavily nested sections may produce unclassified output that you'll need to sort manually.
- Review the generated skill stubs before committing. Auto-generated `description` fields are drafts — they'll work, but you'll likely want to refine the trigger phrases.
- Review the generated hook file. The routing prompt is populated from your extracted router sections, but you should verify the skill names and conditions are correct.

---

## Roadmap

- **Multi-file support** — audit projects that split configuration across multiple `CLAUDE.md` files or `@import` chains and produce a unified report
- **Smarter heuristics for messy sections** — better handling of freeform prose, nested bullets, and sections that are part invariant, part workflow
- **Configurable thresholds** — flags like `--min-savings 30` to suppress migration recommendations below a token savings threshold
- **Description quality pass** — a follow-on mode that scores generated skill descriptions and suggests improvements based on trigger phrase specificity
- **Cursor Rules export** — a `--cursor` flag to emit `.cursorrules` equivalents alongside the Claude-specific output
