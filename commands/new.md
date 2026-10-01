---
description: Decide and record the stack before creating a new Laravel app
argument-hint: "[app name or directory, and any stack constraints you already have]"
---

Establish the stack for a new Laravel application before anything is created: $ARGUMENTS

Invoke the `laravel-stack-interview` skill and follow it exactly:

1. Confirm there is no existing application (no `artisan`, no `composer.json` requiring
   `laravel/framework`). If one exists, use `/laravel:stack` instead.
2. Interview in dependency order, using the `ask` tool in rounds — **framework version first**,
   since it gates everything else; then rendering layer and component layer; then only the axes
   still open (CSS/bundle, auth, data, tests, target). Cap it at three rounds.
3. Validate the coherence chains before recording (maryUI 2 ⇒ daisyUI 5 ⇒ Tailwind 4; daisyUI 5 ⇒
   Tailwind 4; Flux ⇒ Livewire; one component layer per surface; exactly one NativePHP product).
   Surface conflicts with the specific constraint — do not silently comply or silently correct.
4. Fold the answers into a stack number from `references/frontend-stacks.md`, and write the profile
   to `.reasonix/laravel-stack.md` marked `Chosen, not detected`.
5. Scaffold only after recording, then **read the generated `composer.json` / `package.json` /
   `app.css` back** and confirm they match the decision. Scaffolder defaults are not your defaults.
6. Run `laravel-route` before writing the first feature.

If the run is non-interactive, state the assumed stack prominently and mark every axis as assumed.
