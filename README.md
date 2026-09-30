# claude-toolkit

Claude Code plugins. Private; ask David for access.

## Install

In Claude Code:

```
/plugin marketplace add dstrome/claude-toolkit
/plugin install state-skills@dstrome-toolkit
```

Update later with `/plugin marketplace update dstrome-toolkit`.

## Plugins

### state-skills

Two skills for long or complex work that has to survive a `/clear`, a
`/compact` or a new session days later:

- **`/save-state`** writes one state file in the project (`.claude/STATE.md` by
  default, or an existing `STATE.md` / `.planning/STATE.md`): where things stand,
  what was done and *why* (root causes, not just symptoms), what is not yet
  verified, anything still running and how to restart it, changes made outside
  the repository, and the next step. It checks git for uncommitted work and
  commits the file.
- **`/load-state`** reads it back, compares it with git history and running
  processes, says plainly what is stale, and briefs you, then waits for your
  instruction.

Why not just `/compact`: a compact is an automatic summary made when the context
fills up, and tends to keep what happened while dropping why, what is unverified
and what not to touch. The state file is a deliberate briefing that lives in the
repository, can be diffed over time and read days later.

Worth knowing: writing a thorough file costs some tokens, so it pays off on long
sessions rather than short ones; and it commits `.claude/STATE.md` into the
project, which suits personal repositories (add it to `.gitignore` if you would
rather not).
