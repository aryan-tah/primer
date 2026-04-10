---
name: app-onboarding
description: Use when no APP_INTENT.md exists in the project root, or when the user asks to describe their app - conversational onboarding to capture what the app is, who uses it, and why it exists
---

# App Onboarding

Ask the user to describe their app in their own words. Write it down. This is the single highest-value thing you can do to improve Claude's understanding of a codebase.

Code tells you *what*. The user tells you *why*.

<HARD-GATE>
Do NOT skip this step. Do NOT infer the app's purpose from code alone. The human's description is the ground truth — code is just implementation detail.
</HARD-GATE>

## Git Required

<HARD-GATE>
Before doing ANYTHING else — before asking questions, before reading files, before writing anything — run this check:
```bash
git rev-parse --is-inside-work-tree 2>/dev/null && echo "PRIMER_GIT_OK" || echo "PRIMER_GIT_FAIL"
```

If the output contains `PRIMER_GIT_FAIL` or the command errors with "command not found":

> "Primer requires a git repository to work. It uses git to keep your code map current between sessions — without it, the map goes stale and causes more problems than it solves.
>
> If git is installed, run `git init && git add -A && git commit -m 'Initial commit'` to get started.
> If git isn't installed, you'll need to install it first (https://git-scm.com)."

Then STOP. Do NOT proceed. Do NOT create APP_INTENT.md. This gate is non-negotiable.
</HARD-GATE>

## When to Run

- Project is a git repo **and** no `APP_INTENT.md` exists in the project root
- The user explicitly asks to describe or re-describe their app
- First time using Primer on a project

## The Process

**This is a conversation, not a form.** Ask one question at a time. Keep it natural.

### Step 1: The Opening Question

Start with the big picture:

> "Before I start exploring your codebase, I'd like to understand what you're building. Can you describe your app in a few sentences — what it does, who it's for, and what problem it solves?"

Wait for the user's response before continuing.

### Step 2: Follow-Up Questions (one at a time)

Based on their answer, ask about what's still unclear. Pick from these — only what's relevant:

- **Tech stack:** "What's the main tech stack?" (framework, language, database)
- **Key flows:** "What are the 2-3 most important things a user does in the app?"
- **Non-obvious architecture:** "Is there anything about how the project is structured that isn't obvious from the file names?"
- **External services:** "Does the app integrate with any external services or APIs?"
- **Current state:** "Is this a production app, a side project, something you're building from scratch?"

Don't ask all of these. Ask what matters based on the user's initial description. 2-4 follow-ups max. One question per message.

### Step 3: Write APP_INTENT.md

Once you have enough context, write `APP_INTENT.md` to the project root:

```markdown
# App Intent

## What This App Does
[2-3 sentences from the user's description]

## Who It's For
[Target users/audience]

## Tech Stack
[Framework, language, key libraries, database]

## Key Flows
[The 2-3 most important user journeys]

## Architecture Notes
[Anything non-obvious about the project structure — only if the user mentioned something]

## External Integrations
[APIs, services — only if relevant]
```

Keep it concise. Under 50 lines. It's a quick-reference, not documentation.

### Step 4: User Review

Show the user what you wrote:

> "Here's the app intent I've captured. Does this look right, or would you change anything?"

If they suggest changes, update the file. Only proceed once the user approves.

### Step 5: Transition to Scanning

If `CODE_MAP.md` doesn't exist yet:

> "Now let's map out your project structure so I can navigate it efficiently. I'll go folder by folder and you can review my descriptions."

Then invoke `primer:project-scanner`.

If `CODE_MAP.md` already exists, you're done.

## Key Principles

- **One question at a time.** Don't overwhelm.
- **Their words, not yours.** Use the user's language, not jargon they didn't use.
- **Brief is better.** APP_INTENT.md should be skimmable in 10 seconds.
- **This is a one-time cost.** Spend 2 minutes now, save tokens every session after.
