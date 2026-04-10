---
name: enable
description: Use when the user wants to enable Primer for their project - creates the .primer marker file and starts onboarding if needed
---

# Enable Primer

Turn on Primer for this project. Creates a `.primer` marker file in the project root so the session-start hook knows to activate.

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
