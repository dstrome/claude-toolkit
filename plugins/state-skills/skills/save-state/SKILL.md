---
name: save-state
description: "Write a comprehensive state file capturing the current session's full context, so a future Claude Code session (after /clear, /compact, or in a brand-new session) can resume this exact project without needing this conversation's history."
argument-hint: "[optional note about what changed or why you're saving now]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

<objective>
Persist everything about the CURRENT conversation's session — decisions made, work done, bugs found and fixed (with root causes, not just symptoms), what's still unverified, and exactly where things stand — into ONE durable state file, so that a future session (after a `/clear`, after `/compact`, or a fresh `claude` invocation days later) can read that file alone and resume immediately with full context.

This is not a git commit summary and not a changelog entry. Write it as if briefing a competent engineer who has zero memory of this conversation but needs to keep working on this exact project in the next five minutes. Optimize for that person never having to ask "wait, why was this done this way?" or "is this actually working?"

Works in any project, git-backed or not, GSD-initialized or not. Do not assume this project follows any particular workflow — detect and match whatever convention already exists (see step 1).
</objective>

<process>

## 1. Locate or create the state file — don't fragment state across files

Check, in this order, for an existing state file and reuse whichever one is already established:

- `.planning/STATE.md` (GSD-initialized projects use this — if it exists, this is almost certainly the right file to update)
- `STATE.md` at the project root
- `.claude/STATE.md`
- Any file whose name strongly suggests it's already serving this purpose (ask the user if genuinely ambiguous rather than guessing wrong and creating a second, competing file)

If none exists, create `.claude/STATE.md` at the project root by default (create the `.claude/` directory if needed) — this keeps a generic-sounding filename out of the repo root unless the project's own conventions call for it there.

Once you've picked a file, use it every time this skill runs in this project. Never create a second state file alongside an existing one.

## 2. Gather ground truth — don't rely on conversational memory alone for verifiable facts

- If this is a git repo: `git log --oneline -30` (more if this session covered more ground) for the exact commit list, and `git status --short` / `git diff --stat` to catch anything uncommitted. **Uncommitted work is the single most common thing lost across a `/clear` — flag it explicitly and prominently if any exists, don't bury it.**
- If there's a test suite and this session ran it, use the actual last-known pass/fail count from this session's own output — don't re-run tests solely to produce a number unless that's cheap and fast.
- Check for anything this session started that's still running or otherwise live: background processes, dev servers, watched log streams, open ports — note exact PIDs/commands to restart them, don't assume a future session can rediscover this by trial and error.
- Note any state changed OUTSIDE the repo this session: system/OS settings, config files elsewhere on disk, external services or accounts touched, environment variables set. These are invisible to git and the easiest thing to forget.

## 3. Write the state file

Adapt section headers to match the existing file's own conventions if one already exists — consistency across saves matters more than a rigid template. If creating fresh, use something close to this shape:

- **Header/status line** — one or two plain-language sentences: what's the state of the project RIGHT NOW. Skip jargon (like phase numbering) the project doesn't already use.
- **What was done this session** — chronological, one entry per meaningful unit of work (a feature, a bug fix, a decision), not one entry per commit if several commits are one logical change, and never vague ("various fixes," "some cleanup"). For every bug that was fixed, capture the ROOT CAUSE, not just the symptom that was observed — a future session re-deriving "why did X actually break" from scratch is exactly the wasted effort this file exists to prevent.
- **What's NOT verified / open loops** — be explicit and honest about anything implemented but not actually confirmed working: no display/audio access to check by eye, waiting on the user to test something live, an assumption that hasn't been validated yet. Never let this section imply something works when it hasn't actually been checked — that's worse than not mentioning it at all.
- **Current blockers/concerns** — anything actively in the way, or worth flagging before continuing.
- **Environment/runtime notes** (only if relevant) — running processes, live file locations, versioning state, access/credentials already set up, anything a fresh session would otherwise waste time rediscovering.
- **Next step** — the single most obvious next action, concrete enough that a fresh session can act on it immediately without first asking the user "what should I do?"

Be concrete, not just accurate: name exact files, exact function/method/class names, exact error messages, exact commit hashes. "Fixed a bug in the notification code" wastes the next session's time; "`NotificationCoordinator.handleStrike` was missing the `isRegionSnoozed` check, added at `NotificationCoordinator.swift:262`" is what actually saves it.

## 4. Commit it (if applicable)

If the project is a git repo and there's no indication the user wants otherwise, commit the state file as its own commit (state-file updates are meant to be checked in and diffed over time — this also means the commit history itself becomes a secondary trail of "when did state get saved and why").

## 5. Confirm to the user

Report the exact file path written/updated and a one-line summary of what it now captures. Don't restate the full file content back — they can read it.

</process>

<what_good_looks_like>
Test the result against this: a future Claude Code session, given ONLY this file and no conversation history, should be able to —

- State correctly what the project currently does and doesn't do.
- Know which bugs were already found and fixed, and WHY, without re-investigating from scratch.
- Know exactly what still needs testing/verification, and by whom.
- Pick up the next task immediately, without asking the user to re-explain where things stand.

If any of those would fail, the state file isn't finished yet — go back and add what's missing.
</what_good_looks_like>
