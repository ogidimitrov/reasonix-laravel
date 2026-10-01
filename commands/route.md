---
description: Route the detected Laravel stack to the skills and version knowledge that apply
argument-hint: "[task you are about to do, to sharpen the route]"
---

Route this Laravel application to the exact skills and version knowledge it needs, for: $ARGUMENTS

Invoke the `laravel-route` skill and follow it exactly:

1. Read `.reasonix/laravel-stack.md`. If it is missing, run `laravel-stack` first — do not route
   from assumption. If it is stale against `composer.lock` / `package-lock.json`, refresh it.
2. Consult `references/routing-table.md` and match every signal present in the profile —
   framework and skeleton era, rendering layer, CSS/component layer, JS, auth, data/async,
   and quality tooling. Layers compose; route all of them.
3. Emit the **version knowledge pack**: only the rules in force for this app's exact versions,
   plus the specific APIs that are unavailable on them.
4. Name the documentation sources for the installed versions, using an official `llms.txt` where
   one exists (daisyUI, Filament) and the versioned prose docs where none does (Tailwind,
   Livewire, Inertia).
5. Report the route as an ordered skill list, then actually load the routed skills — naming a
   skill without invoking it injects nothing.
