---
name: load-state
description: "Find and read the project's saved state file (written by /save-state) to resume work with full context after a /clear, /compact, or a brand-new session — no need to re-explain where things stand."
argument-hint: "[optional: path to a specific state file, if the default discovery shouldn't be used]"
allowed-tools:
  - Read
  - Glob
  - Bash
  - Grep
---

<objective>
Resume this project immediately with full context by finding and reading whatever state file `/save-state` (or an already-established equivalent convention) maintains, then briefing the user with a concise summary of where things stand and what's next — without making them re-explain anything that was already captured.

This is the read half of `/save-state`'s write half — the two must agree on where state lives and how to interpret it. If `/save-state` isn't installed or hasn't been run in this project yet, say so plainly rather than guessing.
</objective>

<process>

## 1. Locate the state file

Use the SAME search order `/save-state` uses, so the two skills never disagree about where state lives:

- `.planning/STATE.md`
- `STATE.md` at the project root
- `.claude/STATE.md`
- Any other file whose name/location strongly suggests it's already serving this purpose

If none of these exist: say so plainly. Do not fabricate a summary from git history alone and present it as if it were a saved state — that's a materially weaker, unverified substitute (no captured root causes, no "what's unverified" honesty, no session narrative). Offer to run `/save-state` going forward, or to do a best-effort git-log-based orientation ONLY if the user explicitly asks for that instead.

## 2. Read the FULL file, not just the header

The real value is in the body — "what was done this session," "bugs fixed (with root cause)," "what's NOT verified," "blockers," "next step" — not just a status/frontmatter line. Read all of it before summarizing anything.

## 3. Cross-check the file's claims against current ground truth before trusting it blindly

A saved state file goes stale the moment work happens outside a session that ends with `/save-state` — don't silently treat it as current without checking:

- If it's a git repo and the file records a commit hash (e.g. `state_head`) or references specific commits: run `git log --oneline -10` and `git status --short`. If `HEAD` has moved past what the file recorded, say so explicitly — everything after that commit is UNKNOWN to the file and should be treated as needing re-discovery, not silently papered over as if the file already accounts for it.
- If the file claims something is "currently running" (a background process, a dev server, a watched log stream): actually verify with `pgrep`/`ps` or the equivalent rather than assuming it's still true — these go stale fast and a wrong assumption here wastes the user's time.
- If the file references version numbers, build artifacts, or file paths: spot-check at least one for plausibility (e.g. does the file still exist) rather than repeating it uncritically.

## 4. Brief the user concisely

Summarize, don't re-paste the file verbatim:

- What the project currently does and its overall state, in plain language.
- What was done recently, calling out anything the file marked as NOT yet verified (surface this prominently — it's exactly the kind of detail easy to silently drop in a summary).
- Any blockers or open concerns worth knowing before doing anything else.
- The file's own recommended next step, if it has one — and whether that's still consistent with what step 3's cross-check found.

Keep it tight enough to be genuinely useful at a glance, not a wall of restated text. The user was just given this exact information once already (by whoever ran `/save-state`) — this is a reminder, not a first read.

## 5. Don't start unprompted work after briefing

Loading context is this skill's whole job. After the summary, wait for the user's actual next instruction — do not automatically start executing the file's "next step" unless the user's own message invoking this skill already asked for that explicitly.

</process>

<what_good_looks_like>
After this skill runs, the user should be able to say "keep going" with zero further explanation and have the session pick up exactly where the last one left off — including correctly distrusting any part of the saved state that step 3's cross-check flagged as stale.
</what_good_looks_like>
