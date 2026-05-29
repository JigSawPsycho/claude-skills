---
name: git-catchup
description: Summarize what changed on the current branch since the user's own last commit — a "catch me up on what my collaborators did" briefing. Identifies the user's last commit, lists every commit after it, then explains the important code changes, any breaking changes, and a functional rundown of how new features work (or how existing behavior changed). Use when the user asks to "catch up", "summarize commits since my last commit/push", "what changed while I was away", "what did my collaborator(s) do", or wants a functional rundown of recent branch history.
---

# git-catchup

Produce a **functional briefing** of everything that landed on the current branch since the user last contributed to it. The goal is not a commit list — it's understanding: what the new code *does*, what would break the user's existing work, and how behavior changed.

## Output the user wants

Three things, in this order:

1. **Important code changes** — grouped by theme/feature, not by commit. Authors and dates noted.
2. **Breaking changes** — anything that would break the user's existing work or assumptions: changed/removed APIs, renamed files, schema/contract version bumps, config keys, signature changes, moved modules, changed defaults.
3. **Functional rundown** — for *new* features, how they work end-to-end; for *changed* features, how the behavior differs from before. This is the most valuable part: read the actual code, don't just paraphrase commit messages.

## Procedure

### 1. Establish the baseline

Identify who "the user" is and find their last commit on the current branch.

```bash
git rev-parse --abbrev-ref HEAD                 # current branch
git config user.email; git config user.name     # who is the user
git fetch 2>&1 | tail -3                          # be current (don't fail the skill if offline)
```

Find the user's last commit. **Match by both email AND name** — commits are often authored under a different email than the configured one (e.g. a GitHub noreply address, a work vs personal address). Try email first, then name; if they disagree, prefer whichever is more recent and mention the ambiguity.

```bash
git log --author="<email>" -1 --format="%H %ci %s"
git log --author="<name>"  -1 --format="%H %ci %s"
```

Decide the range end. By default summarize the local branch (`HEAD`). If the user said "since my last **push**", compare against the remote tracking branch (`@{u}` or `origin/<branch>`) instead, and first confirm local vs remote with `git rev-list --left-right --count @{u}...HEAD`.

**Handle these cases explicitly:**
- *No commits by the user on this branch* — say so, and offer to summarize the whole branch, or from a ref/date they name, or from where the branch diverged from main (`git merge-base main HEAD`).
- *No new commits since the user's last one* — say the branch is even; there's nothing to catch up on. Check for uncommitted/untracked work and mention it instead.
- *Not a git repo* — say so and stop.

### 2. Survey the commits

```bash
git log <baseline>..<end> --format="%h | %an | %ci | %s"      # the commit list
git log <baseline>..<end> --stat --format="### %h %s (%an, %ad)%n" --date=short
```

The diffstat tells you *where* the work is (which files grew, what's new vs modified) and points you at what to read. Note new files (functional rundown candidates), large modifications (behavior changes), and deletions/renames (breaking-change candidates).

### 3. Read for understanding — this is the real work

A diffstat is not a functional summary. Read the actual changes:

- **Design docs / handoff notes / ADRs first.** Look for `*.md` in `docs/`, `handoff/`, `design/`, `CONTRACTS.md`, `CHANGELOG.md`, READMEs touched by the range. These often *state* the intent, the locked decisions, and explicitly call out contract/schema version bumps. They are the highest-signal-per-token source.
- **New modules** — read their top-of-file docstring/header and public surface to explain what they do and how they fit the data flow.
- **Heavily modified core files** — read the diff (`git diff <baseline>..<end> -- <path>`) to see how behavior changed, not just that it did.
- **Contract/interface files** — schema versions, API route definitions, config templates, type definitions, public exports. Changes here are the breaking-change candidates.

Use `git show <sha> -- <path>` or `git diff <baseline>..<end> -- <path>` for specifics. Parallelize reads.

For a large range, fan out: read the cluster of files behind each theme, then synthesize. Don't try to read everything — follow the diffstat's signal (biggest changes, new files, contract files, docs).

### 4. Synthesize

Write the three-part briefing. Guidelines:

- **Group by feature/theme**, then attribute commits — "the recorder build-out (`c238263`, May 27)" reads better than walking commits chronologically.
- **Lead with the one-sentence "big idea"** of the whole range before drilling in. End with a one-sentence bottom line.
- **Be concrete and functional.** "Captures live audio via streamlink-over-SOCKS because ffmpeg can't do SOCKS proxies" beats "added capture support". Cite the mechanism.
- **Flag breaking changes prominently** even if small — a renamed export or bumped schema version is exactly what trips up someone resuming work.
- **Note anomalies** worth the user's attention: committed build/runtime artifacts that look like they belong in `.gitignore`, secrets, generated files, debug logging left in, TODO/FIXME spikes.
- Reference commits as short SHAs and files as `path:line` so they're clickable.
- Scale to the range: 1–3 commits → a few tight paragraphs; a large range → themed sections with a table of commits up top.
- Offer a concrete next step (deep-dive a specific change, scrub stray artifacts, review a breaking change).

## Notes

- Read-only by default — this skill summarizes, it doesn't modify the tree. Only act further if the user asks.
- If `git fetch` fails (offline/no remote), continue against local refs and say the remote view may be stale.
- Don't invent. If a commit message claims something the diff doesn't show, trust the diff and say so.
