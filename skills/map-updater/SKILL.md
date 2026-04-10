---
name: map-updater
description: Use at session start when CODE_MAP.md exists - checks git hash for changes since last scan, re-scans only modified files, and updates the map with human review
---

# Map Updater

Keep `CODE_MAP.md` current without re-scanning the entire project. Check the git hash, diff against it, re-describe only what changed.

## When to Run

At the **start of every session** where `CODE_MAP.md` exists in the project root. This runs before the navigator skill takes over. (Primer requires git — if the project isn't a git repo, the navigator skill will catch it and stop.)

## The Process

### Step 1: Check for Changes

1. Read the `primer-git-hash` comment from the top of `CODE_MAP.md`:
   ```
   <!-- primer-git-hash: a1b2c3d -->
   ```
   **If the comment is missing or malformed** (no match for the pattern `<!-- primer-git-hash: ... -->`), the map has no baseline to diff against. Ask the user:
   > "The code map doesn't have a valid git hash. Want me to do a fresh full scan to rebuild it?"
   If yes, invoke `primer:project-scanner`. If no, proceed with the existing map as-is (it may be stale).

2. Get the current git hash:
   ```bash
   git rev-parse --short HEAD
   ```
   **If this command fails**, something is wrong with the git repo. Tell the user:
   > "Git returned an error when checking the current commit. Your repo might be in an unusual state. Skipping map update for this session — the existing map will be used as-is."
   Then proceed to `primer:navigator` with the existing map. Don't block the whole session over a git hiccup.

3. **If they match** → no new commits. But there might still be uncommitted or untracked changes. Run a quick check:
   ```bash
   git status --porcelain
   ```
   If the output is empty → truly no changes. Say nothing. Proceed to `primer:navigator` silently.
   If the output is not empty → there are uncommitted changes. Proceed to Step 2.

4. **If they differ** → proceed to Step 2.

### Step 2: Identify Changed Files

There are three sources of changes to check — committed, uncommitted, and brand new files.

**Committed changes** (everything between the stored hash and current HEAD):
```bash
git diff --name-only <stored-hash> HEAD
```

**Uncommitted changes** (modified files not yet committed — staged or unstaged):
```bash
git diff --name-only HEAD
```

**Untracked files** (brand new files never git-added):
```bash
git ls-files --others --exclude-standard
```

Combine all three lists and deduplicate. This catches every possible change — whether someone coded for a week and committed everything, coded and forgot to commit, or just created new files.

Filter out:
- `APP_INTENT.md` and `CODE_MAP.md` themselves — Primer's own files, not project code
- Files in gitignored directories (already handled by `--exclude-standard`)
- Generated files (lock files, build output)
- Files that aren't in the map's scope (if the user chose a partial scan)

### Step 3: Re-scan Changed Files

For each changed or added file:
1. Read the first 30-50 lines
2. Generate a new one-liner description
3. Note whether it's new, modified, or deleted

### Step 4: Show Changes for Review

Present **only the changes** to the user:

> "Your codebase has changed since the last scan. Here are the updated map entries:
>
> **Modified:**
> - `src/auth/verification.ts` — was: *Email verification via OTP codes* → now: *Email and phone verification via OTP*
>
> **Added:**
> - `src/auth/phone.ts` — *Phone number verification service*
>
> **Removed:**
> - `src/auth/legacy-auth.ts`
>
> Look right?"

Wait for the user to approve or correct before applying.

### Step 5: Apply Updates

Once the user approves (or makes corrections):
1. Update the relevant entries in `CODE_MAP.md`
2. Add new file entries in the correct position in the tree
3. Remove deleted file entries
4. Update the `primer-git-hash` comment to the current hash
5. Update the `primer-scanned` date

### Step 6: Hand Off

After updating (or if no changes were needed), the `primer:navigator` skill governs navigation for the rest of the session.

## Edge Cases

**Many files changed (10+):** Group by directory and present one folder at a time, same as the initial scan flow.

**Map was manually edited:** That's fine. Only update entries for files that git says changed. Don't overwrite manual edits to other entries.

**Large diffs (50+ files):** Ask the user:

> "A lot has changed since the last scan (50+ files). Want me to go through them, or just do a fresh full scan?"

**Stored hash no longer exists** (e.g., after a rebase or force push): The `git diff` command will fail. Fall back to a fresh full scan:

> "The git history has changed since the last scan (maybe a rebase or force push) — can't do an incremental update. Want me to do a fresh full scan?"

**Git diff command fails for any other reason:** Don't block the session. Tell the user:

> "Couldn't check for changes — git returned an error. Skipping map update for this session. The existing map will be used as-is, but it might be slightly out of date."

Then proceed to `primer:navigator`. A slightly stale map is better than no session at all.

## Why Incremental

A full re-scan on a 100-file project reads 100 files every session. An incremental update after a typical commit reads 3-5 files. Over weeks of daily use, that's thousands of saved file reads — exactly the kind of token savings Primer exists to provide.
