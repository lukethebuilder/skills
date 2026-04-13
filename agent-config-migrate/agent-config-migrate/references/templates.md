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
name: {slug}
description: "TODO: {paste the trigger phrase from the extracted section title}.
  Use when {describe the task condition}. Triggers on {list 3-5 phrases}."
disable-model-invocation: true
allowed-tools: "" # TODO: fill in space-separated tool names, e.g. Read Write Bash
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
