---
name: navigator
description: Use on every session when APP_INTENT.md and CODE_MAP.md exist in the project root - replaces blind file exploration with map-guided navigation to reduce token usage
---

# Navigator

Consult the map before opening any file. Navigate directly instead of exploring blindly.

<HARD-GATE>
Do NOT read files to "understand the project structure." The map already tells you the structure. Do NOT grep/glob for file locations if the map already has the answer. Every unnecessary file read is wasted tokens.
</HARD-GATE>

## The Rule

```
BEFORE opening, reading, or searching for ANY file:

1. CHECK the map — does CODE_MAP.md tell you where to look?
2. CHECK the intent — does APP_INTENT.md clarify what you're looking at?
3. THEN open ONLY the specific file(s) the map points you to

Skip this = wasting the user's tokens.
```

## Git Required

<HARD-GATE>
Before loading anything, run this check:
```bash
git rev-parse --is-inside-work-tree 2>/dev/null && echo "PRIMER_GIT_OK" || echo "PRIMER_GIT_FAIL"
```

If the output contains `PRIMER_GIT_FAIL` or the command errors with "command not found":

> "Primer requires a git repository to work. It uses git to keep your code map current between sessions — without it, the map goes stale and causes more problems than it solves.
>
> If git is installed, run `git init && git add -A && git commit -m 'Initial commit'` to get started.
> If git isn't installed, you'll need to install it first (https://git-scm.com)."

Then STOP. Do NOT load the map. Do NOT proceed. This gate is non-negotiable.
</HARD-GATE>

## Session Start

At the start of each session where both files exist and the project is a git repo:

1. Check if `primer:map-updater` needs to run — compare the `primer-git-hash` in `CODE_MAP.md` to current HEAD. If they differ, run the map-updater flow first.
2. Read `APP_INTENT.md` (small — always load fully)
3. Read `CODE_MAP.md` (small — always load fully)
4. Use these as your primary navigation tools for the entire session

## What Changes

**Without Navigator:**
- Claude reads 5-15 files trying to find where things live
- Each file read costs 200-2000 tokens
- This happens every single session, from scratch
- Claude still misunderstands the app's purpose

**With Navigator:**
- Claude loads ~300-500 tokens of map + intent
- Navigates directly to the right files
- Understands the app's purpose from the human-written intent
- Only reads files it actually needs to modify or understand deeply

## Hard Rules

1. **Map first, files second.** Never grep/glob for a file's location if the map already tells you where it is.
2. **Intent over inference.** Don't guess what the app does from code patterns. The human told you in `APP_INTENT.md`.
3. **Trust but verify.** If a file doesn't match its map description when you open it, flag it — don't silently proceed with stale info.
4. **Don't read what you don't need.** If the user asks about payments and the map shows `src/payments/stripe.ts`, go there directly. Don't read `src/auth/` "for context."
5. **No exploratory reads.** Don't read files "to get a better understanding of the codebase." The map and intent are your understanding.

## When to Ignore the Map

The map is a guide, not a cage. Ignore it when:

- The user explicitly asks you to explore broadly
- A map entry seems wrong or outdated when you open the actual file
- You need to understand cross-cutting concerns that span many files
- The task requires reading files not in the map (newly created files, config changes, etc.)
- You're debugging and need to trace through code the map can't anticipate

In these cases, read what you need — but update the map entry if you discover it's wrong.

## Stale Map Detection

If you open a file and it doesn't match its `CODE_MAP.md` description:

1. Flag it to the user:
   > "The map says `stripe.ts` handles webhook processing, but it looks like it's been refactored to only handle payment intents. Want me to update the map?"
2. If they approve, update just that entry in `CODE_MAP.md`
3. Continue with your task

Don't silently work with stale information. Don't silently fix it either — the user reviews all map changes.

## Missing Files

If either `APP_INTENT.md` or `CODE_MAP.md` is missing:

- **No APP_INTENT.md:** Invoke `primer:app-onboarding`
- **No CODE_MAP.md:** Invoke `primer:project-scanner`
- **Neither exists:** Start with `primer:app-onboarding`, then `primer:project-scanner`

## Why This Exists

Claude Code on the Pro plan has limited tokens. Every file read that could have been avoided is waste. This skill makes those tokens count — navigate with purpose, not by exploration. A 300-token map that replaces 3,000 tokens of file reads every session pays for itself immediately.
