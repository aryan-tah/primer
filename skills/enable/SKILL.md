---
name: enable
description: Use when the user wants to enable Primer for their project - creates the .primer marker file and starts onboarding if needed
---

# Enable Primer

Turn on Primer for this project. Creates a `.primer` marker file in the project root so the session-start hook knows to activate.

## Git Required

<HARD-GATE>
Before doing ANYTHING else, run this check:
```bash
git rev-parse --is-inside-work-tree 2>/dev/null && echo "PRIMER_GIT_OK" || echo "PRIMER_GIT_FAIL"
```

If the output contains `PRIMER_GIT_FAIL` or the command errors with "command not found":

> "Primer requires a git repository to work. It uses git to keep your code map current between sessions — without it, the map goes stale and causes more problems than it solves.
>
> If git is installed, run `git init && git add -A && git commit -m 'Initial commit'` to get started.
> If git isn't installed, you'll need to install it first (https://git-scm.com)."

Then STOP. Do NOT create the `.primer` file. This gate is non-negotiable.
</HARD-GATE>

## The Process

1. Create a `.primer` file in the project root:
   ```
   enabled
   ```
   That's it — just the word "enabled." The hook checks for this file's existence, not its contents.

2. Tell the user:
   > "Primer is now enabled for this project. The `.primer` file has been added to your project root — commit it to git so it persists."

3. If `APP_INTENT.md` doesn't exist, invoke `primer:app-onboarding` to start the setup.
   If both `APP_INTENT.md` and `CODE_MAP.md` already exist, just confirm:
   > "Your map is already set up. Primer will activate automatically at the start of every session."
