---
name: project-scanner
description: Use when no CODE_MAP.md exists in the project root, or user asks to re-scan the project - goes folder by folder generating one-line file descriptions with human review after each folder
---

# Project Scanner

Generate a lightweight directory index with one-line descriptions per file. Go folder by folder, get human sign-off on each section. Write it to `CODE_MAP.md`.

This is a one-time setup. The map is small (~300 tokens for a medium project), loads every session, and replaces blind file exploration.

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

Then STOP. Do NOT proceed. Do NOT create CODE_MAP.md. This gate is non-negotiable.
</HARD-GATE>

## When to Run

- Project is a git repo **and** no `CODE_MAP.md` exists in the project root
- User explicitly asks to re-scan or regenerate the map
- After `primer:app-onboarding` completes

## The Process

### Step 1: Identify Top-Level Structure

List the top-level directories in the project. Skip common ignores: `node_modules`, `.git`, `dist`, `build`, `__pycache__`, `.next`, `vendor`, `coverage`, `.cache`, etc. Also respect `.gitignore`.

Present to the user:

> "Here are the top-level folders in your project:
> - `src/`
> - `tests/`
> - `config/`
> - `scripts/`
>
> I'll go through each one and generate short descriptions for every file. After each folder, I'll show you what I came up with so you can correct anything. Sound good?"

Wait for confirmation before proceeding.

### Step 2: Scan Folder by Folder

For each top-level directory:

1. **Read each file briefly** — first 30-50 lines is usually enough to write a one-liner. Don't read entire files.
2. **Generate the annotated tree** for that folder:

```
src/auth/
├── verification.ts      # Email verification via OTP codes
├── session.ts           # JWT session creation and validation
└── middleware.ts        # Express auth guard middleware
```

3. **Show the user for review:**

> "Here's what I've got for `src/auth/`:
> ```
> src/auth/
> ├── verification.ts      # Email verification via OTP codes
> ├── session.ts           # JWT session creation and validation
> └── middleware.ts        # Express auth guard middleware
> ```
> Does this look right? Any corrections?"

4. **Apply their corrections** before moving to the next folder.

5. **Repeat** for every top-level directory.

### Step 3: Handle Root Files

After all directories, scan root-level files (`package.json`, `tsconfig.json`, `.env.example`, Dockerfile, etc.) and present those too:

> "And here are the root-level files:
> ```
> ├── package.json            # Dependencies and scripts
> ├── tsconfig.json           # TypeScript compiler config
> ├── Dockerfile              # Production container build
> └── .env.example            # Required environment variables
> ```
> Anything to adjust?"

### Step 4: Write CODE_MAP.md

Once all folders are reviewed and approved, assemble the full map and write it to `CODE_MAP.md` in the project root:

```markdown
<!-- primer-git-hash: a1b2c3d -->
<!-- primer-scanned: 2024-01-15 -->

# Code Map

project-root/
├── package.json            # Dependencies and scripts
├── tsconfig.json           # TypeScript compiler config
│
├── src/
│   ├── auth/
│   │   ├── verification.ts      # Email verification via OTP codes
│   │   ├── session.ts           # JWT session creation and validation
│   │   └── middleware.ts        # Express auth guard middleware
│   └── payments/
│       ├── stripe.ts            # Stripe payment intent handling
│       └── invoices.ts          # PDF invoice generation
│
└── tests/
    ├── auth/
    │   └── verification.test.ts # Tests for email verification flow
    └── payments/
        └── stripe.test.ts       # Tests for Stripe integration
```

**Critical:** The `primer-git-hash` comment at the top is how `primer:map-updater` knows when the map is stale. Get it by running `git rev-parse --short HEAD`. Always include it. Also include the scan date.

### Step 5: Confirm Completion

> "Code map saved to `CODE_MAP.md`. From now on, I'll use this map to navigate your project instead of reading files blindly. If your codebase changes, the map will auto-update at the start of each session."

## Scanning Rules

- **One line per file.** No exceptions. If a file does too many things, describe the primary responsibility.
- **Skip generated files.** Don't map lock files (`package-lock.json`, `yarn.lock`), build output, or generated code.
- **Use the user's language.** If they called it "the payments module" in `APP_INTENT.md`, call it that in the map.
- **Respect .gitignore.** If it's gitignored, don't map it.
- **Nest naturally.** Show the actual directory hierarchy with tree characters (`├──`, `└──`, `│`).

## For Large Projects

If the project has more than ~15 top-level directories or 200+ files:

> "Your project is pretty large. Want me to scan everything, or focus on the most important folders first? I can always add more later."

Let the user decide what to prioritize. A partial map that covers the hot paths is better than an exhaustive map that burns tokens to generate.
