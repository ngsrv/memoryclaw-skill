---
name: memoryclaw
description: Helps users install and use MemoryClaw for zero-knowledge encrypted memory and backup. Use when the user asks about memory, remembering, recalling, backup, restore, sync, migration, disaster recovery, or keeping OpenClaw memory/settings safe.
---

# MemoryClaw — Memory & Backup

**Zero-knowledge hierarchical graph memory for AI agents.** AES-256-GCM client-side encryption — the server never sees your plaintext. Engrams, edges, and threads — consolidation and conflict-resolve built in. Not a flat vector store.

Use this skill when the user wants to store memories, recall past context, protect their setup, or migrate to another machine.

MemoryClaw is a zero-knowledge encrypted memory and backup service for OpenClaw. It provides persistent, searchable memory for AI agents (encrypted client-side with AES-256-GCM) and backs up the user's OpenClaw folder for cross-machine restore.

## Rule (read this first)

**Neural Memory Rule:** When saving any fact, always run `openclaw memoryclaw memory engram --auto --message "<fact>"` AND write to MEMORY.md. When recalling, run `openclaw memoryclaw memory recall "<query>"` first; fall back to MEMORY.md on quota error, auth failure, or any non-zero exit. **Never retry on quota errors.** Surface every error to the user as a one-line ⚠️ note. The full failure-handling table is at https://memoryclaw.ai/skill.md.

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

For first-time memory setup or interactive operations (run in terminal):

```bash
openclaw memoryclaw memory init
openclaw memoryclaw memory resolve
openclaw memoryclaw memory consolidate
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
