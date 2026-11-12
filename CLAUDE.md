# int-dev

Claude Code plugin marketplace with one plugin, `int`. It does two things: `/int:init` copies the INT standard files into a project, and `/int:doctor` checks that the project still follows the standard. The repo has no build or package manager. It is Markdown and JSON only.

## Layout

- `source/`: source of truth for the INT standard. Do not edit it to fit the templates. Change the source first, then re-copy.
- `plugins/int/templates/`, `plugins/int/design-profiles/`: what gets shipped to target projects, copied from `source/`.
- `plugins/int/skills/{init,doctor}/SKILL.md`: the two skills.
- `OPEN-QUESTIONS.md`: unresolved questions about the standard.

## Verify

```bash
claude plugin validate --strict plugins/int
claude plugin validate --strict .
diff -r source/21-rules-nextjs plugins/int/templates/stacks/nextjs/.claude/rules/stack
```

## Rules

- Do not add agents, hooks or commands that duplicate ECC.
- The plugin version lives only in `plugins/int/.claude-plugin/plugin.json`. Do not add it to `marketplace.json`.
- When you bump the version, also add an entry to `CHANGELOG.md`.
- The repo is public. Do not commit secrets or internal-only content.
