---
description: Interrogate the plan until syntax confidence is established, before any code is written
argument-hint: "[what you are about to build, so the grill can be scoped to it]"
---

Run the `laravel-grill` confidence gate **before writing any code** for: $ARGUMENTS

Invoke the `laravel-grill` skill and follow it exactly:

1. Walk the chain in order — **intent → packages → version → syntax → fit** — and do not advance past
   a link you cannot establish.
2. **Look up facts, never ask them.** Columns, route names, config keys, and existing abstractions are
   discoverable with `laravel-recon`. Asking the user for a fact that is in the repository is the
   failure this skill exists to prevent.
3. **Ask about decisions, one at a time where the answers are dependent**, each with a recommended
   answer and the reason so the user can confirm rather than compose.
4. Classify every link as **established / assumed / unknown**. An unknown link with an
   expensive-to-reverse decision means **stop and ask** — do not pick the most likely option.
5. If you cannot cite the syntax from the version's docs, this plugin's tables, or the app itself,
   **say "I don't know" and stop** rather than writing something plausible.
6. Record the outcome: resolved facts to `.reasonix/laravel-stack.md`, recurring decisions via
   `laravel-project-rules`, structural choices as a `laravel-design-decision` record.

Skip this entirely when the change is fully constrained, fast to reverse, or has no syntax risk —
grilling trivial work destroys the skill's credibility for the changes that matter.
