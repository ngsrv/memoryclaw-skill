# memoryclaw-skill

Companion ClawHub skill for discovering and onboarding users to MemoryClaw.

Published at: [clawhub.ai/iarayan/memoryclaw-backup](https://clawhub.ai/iarayan/memoryclaw-backup)

## What This Repo Is

This repository contains the standalone ClawHub skill that helps users and agents:

- Discover MemoryClaw through ClawHub skill search
- Understand when MemoryClaw is useful (backup, restore, sync, migration, disaster recovery)
- Install the actual plugin package
- Use safe non-interactive commands for backup, status, and restore workflows
- Handle failures correctly (session expiry, missing passphrase)

## What This Repo Is Not

This repository is **not** the executable plugin.

The installable MemoryClaw plugin lives in the separate repository:

- `ngsrv/memoryclaw`

Users install the plugin with:

```bash
openclaw plugins install clawhub:memoryclaw
```

## Relationship To `memoryclaw` Repo

The `memoryclaw` plugin repo includes a `SKILL.md` file because the plugin ships its own embedded skill/instructions.

This repo is different:

- `memoryclaw` repo: installable code plugin with packaged instructions
- `memoryclaw-skill` repo: standalone discovery skill published to ClawHub skill search

Both SKILLs guide agents to use only non-interactive commands (`push --auto`, `status`, `history`, `whoami`, `claws`, `doctor`) and direct users to their terminal for interactive flows (`pull`, `login`, first-time setup).

## Files

- `SKILL.md` — Skill definition with frontmatter, commands, trigger phrases, security notes, and failure handling
- `examples.md` — Example user intents and recommended response shapes

## Publish

Publish this skill with:

```bash
npx clawhub@latest publish . --slug memoryclaw-backup --name "MemoryClaw Backup" --version 1.0.0 --tags latest
```

## Links

- Skill on ClawHub: [clawhub.ai/iarayan/memoryclaw-backup](https://clawhub.ai/iarayan/memoryclaw-backup)
- Website: [memoryclaw.ai](https://memoryclaw.ai)
- Docs: [memoryclaw.ai/docs](https://memoryclaw.ai/docs)
