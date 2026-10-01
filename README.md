# reasonix-laravel

A Reasonix plugin that gives agents reliable, **version-aware** Laravel engineering behaviour.

**Plugin version:** 0.2.0 · **skills:** 27 · **commands:** 4 · **references:** 14

It works in four stages: **detect** the application's stack, **route** that stack to the exact skills
and version rules that apply, **enforce** Laravel-specific design, security, and performance
discipline while writing code, and **verify** the result against the app and the installed versions
before claiming it is done.

## Why it exists

Most Laravel mistakes from an agent are not knowledge gaps — they are *version* and *context*
gaps:

- Writing Laravel 13 syntax into a Laravel 11 application.
- Reproducing Tailwind 3 config in a Tailwind 4 app, where it silently produces unstyled output.
- Using daisyUI 5 on Tailwind 3, an unsupported pairing.
- Restructuring a classic-skeleton app into the slim layout Laravel's own upgrade guide advises
  against.
- Assuming a front end. Blade, Livewire, Inertia (React/Vue/Svelte), Filament, and API-only apps
  need different code for the same feature.
- Applying generic "SOLID" advice and wrapping Eloquent in pass-through repositories.

This plugin makes the stack a **prerequisite input** rather than an assumption, and makes the
consequences of that stack explicit before code is written.

## Beyond the knowledge cutoff

This is the plugin's deepest purpose, and the one that makes the rest work.

An LLM's training ends at a date. Laravel ships a major **roughly every year**, and the surrounding
ecosystem ships faster — Laravel 13, Livewire 4, Filament 5, Pest 5, Inertia 3, Tailwind 4,
daisyUI 5, NativePHP 4, maryUI 2. So the installed version is frequently **newer than anything the
model actually knows**, and the failure mode is not visible confusion. It is confident, plausible
code written against an API that has already moved.

The plugin attacks this in three tiers, cheapest first:

| Tier | Mechanism | Example |
| --- | --- | --- |
| **1. Embedded, verified** | Dated, sourced knowledge of what each major introduced and what it broke | `whats-new.md`, `version-deltas.md`, `ecosystem-matrix.md`, plus each stack skill's version split |
| **2. Version-scoped fetch** | Request the exact major's exact page, as markdown. No search, no guessing | `https://laravel.com/framework/docs/{x}.x/{page}.md` · `https://filamentphp.com/docs/{x}.x/{path}.md` |
| **3. Official indices** | The library's own machine-readable surface, which declares its version | `https://daisyui.com/llms.txt` · `https://filamentphp.com/docs/llms.txt` · `https://pestphp.com/llms-full.txt` · `https://fluxui.dev/llms.txt` |
| **4. Third-party corpora (fallback)** | When nothing upstream exists, or the library is outside this plugin's coverage | `https://context7.com/{org}/{repo}/llms.txt` · or the MCP server `reasonix mcp add context7 -- npx -y @upstash/context7-mcp` |

Tier 4 is **secondary and must never outrank tiers 1–3**: it is not `llms.txt` format, its version
selection is topic-based rather than path-based (`…/4.x/llms.txt` is a 404), and its corpus is
derived from each project's own docs — so a contributor-focused repo yields a contributor-focused
corpus. maryUI's 1.9 KB, for instance, documents cloning the repo, not using the library.

Tier 2 is the important discovery: **both Laravel and Filament serve every documentation page as
version-scoped markdown.** `13.x/queries.md` and `12.x/queries.md` both resolve, and differ in
content — so fetching replaces recall with the real thing, at one tool call, with no search step.

The operating rule the skills enforce:

> **Never let a version-specific decision rest on recall.** The plugin's verified knowledge and the
> official versioned docs outrank the model's memory. If an API cannot be cited from either, fetch
> it — or say it cannot be confirmed.

This is what detaches the dependency on the training cutoff: the correct syntax for the installed
version comes from the repository and the versioned docs, not from what the model happened to learn
before its cutoff.

**Honest limit.** Tier 1 goes stale — it is a snapshot, and every section carries its verified date
and source for exactly that reason. That is why the fetch path is primary for depth and tier 1
carries only what is stable enough to be worth embedding. A stale teaching section is worse than
none, because it teaches the wrong version with confidence.

### Majors first, minors on evidence

Laravel's documented policy is the entire risk model:

> Major framework releases are released every year (~Q1), while minor and patch releases may be
> released as often as every week. **Minor and patch releases should never contain breaking
> changes.**

Every backward-compatibility risk therefore sits at the **major** boundary. So version knowledge is
three layers, and they are treated very differently:

| Layer | What it is | Shelf life | Where it lives |
| --- | --- | --- | --- |
| **1. Idiom** | How Laravel, daisyUI, Filament do things. Stable across majors | ages slowly | `laravel-conventions`, `laravel-solid`, the stack skills |
| **2. Major deltas** | What broke, and what now exists | re-verified per major | `version-deltas.md`, `whats-new.md`, `ecosystem-matrix.md` |
| **3. Minor / patch** | Additive features, deprecations, fixes. Ships weekly | expires constantly — **deliberately never embedded** | the library's release notes, on evidence only |

**Priority: major first, idiom second, minor only on evidence.** Layer 3 is not written down here by
design — embedding it is exactly how a version matrix goes stale and starts teaching the wrong thing
confidently.

One trap worth knowing, straight from Laravel's release notes: **named arguments are excluded from
its backward-compatibility guarantees** — "we may choose to rename function arguments". So
`dispatch(job: $job)` is *not* safe across a minor upgrade, even though PHP guarantees named
arguments elsewhere.

## Seeing what is loaded

The plugin cannot draw UI or inject standing instructions — a Reasonix plugin manifest has no
instruction contribution, and skills load on demand. What it *can* do is make the agent print a
context banner, so the active skills and their version gates are visible:

```
▸ laravel · Laravel 12.69.3 · PHP 8.2 · slim skeleton
   skills      conventions · solid · versions(§11→12) · blade-livewire(Livewire 4.x)
               · tailwind(4.x) · daisyui(5.x) · recon · verify · design-decision
   sources     whats-new@2026-09-29 · docs.laravel.com/12.x · routing-table@2026-09-29
   not routed  api · inertia · filament · nativephp
```

Every skill shows the version it applies to; every source shows its verified date; and
**`not routed` is mandatory**, because that is how a user spots a missing skill.

There are exactly two mechanisms for making it appear:

| Mechanism | Coverage | Cost |
| --- | --- | --- |
| Banner mandated by `laravel-route` | Whenever routing runs — i.e. any real Laravel task | free |
| A short block in the project's `AGENTS.md` / `REASONIX.md` | **Every** prompt, because Reasonix folds those into the cache-stable system prompt at session boot | Cached, so effectively free per request |

For guaranteed per-prompt visibility, add the snippet documented in `laravel-route` (section *"Always
show the context banner"*) to the repository's `AGENTS.md`.

## Design and structure

Two skills cover the "how should this be built?" fork:

- **`laravel-solid`** — SOLID applied to Laravel, *and where it stops*. It also governs what to do
  when the user corrects a design choice: comply at the seam named, don't sweep unrelated code, and
  capture the rejection as a project rule so it is not re-proposed next session.
- **`laravel-design-decision`** — the evaluation process, run *before* structure is added.

The procedure is deliberately ordered so that inaction is the default:

```
state the decision + the expected change
  → does the framework already solve it?          (usually yes)
  → does this repo already have an abstraction?    (recon, search before create)
  → is the variation real? two implementations, not a hypothetical
  → pick the smallest pattern that covers it       (patterns.md)
  → price the overhead: files, layers, vocabulary, explanation length
  → record the decision in five lines
```

`references/patterns.md` groups the catalogue by how often each is the right answer: the framework's
own patterns (Form Request, Policy, Resource, Job, Pipeline — reach here first), business-logic
structure (Action, Service, Value Object, Enum, DTO, Event, Observer), patterns for *genuine*
variation (Strategy, Adapter, Decorator, Specification, State machine), patterns the framework
already implements (Singleton, Facade, Builder, Chain of responsibility — hand-rolling one is a
defect), and a be-suspicious list.

The governing principle, stated once:

> **A pattern is not a mark of quality.** It is a cost paid in advance for a named expected change.
> If you cannot name the change, the pattern is pure overhead.

That is enforced structurally: routing precedence rule 3 is *"no structure beats structure"* —
absent a named change, the framework mechanism or plain concrete code wins.

### Syntax by version — what exists where

Majors dictate *what exists*. The plugin also carries an explicit answer to "does this syntax exist
on the version I'm writing for?", because that question has a hard failure mode: **a language or
framework feature above the installed floor is a parse error, a fatal error, or a runtime failure.**
The whole request fails, not one path.

`laravel-versions/references/syntax-by-version.md` covers, verified against official per-version
sources:

| Ecosystem | Coverage |
| --- | --- |
| **PHP 8.1 → 8.5** | Enumerations/readonly/Fibers (8.1) · readonly classes/`SensitiveParameter` (8.2) · typed class constants/`#[\Override]` (8.3) · **property hooks**/`#[\Deprecated]`/lazy objects (8.4) · pipe operator/`#[\NoDiscard]`/asymmetric visibility (8.5) |
| **Inertia v3** | What's new (Vite plugin, HTTP requests, optimistic updates, layout props, simplified SSR) and what breaks (**Axios removed**, `qs` removed) |
| **Pest ↔ PHPUnit** | The mapping that bites: Pest 5→PHPUnit 13, 4→12, 3→11, 2→10 — versus Laravel's *own* skeleton pins |
| **Filament** | Namespaces move every major, so the file points at Filament's **fetchable versioned `.md` docs** instead of a delta table that would rot |
| **Alpine** | The v2 → v3 upgrade guide, plus the bundled-by-Livewire trap |

The file also carries a **coverage matrix** that names what is *not* covered — Flux and daisyUI
thickness, and the low-churn utilities (Pint, Sail, Horizon, Dusk) that are excluded deliberately,
because recording their version syntax would create maintenance with almost no iteration savings.

The guarantee is stated precisely: **presence is what the file gives you; the exact syntax comes from
the linked page for the installed version.** A feature name plus the authoritative URL is what makes
correct syntax reachable without guessing — and it is the specific thing a cutoff model gets wrong.

## Performance and security

Two classes of consideration decide whether an application holds up in production, and both get a
dedicated entry point.

### Performance — impact-ordered, because most work aims at the wrong thing

| Rank | Cause | Typical effect |
| --- | --- | --- |
| **1** | **N+1 queries** | One query per row turns a 20 ms page into 2 s |
| **2** | **Missing indexes** | Table scans growing linearly with data |
| **3** | **Unbounded result sets** | The page that worked in dev dies in production |
| **4** | **Work in the request that belongs in a queue** | Latency multiplied by every user |
| **5** | **Caching** | Real — but last, deliberately |

That order is the point. **Caching is fifth because caching an N+1 does not fix it** — it hides it
until the cache goes cold, then the problem returns with a cache to blame it on.

Detection comes before optimisation: `Model::preventLazyLoading()` in dev, Laravel **Pulse** (slow
queries and jobs), **Telescope** (development introspection), **Nightwatch** (production APM), and the
database's own slow query log as ground truth. `php artisan optimize` — config, route, view, and
event caching — is a deploy step, and one of the largest wins available for the effort.

### Security — an auditable list, not a vibe

`laravel-conventions/references/security-checklist.md` covers authentication and session,
authorization, input, output, request forgery, rate limiting, secrets, dependencies, and operations.
Each item names the failure it prevents.

The highest-yield section is **authorization**: a model exposed to users with no policy is a
**finding, not an omission**. Authentication is not authorization, and hiding UI is not access
control. Other high-frequency findings: `$guarded = []`, `?sort=` straight into `orderBy`, SVG
accepted as an image, debug mode in production, and a permanent API token.

Request forgery is version-sensitive: **Laravel 13 formalized it as `PreventRequestForgery` with
origin-aware verification**, so custom CSRF configuration needs review on upgrade.

## Install

```bash
reasonix plugin install <path-to-this-repo>     # or: reasonix plugin install .
reasonix plugin doctor laravel                  # confirm capabilities loaded
```

Skills then invoke as `/laravel:<skill>`. Verify the manifest before installing:

```bash
reasonix plugin install <path-to-this-repo> --dry-run
```

## The workflow

```
New app:   /laravel:new     →  choose the stack (interview) → record → scaffold → verify
Existing:  /laravel:stack   →  detect and record the stack profile
Both:      /laravel:route   →  map that profile to skills + version knowledge
                … work …
           ── verification loop (mandated) ──
           recon   →  read the real schema / routes / config / existing abstractions
           verify  →  assumption ledger + cheapest falsifying checks
           ── end loop ──
           /laravel:review  →  audit the change against the resolved version
           /laravel:skill   →  forge project-scoped skills from the profile
           rules           →  after any correction: record it so it never recurs
```

**There is no path that proceeds on an unestablished stack.** When there is no application to
detect, or a code-changing axis cannot be resolved from the files, the plugin stops and asks
rather than defaulting to "latest everything" — an assumed stack is worse than an open question,
because it wears the authority of the routing table while describing an application that does not
exist.

`laravel-stack` answers *what is this app*. `laravel-route` answers *what must the agent load
because of it*. Routing exists so the agent loads neither too little (generic advice) nor
everything (diluted context that buries the version-specific rules).

## How it works

### 0. When there is nothing to detect

If the workspace has no Laravel application, or an axis that changes code cannot be resolved from
the files, the plugin **does not guess**. `laravel-stack-interview` asks the user, in dependency
order — framework version first, because it gates everything else — then validates the answers
against the coherence chains (maryUI 2 ⇒ daisyUI 5 ⇒ Tailwind 4, and so on), records the profile
marked *chosen, not detected*, and only then permits scaffolding.

It then reads the generated `composer.json`, `package.json`, and `app.css` back and confirms they
match the decision. Scaffolder defaults are the scaffolder's, not the user's — and they have
shifted across versions. If the scaffold produced a different stack, the plugin says so rather
than quietly building on a stack nobody chose.

### 1. Detection replaces assumption

`laravel-stack` reads committed evidence — `composer.lock`, `composer.json`, `package.json`,
`.env`, `resources/css/app.css`, the presence or absence of `app/Http/Kernel.php` — and writes a
profile to `.reasonix/laravel-stack.md`. That file becomes the single canonical answer to "what is
this application", so later turns read it instead of re-deriving it (or guessing).

Two details make detection reliable rather than decorative:

- It records **majors, not presence**. "Tailwind present" is unusable; "Tailwind 4 with daisyUI 5"
  is actionable, because the major is what changes the code.
- It records **absences** too. "Livewire: absent" is what keeps a route narrow.

### 2. Routing turns a profile into a version-gated plan

`laravel-route` matches each profile signal against a routing table and emits four things:

| Output | Purpose |
| --- | --- |
| Ordered skill list | What to load, each with its version gate |
| **Version knowledge pack** | Only the rules in force for *these* versions |
| Doc sources for these versions | Versioned Laravel docs; an official `llms.txt` where one exists |
| Prohibitions | The specific APIs that are wrong on this version |

The knowledge pack is the core idea. Instead of loading every version's notes, the agent receives
the handful of constraints that actually bind this app — "attribute-based model config is 13+ only,
this app is 12, so it is unavailable" — and nothing else.

### 3. Enforcement at the point of writing

The routed skills carry the actual rules: `laravel-conventions` for correctness and security,
`laravel-solid` for design, and the stack skill for whichever layer is being touched.

Two references are load-bearing at this exact moment:

- **`package-syntax-map.md`** — the correct **prefix, namespace, and registration** for every package
  the profile detected. The most common *syntax* error is not a wrong API for the version; it is the
  right API with the wrong prefix. And where a prefix is registration-dependent (Blade components from
  maryUI, Filament panels), the map does not guess — it tells the agent to confirm it in the app.
- **`whats-new.md`** — when the installed version postdates the model's knowledge.

Prefixes are treated as facts about the installed package, not as idioms to recall. A plausible
`<x-mary-card>` that is actually registered as `<x-card>` fails at runtime exactly like a
plausible column name — and costs the same iteration.

### 4. The verification loop — where iterations actually die

Every wrong first attempt costs one of two loops: the **cheap loop** (the agent catches and fixes
its own mistake) or the **expensive loop** (the user runs it, finds it wrong, and reports back).
The plugin's purpose is to move failures from the second into the first.

That requires distinguishing two families of mistake (see
`laravel-recon/references/failure-taxonomy.md`):

- **Version mistakes** — an API that does not exist in this major. Public, documented, and cheap to
  gate. `laravel-versions` handles these.
- **Environment mistakes** — code that is *valid Laravel* but wrong for this app: a column named
  `price_cents` assumed to be `price`, a `ChargeInvoice` action that already exists, a route named
  `invoices.view` referenced as `invoices.show`.

**No amount of Laravel documentation can prevent the second family**, because the answer exists
only in the repository. That is why the plugin invests there instead of adding more prose:

| Step | Skill | What it does |
| --- | --- | --- |
| Recon | `laravel-recon` | Reads the real schema, routes, models, config, and existing abstractions *before* writing — including **search before create**, to avoid duplicating what already exists |
| Ledger + verify | `laravel-verify` | Lists every app-specific assumption the written code makes, then runs the cheapest checks that could falsify the likely error (symbol existence → parse → static analysis → schema → focused tests) |
| Record | `laravel-project-rules` | After a human correction, writes a durable project rule — the only mechanism here that **compounds** |

The loop is part of the route, not an optional extra. A route that loads the right stack skills but
skips recon and verify has prevented only the version failures.

### 5. Review

`laravel-review` audits the finished change against the same resolved version.

## Why this makes the agent better

- **It targets iterations, not knowledge.** The goal is not a more informed agent — it is an agent
  that does not need correcting. Most tooling adds documentation, but documentation cannot prevent
  code that is valid Laravel and wrong for *this* app; only reading the app can. So the plugin
  spends its budget on recon and verification rather than more prose.
- **It removes the guessing.** Most Laravel mistakes from an agent are not knowledge gaps but
  version and context gaps. Those are exactly what detection eliminates.
- **It makes silent failures loud.** The dangerous Laravel/Tailwind failures do not throw:
  Tailwind 3 syntax in a v4 app yields unstyled output, and a missing maryUI `@source` line makes
  a correctly-installed package look broken. The skills call these out by name.
- **It respects the version gate absolutely.** On an 11.x app it uses 11.x APIs. It does not
  "helpfully" modernise code to syntax the app cannot run, and it says so when a version is EOL.
- **It covers the combinations, not just the components.** "Livewire + Tailwind" describes at
  least eight different stacks; the plugin routes by composition, so daisyUI + maryUI and
  Flux are not conflated.
- **It stays honest about uncertainty.** Where no official machine-readable source exists
  (Tailwind, Livewire, Inertia), the skills say so instead of inventing one, and unresolved
  profile fields are carried as explicit assumptions rather than filled in.
- **It compounds.** `laravel-stack-forge` turns the profile into project-scoped skills, so the
  second session knows this app's real paths, commands, and conventions — not just Laravel's.

## Relationship to Laravel Boost

Laravel Boost (`laravel/boost`, latest 2.10.0) is Laravel's official agent package and the closest
thing in the ecosystem to this plugin. It is excellent — and it solves a **different problem**.
Verified against the official Boost docs (2026-09-29).

**What Boost is**

| Capability | Detail |
| --- | --- |
| MCP server, 10 tools | Application Info, Database Schema, Database Query, Database Connections, Read Log Entries, Last Error, Browser Logs, Get Absolute URL, Record Rule, Search Docs |
| Version-aware guidelines | Composed per installed package version: Laravel core + 10.x–13.x · Livewire core + 2/3/4 · Inertia core + 1/2/3 (Laravel/React/Vue/Svelte) · Tailwind core + 3/4 · Flux core/free/pro · Pest core + 3/4 · Volt · Folio · Wayfinder · Pennant · Pint · Sail · PHPUnit · MCP · Herd |
| Agent Skills (13) | `livewire-development`, `fluxui-development`, `inertia-{react,vue,svelte}-development`, `tailwindcss-development`, `pest-testing`, `volt-development`, `folio-routing`, `wayfinder-development`, `pennant-development`, `mcp-development`, `infer-conventions` |
| Docs API | ~17,000 pieces, semantically searchable, filtered by installed package versions |
| Project rules | `.ai/rules` plus a `record-rule` MCP tool; `infer-conventions` bootstraps them |
| Extension | `.ai/guidelines/*`; third-party packages ship their own via `resources/boost/` |

**Where we do the same thing** — version-aware guidance; on-demand skills in the same `SKILL.md`
format; detecting what is installed; matching guidance to installed versions; a project-rules
mechanism. Boost is not wrong about any of this, and the overlap is real.

**Where the architectures genuinely differ**

| Dimension | Laravel Boost | This plugin |
| --- | --- | --- |
| Distribution | `composer require laravel/boost --dev` — a package inside every app | A Reasonix plugin; **zero per-app dependency**, works on read-only or remote repos |
| Trigger model | Guidelines are **always-on**; skills on demand; doc search when the agent elects to search | Router loads nothing unless the profile calls for it |
| App introspection | **MCP tools** against a running app — live schema, queries, logs, routes | `laravel-recon` via `model:show`, `db:table`, `route:list`, `config:show` — same ground truth, no MCP, no running server |
| Composition awareness | Loads guidelines per installed package | Detects **stack composition** and validates coherence chains (maryUI 2 ⇒ daisyUI 5 ⇒ Tailwind 4) |
| Prohibitions | Not produced — it supplies the right version's docs | Explicit "unavailable on this version" list — the thing that actually stops wrong-version code |
| Coverage | No guidelines for daisyUI, maryUI, Alpine, NativePHP, Filament, API design, or SOLID | Covered |
| Cost | Always-on guidelines plus a 17k-corpus search | 672 tok fixed; ~14.7k tok for a typical task |

**The honest trade.** Boost is better at live introspection with a running runtime, at semantic
search over a Laravel-maintained corpus, and at being maintained by Laravel itself. This plugin is
better at footprint (nothing to install, nothing to keep updated in the app), at composition
detection and coherence validation, at producing prohibitions, at context economy, and at covering
stacks Boost has no guidelines for.

Crucially, **neither tool prevents environment-class iterations on its own.** Boost supplies the
tools to inspect the app, but the agent still has to choose to use them; a search-first agent that
skips searching gains nothing. `laravel-recon` and `laravel-verify` exist to make reading the app
and re-checking the result a *mandated step* rather than a good intention.

They are complementary. If a project already has Boost, its MCP tools are a better data source for
`laravel-recon` than artisan probes — the profile records its presence, and recon should prefer
them. Nothing in this plugin requires it.

## Skills

| Skill | Purpose |
| --- | --- |
| `laravel-stack-interview` | **Greenfield entry.** Asks the user to choose the stack before anything is created — or to fill any code-changing axis detection cannot resolve |
| `laravel-stack` | **Entry point (existing apps).** Detect framework/PHP version, skeleton era, rendering, CSS/JS layer, auth, data, queue, testing, tooling. Writes `.reasonix/laravel-stack.md` |
| `laravel-route` | **Router.** Maps the profile to exact skills + version gates, and emits the applicable version knowledge pack |
| `laravel-recon` | **Ground truth.** Read this app's real schema, routes, models, config, and existing abstractions before writing code that must match them |
| `laravel-verify` | **Assumption ledger.** Verify app-specific assumptions and run the cheapest falsifying checks before declaring work done |
| `laravel-project-rules` | **Learning loop.** Turn a correction into a durable project rule so the same fix never recurs |
| `laravel-versions` | Verified 10→13 matrix, support status, skeleton-era rules, authoritative per-version doc URLs |
| `laravel-conventions` | Framework-wide correctness, security, and performance rules valid on every version |
| `laravel-solid` | SOLID applied to Laravel — and the point at which it stops |
| `laravel-design-decision` | **Design decisions.** Evaluates whether a pattern is warranted at all, which one fits, and its overhead — with the null option as the default |
| `laravel-blade-livewire` | Blade + Livewire discipline, with `references/livewire-versions.md` (2.x/3.x/4.x) |
| `laravel-inertia` | Inertia SPA: typed props, shared data, forms, partial reloads |
| `laravel-filament` | Filament panels without leaking domain logic into the UI layer |
| `laravel-api` | Sanctum/Passport, versioning, resources, rate limits, Octane safety |
| `laravel-tailwind` | Tailwind **3 vs 4** — CSS-first config, renamed utilities, versioned docs |
| `laravel-daisyui` | daisyUI 4/5 with the official `llms.txt` index and the Tailwind pairing rule |
| `laravel-alpine` | Alpine 3 for local state only, without fighting Livewire |
| `laravel-flux` | Flux UI free/Pro tiers, layered correctly with Livewire |
| `laravel-volt` | Volt single-file components — and migrating off Volt on Livewire 4 |
| `laravel-maryui` | maryUI on the daisyUI stack — version chaining and the `@source` wiring that makes it actually style |
| `laravel-nativephp` | NativePHP desktop 2.x / mobile 3.x–4.x, use cases, and the persistent-runtime lifecycle constraint |
| `laravel-tooling` | Sail, Pint, Larastan, Rector, Pail, Wayfinder, Pennant, Folio, Dusk, MCP |
| `laravel-performance` | **Speed and scale.** Impact-ordered: N+1 → indexes → unbounded sets → queues → caching. Detection with Pulse/Telescope/Nightwatch, plus runtime and deploy levers |
| `laravel-update` | **The research-and-update loop.** Re-verifies embedded knowledge from primary sources, prunes what is contradicted or superseded, and keeps the plugin current |
| `laravel-testing` | The project's actual runner; fakes over mocks; negative security cases |
| `laravel-review` | Read-only subagent review scoped to version correctness, security, N+1 |
| `laravel-stack-forge` | **Meta.** Generates project-scoped skills from the detected stack |

Commands: `/laravel:new`, `/laravel:stack`, `/laravel:route`, `/laravel:skill`.

References load on demand:

| Reference | Contents |
| --- | --- |
| `laravel-stack/references/stack-catalog.md` | Decision criteria for every modern stack, including the CSS/component layer axis |
| `laravel-stack/references/frontend-stacks.md` | Every Tailwind + Livewire composition, the 13 named canonical stacks, and which combinations are coherent |
| `laravel-stack/references/package-syntax-map.md` | Detected package → correct prefix / namespace / registration, and how to confirm the registration-dependent ones in the app |
| `laravel-versions/references/version-deltas.md` | Per-major framework delta detail, 10→13 |
| `laravel-versions/references/whats-new.md` | What each current major introduced, dated and sourced — the teaching layer for versions newer than the model's cutoff |
| `laravel-versions/references/versioning-model.md` | The three layers of version knowledge — idiom (stable), major (breaking, priority), minor (additive) — plus per-ecosystem cadence and the named-arguments trap |
| `laravel-versions/references/syntax-by-version.md` | **Which syntax exists on which version** — PHP 8.1–8.5, Inertia v3, Pest↔PHPUnit mapping, Filament, Alpine — plus a coverage matrix showing what is and is not covered |
| `laravel-design-decision/references/patterns.md` | Architectural patterns for Laravel with fit criteria, overhead, and when to skip each |
| `laravel-conventions/references/security-checklist.md` | The auditable security list — auth, authorization, input, output, forgery, limits, secrets, dependencies, operations |
| `laravel-daisyui/references/components-and-rules.md` | daisyUI's 65 components, colour names and rules, class categories, and the vendor's own 12 usage rules |
| `laravel-versions/references/ecosystem-matrix.md` | Every package version this plugin reasons about, plus the `llms.txt` surfaces |
| `laravel-route/references/routing-table.md` | Signal → skill → version gate mapping, plus precedence rules |
| `laravel-recon/references/failure-taxonomy.md` | Why agents write code that needs a fix iteration, and which mechanism prevents each class |
| `laravel-blade-livewire/references/livewire-versions.md` | Livewire 2.x/3.x/4.x differences |

## Verified facts

Every version and contract in this repository was checked against a primary source rather than
recalled. All verified **2026-09-29**.

| Fact | Source |
| --- | --- |
| Laravel 13.34.0 current, PHP `^8.3`; 12.69.3 `^8.2`; 11.57.0 `^8.2`; 10.50.3 `^8.1` | Packagist `laravel/framework` |
| Laravel 13 skeleton pins `phpunit ^12.5.12`; Laravel 12 pins `^11.5.3` | Packagist `laravel/laravel` |
| Livewire 4.4.7 · Flux 2.20.1 · Volt 1.11.2 · Inertia 3.4.0 · Filament 5.9.0 | Packagist |
| Tailwind 4.3.3 · daisyUI 5.7.47 · Alpine 3.17.4 · Vite 8.3.1 | npm registry |
| Livewire 4 config/routing changes; **Volt absorbed into Livewire 4** | `livewire/livewire` → `docs/upgrading.md` |
| maryUI 2.9.10 (2.x ⇒ daisyUI 5 + Tailwind 4) · 1.41.8 (⇒ daisyUI 4 + Tailwind 3); Laravel 10–13 | Packagist `robsontenorio/mary` |
| NativePHP: desktop 2.3.1, mobile 4.5.2 and 3.3.8, all PHP `^8.3`, Laravel 10–13 | Packagist `nativephp/desktop`, `nativephp/mobile` |
| daisyUI `llms.txt` live, 79 KB, self-declares `version: 5.7.x` | `https://daisyui.com/llms.txt` |
| Filament `llms.txt` live, 44 KB, versioned index | `https://filamentphp.com/docs/llms.txt` |
| Pest `llms.txt` (9 KB) + `llms-full.txt` (~400 KB, entire docs) | `https://pestphp.com/llms.txt` |
| Flux UI `llms.txt` + per-page markdown + v1→v2 upgrade guide | `https://fluxui.dev/llms.txt` |
| Context7 corpora — Livewire 35 KB, Inertia 55 KB, Alpine 45 KB; Tailwind and NativePHP **404** | `https://context7.com/{org}/{repo}/llms.txt` — own format, **not** `llms.txt` |
| Tailwind publishes **no** `llms.txt` (deliberate upstream decision) | upstream proposal rejected |
| **Livewire and Alpine have no `llms.txt`** — their `…/llms.txt` paths return **200 with HTML** (SPA fallback) | content check, not status code |
| Reasonix plugin manifest contract | `reasonix-cli` v1.39.5, probed |

### Self-contained by design

This plugin **does not depend on `laravel/boost`** — Laravel's official agent package. It ships
and maintains its own version matrix and guidance. If a project happens to have Boost installed,
the stack profile records it as an optional additional source; nothing here requires it, and the
routing is identical either way.

### Reasonix format notes (probed against v1.39.5)

Native manifests must declare `apiVersion: "reasonix.io/plugin/v2"` exactly — a v1-style manifest
is **rejected**. Accepted top-level fields are `apiVersion`, `name`, `version`, `description`,
`homepage`, `contributes` (`skills`, `commands`, `agents`, `hooks`, `mcpServers`), `provides`, and
`runtime`. Unknown fields are hard errors, so `author`/`license`/`keywords` must not be added.

## Cost model: why detail is cheap here

This plugin targets **DeepSeek**, whose prefix cache changes the economics decisively.

| USD per 1M tokens (Flash / V4-Pro) | Cache **hit** | Cache **miss** | Output |
| --- | --- | --- | --- |
| Input / output | $0.003 / $0.006 | $0.15 / $0.30 | $0.60 / $1.20 |

Off-peak is half of peak; peak is Mon–Fri 01:00–04:00 and 06:00–10:00 UTC.

Three consequences drive every design decision in this plugin:

1. **Loaded context is ~50× cheaper than fresh context.** A skill body loaded once and then cached is
   effectively free for the rest of the session. **Context is not the thing to ration** — a loaded
   rule that prevents one wrong line is worth far more than the tokens it occupies.
2. **Output costs ~200× a cached-hit input token.** The expensive artifact is what the model
   *writes* — including the wrong code it later has to rewrite.
3. **Round trips dominate.** Every iteration is another cache miss on new content plus another
   output. Time and money point the same way: **fewer iterations.**

So the plugin spends its budget on **detection and detail**, not on brevity. The optimisation target
is not token count. It is: *the correct version, the correct packages, the correct prefixes, and
enough instruction to write it right the first time.*

### What it actually costs

Measured with `tiktoken` (`cl100k_base`) over these files; other tokenizers differ ~±10–20% on
markdown.

| Part | Tokens | When it is paid |
| --- | --- | --- |
| Skills index (27) + commands index (4) | **740** | Always — sits in the cache-stable system prefix |
| Typical task (9 skills + 1 reference, incl. the verification loop) | ~17,300 | Once, when the skills are loaded |
| + `whats-new.md`, when the installed version postdates the model's knowledge | +2,600 | Only on versions the model does not know |
| + `patterns.md`, when a design decision is being made | +1,700 | Only when structure is being chosen |
| + `security-checklist.md`, on an auth/input/output change | +1,700 | Only on security-relevant surfaces |
| + `package-syntax-map.md`, when writing against a detected package | +1,600 | Only when a package prefix matters |
| + `syntax-by-version.md`, when a syntax feature's availability is in question | +2,200 | Only on versions whose syntax is uncertain |
| + `components-and-rules.md`, when writing daisyUI markup | +1,400 | Only on daisyUI surfaces |
| A version or syntax question end to end | ~9,100 | Loads two skills and two references — cheaper than a feature task |
| Every skill routed | ~41,400 | Only if routing over-loads, which the router forbids |
| Absolute worst (all skills + all references) | ~72,800 | Not reachable in practice |

Off-peak Flash, one 20-turn session:

| | Cost |
| --- | --- |
| Fixed index, per request | ~$0.000002 |
| Typical task — first load (a cache miss) | $0.0026 |
| **Typical task — 20 turns total** | **~$0.0036** |
| Performance/security task — 20 turns | ~$0.0028 |
| Worst case (all skills + references) — 20 turns | **~$0.0151** |

**Under one cent per session** — which is the point. At these rates the trade-off disappears: a
typical task loads ~17,300 tokens for **$0.0036** across 20 turns, while a single user-caught
iteration costs a round trip, a re-read, and a rewrite. The plugin is not optimising against your
token bill; it is optimising against iterations, and the token bill happens to be negligible.

Thrift is still correct in exactly three places, and none of them is a token argument:

| Keep | Reason |
| --- | --- |
| The fixed index small | It is paid on every request and competes for the model's attention |
| Artefacts non-redundant | A rule buried in prose is a rule that does not get applied |
| Knowledge current | Stale guidance teaches the wrong version with confidence |

## The research and update loop

This plugin's value is that its embedded knowledge is **current and verified**. That is a property
you maintain — it does not hold on its own. So the loop is defined, executable, and shipped as the
`laravel-update` skill.

**What it does, in order:** re-resolve every tracked package version from Packagist and npm → diff
against the matrix → re-fetch each major's release notes and upgrade guide → verify every fetch
pointer still resolves → **prune** superseded or contradicted content → re-measure the cost numbers
→ bump the version → report a short diff.

**What triggers it:** a new major in any tracked ecosystem (Laravel, Livewire, Tailwind, Filament,
Pest, Inertia, daisyUI, NativePHP, maryUI) · a **quarterly floor** · a support window closing · a
doc URL that stops resolving · a Reasonix upgrade · starting work on an unfamiliar stack.

**On scheduling, plainly.** The plugin cannot schedule itself — it is a repository, not a daemon.
"Scheduled" means *you* run it on a cadence: a cron job that invokes the agent, a CI job, a calendar
reminder, or simply asking. The procedure exists so that is a five-minute task instead of an
open-ended research project.

### The detail rule

The goal is the **latest correct, specific, actionable knowledge** — not the least text. Under
DeepSeek's cache economics, loaded context is ~50× cheaper than fresh context and output costs ~200×
a cached-hit input token, so token count is not the constraint. **Iterations and round trips are.**

Every addition must pass one test:

> **Can this change a line of code, or prevent a fix iteration?** If not, it does not belong.

| Add | Never add |
| --- | --- |
| A version's breaking changes | A version's changelog |
| A constraint that fails *silently* | Style preferences |
| The correct prefix for a detected package | A paraphrase of the package's README |
| Support status and PHP floors | Historical release trivia |
| A pointer to the authoritative source | A restatement of that source |

**Depth lives behind a pointer, not in the body** — for a *staleness* reason, not a token one: a
smaller surface is a smaller thing to re-verify. Where the version-scoped docs already say it, link
them.

Since cached context is cheap, **length is not what to police.** What matters is that every artefact
is non-redundant, current, and detailed enough to prevent an iteration:

| Artefact | Guidance |
| --- | --- |
| Skill body | State the decision and the prohibition. **No upper limit worth enforcing** — an unstated rule costs an iteration; an extra paragraph costs almost nothing when cached |
| Reference file | Split by **decision**, not by size — split when a reader would load it for a different reason |
| Fixed index | ~1,000 tokens. The one place thrift is genuinely right: paid on every request, and it competes for the model's attention |

So the prune target is **staleness and redundancy, not length**. Remove what is contradicted,
superseded, or covered elsewhere; keep everything that changes a line of code — and do not cut a
version gate to hit a number.

**Pruning is not optional.** An update that only adds is how a knowledge plugin dies. Every pass must
remove superseded sections, contradicted claims, and anything that cannot change a line of code.

### Re-verify a version yourself

```powershell
Invoke-RestMethod "https://packagist.org/packages/<vendor>/<pkg>.json"   # PHP
Invoke-RestMethod "https://registry.npmjs.org/<pkg>/latest"              # JS
```
