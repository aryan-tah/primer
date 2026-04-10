---
name: disable
description: Use when the user wants to disable Primer for their project - removes the .primer marker file so the session-start hook stays silent
---

# Disable Primer

Turn off Primer for this project. Removes the `.primer` marker file so the session-start hook won't activate.

## The Process

1. Delete the `.primer` file from the project root.

2. Tell the user:
   > "Primer is now disabled for this project. Your `APP_INTENT.md` and `CODE_MAP.md` files are still there — nothing was deleted. Run `primer:enable` anytime to turn it back on."

Do NOT delete `APP_INTENT.md` or `CODE_MAP.md`. The user might want to re-enable later without redoing the setup.
