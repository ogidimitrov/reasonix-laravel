---
description: Forge project-scoped Laravel skills from the detected stack
argument-hint: "[area to cover, e.g. billing or api; omit to cover every gap]"
---

Generate project-scoped Reasonix skills for this Laravel application, covering: $ARGUMENTS

Invoke the `laravel-stack-forge` skill and follow it exactly:

1. Read `.reasonix/laravel-stack.md`. If it is missing or stale, run `laravel-stack` first —
   do not infer the stack here.
2. Decide which capability areas have genuine, non-obvious project conventions that the
   general plugin skills would get wrong. Skip anything already covered generically.
3. Gather evidence from real files (existing models, actions, tests, `composer.json`
   `scripts`) before writing. Every path and command in a forged skill must exist.
4. Write each skill to `.reasonix/skills/<name>/SKILL.md` with `name` and `description`
   frontmatter, keeping each body small and operational.
5. Verify discovery with `reasonix doctor capabilities --json`, then report what was forged,
   the evidence behind each, and what was deliberately skipped.
