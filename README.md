# Primer

**Claude Code reads your entire project every time you start a session. On the Pro plan, that eats your quota fast. Primer fixes that.**

Most people building with Claude Code don't even know they're wasting tokens. Every session, Claude blindly reads 5-15 files trying to figure out where things are and what your app does. It does this from scratch, every single time. And after all that reading, it still sometimes misunderstands your project.

Primer replaces that with a one-time setup: you describe your app, Primer maps your files, and from then on Claude navigates with purpose instead of exploring blindly. ~300 tokens instead of thousands. Every session. Forever.

## How It Works

There's no configuration, no setup files to edit, no terminal commands to memorize. It's just a conversation.

**1. You describe your app**
Primer asks you what you're building, who it's for, and what problem it solves. Just a few questions, one at a time. Your answers get saved to a small file called `APP_INTENT.md`.

**2. Primer walks through your project with you**
It goes folder by folder through your codebase, writes a one-line description for each file, and shows you each folder before moving on. You say "looks good" or correct anything that's off. The result is a simple map saved to `CODE_MAP.md`.

**3. That's it. You're done.**
Every session after, Claude loads your map (~300 tokens) instead of reading your whole project (~3,000-10,000 tokens). It knows what your app does because you told it. It knows where things are because the map says so.

**4. The map stays current automatically**
Primer checks what changed in git since the last scan and only re-describes those files. You review the updates, approve them, done. No full re-scans.

## Who This Is For

- **Non-technical builders** using Claude Code to build their first app — you shouldn't need to understand MCP servers or vector databases to stop wasting tokens
- **Pro plan users** who hit their quota too fast and want every token to count
- **Anyone with a growing codebase** where Claude keeps getting slower and more confused each session
- **Developers** who want a simple, zero-infrastructure solution that just works

## How It's Different

There are other tools that index codebases for AI. They use vector databases, knowledge graphs, Tree-sitter parsers, and MCP servers. They're powerful — and they require technical knowledge to set up and maintain.

Primer is the human-first alternative. No infrastructure. No dependencies. Just two small markdown files that anyone can open, read, and edit. The setup is a conversation, not a configuration.

And Primer does something none of those tools do: it asks *you* what your app is for. An automated scanner can tell Claude "this file handles authentication." Only you can tell Claude "I built this for small restaurant owners who've never used software before, and simplicity is everything." That context makes Claude dramatically smarter about your project — no amount of indexing can replace it.

## Requirements

- **Git.** Your project must be a git repo. Primer uses git to detect what changed between sessions so the map stays current. Without git, the map goes stale silently and Claude navigates to wrong places confidently — that's worse than no map. If your project isn't a git repo yet, just run `git init` and make a first commit before installing Primer.

## Install

```bash
claude plugin add primer
```

Or install from source:

```bash
git clone https://github.com/aryantah/primer.git
claude plugin add ./primer
```

## What Gets Created

Primer creates two files in your project root:

| File | What It Is | Who Writes It |
|------|-----------|---------------|
| `APP_INTENT.md` | What your app does, who it's for, key flows | You, through conversation |
| `CODE_MAP.md` | Annotated file tree with one-liner per file | Primer generates, you review |

Both are small, readable markdown. Commit them to your repo so they persist across machines and teammates.

## The Skills

Primer is built as four composable skills that work together:

| Skill | What It Does | When It Runs |
|-------|-------------|--------------|
| `app-onboarding` | Asks you to describe your app in your own words | Once, on first use |
| `project-scanner` | Walks through your project folder by folder with you | Once, after onboarding |
| `map-updater` | Checks git for changes, updates only what changed | Each session start |
| `navigator` | Makes Claude consult the map before reading any file | Every session |

## The Token Math

A typical Claude Code session without Primer:
- Claude reads 5-15 files to understand your project: **2,000-10,000 tokens**
- This happens every session, from scratch

With Primer:
- Claude loads APP_INTENT.md + CODE_MAP.md: **~300-500 tokens**
- Navigates directly to what it needs

Over 50 sessions, that's tens of thousands of tokens saved — just on navigation. That's quota you get back for actual work.

## FAQ

**Does this work without git?**
No. Git is required. Primer uses git to detect file changes between sessions and keep the map current. Without it, the map would go stale and Claude would confidently navigate to the wrong places. If your project isn't a git repo yet, run `git init` and make a first commit.

**What if my project is huge?**
Primer will ask if you want to scan everything or focus on the important folders first. A partial map that covers your main code is still better than no map at all.

**What if the map gets outdated?**
If Claude opens a file and it doesn't match the map description, it'll flag it and ask if you want to update that entry. The map self-corrects through use.

**Can I edit the map manually?**
Absolutely. It's just markdown. Open `CODE_MAP.md` in any editor and change whatever you want. Primer won't overwrite your manual edits.

**Does this replace Superpowers?**
No — they're complementary. Superpowers changes how you *build* (TDD, brainstorming, plans). Primer changes how Claude *navigates your project*. Use both.

## Contributing

This is v1. There are rough edges. If you find them, [open an issue](https://github.com/aryantah/primer/issues) or submit a PR. The whole point of open-sourcing this early is to get real-world feedback from real projects.

## License

MIT
