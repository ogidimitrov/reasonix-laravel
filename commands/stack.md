---
description: Detect and record this Laravel application's full stack profile
argument-hint: "[path to the Laravel app, defaults to the workspace root]"
---

Determine and record the stack of the Laravel application at $ARGUMENTS (or the workspace
root if no path is given).

Invoke the `laravel-stack` skill and follow it exactly:

1. Read `composer.lock` for the locked `laravel/framework` version, `composer.json` for the
   PHP constraint and package set, and `package.json` for the front-end stack. Check for
   `app/Http/Kernel.php` versus `bootstrap/app.php` to determine the skeleton era.
2. Write the verified profile to `.reasonix/laravel-stack.md`.
3. Report the profile, flag any EOL or security-only framework major, and name the skills
   that apply to this stack (`laravel-versions` plus the matching front-end/admin skill).

Do not plan or perform any code change in this command. Detection only.
