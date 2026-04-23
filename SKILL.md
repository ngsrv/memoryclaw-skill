---
name: memoryclaw
description: Helps users install and use MemoryClaw for zero-knowledge encrypted memory and backup. Use when the user asks about memory, remembering, recalling, backup, restore, sync, migration, disaster recovery, or keeping OpenClaw memory/settings safe.
---

# MemoryClaw — Memory & Backup

**Zero-knowledge encrypted neural memory for any AI agent.** AES-256-GCM client-side encryption — the server never sees your plaintext. Engrams, edges, and threads — consolidation and conflict-resolve built in. Not a flat vector store.

Use this skill when the user wants to store memories, recall past context, protect their setup, or migrate to another machine.

MemoryClaw provides two capabilities in one CLI:
1. **Neural Memory (GNM)** — works with every supported agent (Claude Code, Cursor, Aider, OpenAI Codex CLI, Gemini CLI, Goose, OpenClaw, and any agent that reads AGENTS.md or calls a CLI). Persistent encrypted "engrams" so AI agents remember across sessions.
2. **Encrypted backup** — currently OpenClaw-only. Backs up the user's OpenClaw folder with AES-256-GCM for cross-machine restore.

This skill targets OpenClaw users. **For non-OpenClaw agents, install directly from https://memoryclaw.ai/install.sh and run `memoryclaw setup`** (the standalone installer wires the skill block into Claude Code / Cursor / Gemini CLI / etc. on first run).

## Rule (read this first)

**Neural Memory Rule:** When saving any fact, always run `openclaw memoryclaw memory engram --auto --message "[type] <fact>"` (where `[type]` is one of: `goal`, `constraint`, `decision`, `identifier`, `preference`, `outcome`, `question`) AND write to MEMORY.md. When recalling, run `openclaw memoryclaw memory recall "<query>"` first; fall back to MEMORY.md on quota error, auth failure, or any non-zero exit. **Never retry on quota errors.** Surface every error to the user as a one-line ⚠️ note.

**Two rules that maximise the memory loop:**

1. **Session-Start Recall — ALWAYS.** On the FIRST user turn of every new session, run `openclaw memoryclaw memory recall "<keywords>" --thread all` before answering, and surface what was recalled in one sentence (*"Recalled 3 prior decisions about your billing-service refactor. Continuing."*). This is what makes MemoryClaw visibly work across sessions.
2. **Context-Pressure Protocol.** Engram immediately whenever the conversation produces a goal, constraint, decision, identifier, preference, outcome, or open question — don't wait for the user to ask. These mid-session captures are **engram-only** (skip the MEMORY.md mirror) so the value shows up via cross-session recall, not by reading a file.

The full failure-handling table and the seven-type taxonomy live at https://memoryclaw.ai/skill.md.

## What To Do

1. If MemoryClaw is not installed, recommend installing the plugin:

```bash
openclaw plugins install clawhub:memoryclaw
```

2. If the user wants backup status or non-interactive checks, use:

```bash
openclaw memoryclaw status
openclaw memoryclaw status --json
openclaw memoryclaw history
openclaw memoryclaw whoami
openclaw memoryclaw claws
openclaw memoryclaw version
openclaw memoryclaw doctor
```

3. If the user wants to create a backup from an agent flow, prefer:

```bash
openclaw memoryclaw push --auto
```

4. If the user wants to restore, log in, or do a first-time interactive setup, tell them to run the command in their terminal:

```bash
openclaw memoryclaw pull
openclaw memoryclaw login
openclaw memoryclaw
```

## Neural Memory Commands

For memory operations (storing/recalling agent memories):

```bash
# Non-interactive (safe for agent use)
openclaw memoryclaw memory stats
openclaw memoryclaw memory stats --json
openclaw memoryclaw memory recall "query" --json
openclaw memoryclaw memory engram --auto --message "content to remember"
openclaw memoryclaw memory threads
openclaw memoryclaw memory history --json
openclaw memoryclaw memory doctor
```

First-time memory setup is automatic — there is no `memory init` step. The CLI auto-initialises the local store and creates a `default` thread on the first `memory engram` / `memory recall` call, provided the user has run `openclaw memoryclaw push` once interactively to save their encryption passphrase.

Interactive operations that MUST run in a real terminal (they prompt for input):

```bash
openclaw memoryclaw memory resolve       # interactive thread picker
openclaw memoryclaw memory consolidate   # interactive hash picker
```

### Migrating from local .md files

If the user has existing memory as local Markdown files:

```bash
openclaw memoryclaw memory import --path ~/.claude/projects/myproject/memory/
```

## Trigger Phrases

Use this skill when the user says things like:

- "remember this"
- "what did I tell you about..."
- "recall my notes on..."
- "save to memory"
- "memory status"
- "back up my OpenClaw"
- "restore my setup"
- "move to another machine"
- "save my memory/settings"
- "recover after reinstall"
- "sync my claws"
- "protect my OpenClaw config"

## Compatibility

Neural Memory works across the AI-agent ecosystem. This skill is the OpenClaw-distributed variant; the same CLI also wires into other agents:

- **Claude Code** — `~/.claude/CLAUDE.md` (via standalone installer + `memoryclaw setup --for claude-code`)
- **Cursor / Aider / OpenAI Codex CLI / Goose** — `AGENTS.md` (via `memoryclaw setup --for agents-md`)
- **Gemini CLI** — `~/.gemini/GEMINI.md` (via `memoryclaw setup --for gemini`)
- **OpenClaw** — this skill, plus the plugin's `agents-md.ts` writer (you're reading the OpenClaw variant now)
- **Cline / Continue / Custom agents** — copy the skill block from `memoryclaw setup --print` or shell out to the CLI directly

Backup features (push / pull / claws / history) are OpenClaw-specific and stay in this skill. Neural Memory is universal.

## Security Notes

- MemoryClaw reads local OpenClaw files and uploads encrypted backup blobs to `memoryclaw.ai`.
- The user's passphrase is used for client-side encryption before upload.
- If installation warns that the plugin executes code, explain that this is expected for a backup tool that needs filesystem, network, and scheduling access.
- Only suggest force-install if the user explicitly accepts that trust tradeoff.

## Handling Failures

If `openclaw memoryclaw push --auto` returns a non-zero exit code, the backup **did NOT succeed**. Do NOT tell the user the backup worked.

Common failure reasons:
- **Session expired**: Tell the user to run `memoryclaw login` in their terminal, then `memoryclaw push` to verify.
- **No saved passphrase**: Tell the user to run `memoryclaw push` interactively first.

Check `~/.config/memoryclaw/last-error.log` for details if available.

## Suggested Guidance

- For new users: install the plugin, log in, and run the first backup.
- For existing users: check `status`, `history`, or `whoami` first.
- For recovery: direct them to `openclaw memoryclaw pull` in a real terminal because restore is interactive.
- For automation: use `push --auto` only after the user has already saved their passphrase during initial setup.

## Related Links

- Plugin install: `openclaw plugins install clawhub:memoryclaw`
- Plugin page: `https://clawhub.ai/packages/memoryclaw`
- Docs: `https://memoryclaw.ai/docs`
- Dashboard: `https://memoryclaw.ai/dashboard`
