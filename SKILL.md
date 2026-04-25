---
name: memoryclaw
description: Back up + restore OpenClaw context AND give AI agents persistent zero-knowledge memory (engrams, recall, threads). Use when user asks to back up, save, or restore data, or to store/recall persistent memories across sessions.
user-invocable: true
---

# MemoryClaw

**Zero-knowledge encrypted neural memory for any AI agent.** AES-256-GCM client-side encryption — the server never sees your plaintext. Engrams, edges, and threads — consolidation and conflict-resolve built in. Not a flat vector store.

**Supported agents** (via `memoryclaw setup` or manual AGENTS.md paste): Claude Code, Cursor, Aider, OpenAI Codex CLI, Gemini CLI, Goose, OpenClaw, Cline, Continue, and any agent that can call a CLI or read AGENTS.md.

Two capabilities in one CLI:
1. **Neural Memory (GNM)** — works with every supported agent. Store and recall persistent encrypted "engrams" so your AI remembers across sessions. Zero-knowledge: server stores ciphertext only, never sees content.
2. **Encrypted backup** — currently OpenClaw-only. Backs up your OpenClaw folder with AES-256-GCM. Non-OpenClaw users install with `MEMORYCLAW_MODE=memory` and skip this entirely.

Invoke as `memoryclaw <command>` after the install script symlinks it into `/usr/local/bin/memoryclaw`. OpenClaw users who installed via `openclaw plugins install clawhub:memoryclaw` can also invoke it as `openclaw memoryclaw <command>`.

MemoryClaw is owned and operated by **NGSRV, LLC**.

Service features, limits, and terms may change over time; see https://memoryclaw.ai/terms.

Transactional system emails (e.g., password resets) are intended to come from `info@memoryclaw.ai`.

## Compatibility

MemoryClaw's Neural Memory layer works with every major AI-agent ecosystem in 2026:

| Agent | Integration | How wired |
|---|---|---|
| **Claude Code** | Reads `~/.claude/CLAUDE.md` | `memoryclaw setup --for claude-code` |
| **Cursor** | Reads `AGENTS.md` / `.cursor/rules/` | `memoryclaw setup --for agents-md` |
| **Aider** | Reads `AGENTS.md` / `CONVENTIONS.md` | `memoryclaw setup --for agents-md` |
| **OpenAI Codex CLI** | Reads `AGENTS.md` natively | `memoryclaw setup --for agents-md` |
| **Gemini CLI** | Reads `~/.gemini/GEMINI.md` (takes precedence over AGENTS.md) | `memoryclaw setup --for gemini` |
| **Goose** | Reads `AGENTS.md` + `.goosehints` | `memoryclaw setup --for agents-md` |
| **OpenClaw** | Plugin loader | `openclaw plugins install clawhub:memoryclaw` (backup features enabled) |
| **Cline / Continue** | Project-level rules | Manually copy the skill block from `memoryclaw setup --print` |
| **Claude Desktop** | MCP only | Phase 2 — MCP server coming |
| **Custom agents (LangChain, OpenAI SDK, …)** | Shell out to CLI | The agent calls `memoryclaw memory engram` / `memory recall` directly |

The fastest path for ANY agent: `memoryclaw setup` walks the user through login, passphrase, and per-agent wiring in one flow.

## First-time setup

One-time install + setup:

```bash
# 1. Install
curl -fsSL https://memoryclaw.ai/install.sh | MEMORYCLAW_MODE=memory bash
# (or MEMORYCLAW_MODE=full if you also want OpenClaw backup)

# 2. Sign in + set passphrase + wire into detected agents
memoryclaw setup

# 3. (Optional) skip the interactive picker
memoryclaw setup --for claude-code   # or agents-md, gemini, all
```

The setup command detects Claude Code / Cursor / Gemini CLI / Aider / Codex CLI / Goose and writes the MemoryClaw skill block into each detected agent's preferred instruction file using idempotent sentinel markers (`<!-- >>> memoryclaw memory >>> -->` … `<!-- <<< -->`). Re-running `memoryclaw setup` refreshes the block in place; `memoryclaw setup --remove` strips it cleanly.

## CRITICAL: Install Modes — gracefully handle disabled features

A user can install MemoryClaw in one of three modes (picked at install time, changeable later via `memoryclaw mode`):

- **`full`** — Neural Memory + Encrypted Backup (everything enabled).
- **`memory`** — Neural Memory only. Backup commands (`push`, `pull`, `claws`, `history`) exit with a "not enabled in this install mode" message.
- **`backup`** — Encrypted Backup only. Every `memory *` subcommand exits with the same kind of message.

**As an agent, treat these errors as a signal to stop calling the disabled feature for the rest of the session, not as a fault.** If `memoryclaw memory recall` returns "Neural Memory is not enabled", the user has chosen `backup` mode — don't keep retrying recall this session, fall back to MEMORY.md, and don't suggest the user "upgrade their plan" (this is a local config choice, not a billing limit). To check the user's mode:

```bash
memoryclaw mode --json
# returns { "mode": "memory" | "backup" | "full", "label": "...", ... }
```

If you call a disabled command, the error message includes the exact remediation: "Run \`memoryclaw mode --set full\` to enable." Surface that to the user verbatim — it's the right next step.

## CRITICAL: Non-Interactive Mode

**When an AI agent runs memoryclaw (Claude Code, Cursor, OpenClaw, Gemini CLI, any shell-calling agent), it MUST use non-interactive flags only.** Interactive commands (that prompt for user input) will hang and fail.

### Safe commands (use these):

```bash
# Back up — ALWAYS use --auto when running as agent
# Add --quiet (-q) for silent operation (errors only to stderr)
# Add --verbose or --debug for troubleshooting output
memoryclaw push --auto
memoryclaw push --auto --quiet

# Check status including subscription/billing info (non-interactive)
# Add --json for machine-readable output (useful for parsing)
memoryclaw status
memoryclaw status --json

# Check status for a specific claw (non-interactive)
memoryclaw status --claw_id <name>

# List all registered claws (non-interactive)
memoryclaw claws

# Show backup history (non-interactive, supports --json)
memoryclaw history
memoryclaw history --json

# Show logged-in user (non-interactive, supports --json)
memoryclaw whoami
memoryclaw whoami --json

# Show version (non-interactive, supports --json)
memoryclaw version
memoryclaw version --json

# Show config (non-interactive)
memoryclaw config

# Check for updates (non-interactive)
memoryclaw update --check

# Run health checks (non-interactive)
memoryclaw doctor

# Show install mode (non-interactive — see "Install Modes" section above)
memoryclaw mode --json
# Change install mode non-interactively
memoryclaw mode --set full
memoryclaw mode --set memory
memoryclaw mode --set backup

# ── Neural Memory (GNM) — zero-knowledge persistent memory ──
# No manual init needed — every `memory` subcommand auto-initializes the local
# store + creates a "default" thread on first use, as long as the user has
# already run `memoryclaw push` once interactively (which saves the
# encryption passphrase).

# Store a memory non-interactively
memoryclaw memory engram --auto --message "user prefers dark mode"

# Search memories by free-text query (defaults to the active thread —
# pass --thread to scope to a different one, or --thread all to search
# every thread)
memoryclaw memory recall "dark mode preference"
memoryclaw memory recall "deadline" --thread billing-service
memoryclaw memory recall "anything I've ever said" --thread all

# List threads (no subcommand = list)
memoryclaw memory threads

# Create a new thread
memoryclaw memory threads create project-alpha

# Show engram history for the active thread
memoryclaw memory history
memoryclaw memory history --thread <id> --depth 50

# Show plan + GNM usage (engrams/recalls remaining etc)
memoryclaw memory stats

# Run GNM health checks
memoryclaw memory doctor

# Replace 2+ engrams with one summary (all args required for non-interactive)
memoryclaw memory consolidate \
    --hashes hashA,hashB,hashC --summary "merged note"

# Resolve contradicts edges between two threads
memoryclaw memory resolve --source <id> --target <id>

# Import .md files as engrams (deterministic; supports --dry-run)
memoryclaw memory import --path ./notes/
```

### DO NOT run these as agent (they require interactive input):

- `memoryclaw push` (without --auto) — prompts for passphrase
- `memoryclaw pull` — prompts for backup selection, claw picker, and confirmation
- `memoryclaw pull --claw_id <name>` — still prompts for confirmation
- `memoryclaw login` — opens browser and waits
- `memoryclaw claws rename <name>` — requires auth
- `memoryclaw claws delete <name>` — removed from CLI; use website dashboard instead
- `memoryclaw claws add` — prompts for name and directory
- `memoryclaw claws switch` — prompts for claw selection
- `memoryclaw` (no args) — interactive menu
- `memoryclaw memory engram` (without `--auto --message "..."`) — prompts for content
- `memoryclaw memory threads switch` — prompts for thread picker
- `memoryclaw memory consolidate` (missing `--hashes` or `--summary`) — prompts to confirm/select
- `memoryclaw memory resolve` (missing `--source`/`--target`) — prompts to pick threads
- `memoryclaw memory import` (missing `--path`) — prompts for directory

### If push --auto fails with "no saved passphrase"

Tell the user:

> Your encryption passphrase hasn't been saved yet. Please run `memoryclaw push` in your terminal once to set it up — this will save your passphrase locally (encrypted) so future automatic backups work without prompting.

### If push --auto fails with "Session expired" or a non-zero exit code

Tell the user clearly that their **backup did NOT succeed** and they need to re-authenticate:

> Your MemoryClaw session has expired and automatic backups are **not running**. Please run `memoryclaw login` in your terminal to re-authenticate, then run `memoryclaw push` once to verify backups are working again.

**Do NOT tell the user the backup succeeded if push --auto returned a non-zero exit code.** A non-zero exit code always means the backup failed. Check `~/.config/memoryclaw/last-error.log` for details if available.

### Cron job failure handling

When running `push --auto` from an OpenClaw cron job and it fails, **you MUST notify the user immediately.** Do not silently continue. Tell the user:

> **MemoryClaw automatic backup failed.** Your backups are not running. Run `memoryclaw status` to diagnose, or `memoryclaw login` if your session expired. Then run `memoryclaw push` to verify backups are working again.

The user depends on automatic backups working silently. If they break, the user must know — otherwise they believe their data is protected when it isn't.

## When to Use

- User says "back up", "save", or "backup my config/memory/settings" -> run `push --auto`
- User asks about backup status or storage usage -> run `status`
- User asks about their subscription, billing, or plan details -> run `status`
- User asks about backup history -> run `history`
- User wants to know who's logged in -> run `whoami`
- User asks about their claws or what machines are connected -> run `claws`
- User asks about a specific claw's status -> run `status --claw_id <name>`
- User asks about updates or wants to update the plugin -> run `update --check`
- User reports issues or wants to diagnose problems -> run `doctor`
- If user says the plugin is blocked by a beta/launch password page -> that is a website configuration issue; the device login page (`/auth/device`) and `/api/*` must bypass the gate so the plugin can authenticate. (Website login lives at `https://memoryclaw.ai/login` and may support Google sign-in.)
- User wants to restore -> **tell them to run `memoryclaw pull` in their terminal** (interactive only)
- User wants to restore from a specific claw -> **tell them to run `memoryclaw pull --claw_id <name>` in their terminal**
- User wants to log in -> **tell them to run `memoryclaw login` in their terminal** (interactive only)
- User wants to restore from a downloaded file -> **tell them to run `memoryclaw pull --file /path/to/backup.enc` in their terminal**

### Neural Memory (GNM) — explicit user requests

- User says "remember that …", "save this for next time", "store this fact" -> run `memory engram --auto --message "<the fact>"`
- User asks "what do you remember about X?" or "recall …" -> run `memory recall "<query>"`
- User asks how much memory they've used / quotas / GNM plan info -> run `memory stats`
- User wants memory in a separate context (e.g. project, client, persona) -> use `memory threads create <name>` then pass `--thread <id>` on subsequent engrams
- User wants to see what's been saved -> run `memory history` (current thread) or `memory history --thread <id>`
- User wants to import notes/markdown files -> run `memory import --path <dir>`
- User reports memory issues -> run `memory doctor`
- User wants to merge or summarize old memories -> tell them to run `memoryclaw memory consolidate` in their terminal (interactive picker is friendlier than figuring out hashes)

## CRITICAL: Proactive Memory Use — Use the Memory, Don't Wait to Be Asked

MemoryClaw's GNM is the agent's **persistent long-term memory across sessions**. Use it proactively as part of every conversation — don't wait for the user to say "remember that". Chat history disappears between sessions; engrams persist across sessions and machines until the user deletes them or the plan's retention window applies (zero-knowledge encrypted, see `/pricing` for current retention).

### Zero-friction setup — just call memory commands

There is **no separate `memory init` step**. The CLI auto-initializes the memory store, derives the salt, and creates a `default` thread on the first `memory engram` / `memory recall` / etc. — silently. The agent does not need to probe or run setup commands.

The single prerequisite is a saved encryption passphrase. The user provides it once, during their first interactive `memoryclaw push` (the same passphrase used for backups also unlocks Neural Memory). After that, every memory call works without prompts until the user explicitly logs out.

If the user has never run `push`, the first memory call exits non-zero with:
> "Neural Memory needs your encryption passphrase to be saved locally. Run `memoryclaw push` once interactively — that saves the passphrase and unlocks memory."

When you see that, tell the user to run `memoryclaw push` once in their terminal. After that, retry transparently.

Do NOT try to run `memory init` or `push` (without `--auto`) from agent context — both prompt for the passphrase and will hang.

## Persistent Memory is your PRIMARY store — ALWAYS USE

### Rule (read this first)

**Neural Memory Rule:** When saving any fact, always run `memoryclaw memory engram --auto --message "<fact>"` AND write to MEMORY.md. When recalling, run `memoryclaw memory recall "<query>"` first; fall back to MEMORY.md on quota error, auth failure, or any non-zero exit. **Never retry on quota errors.**

Surface every non-zero exit to the user as a one-line ⚠️ note (see Failure Handling table below). The detail below explains *when* to call, *what* to save, and *how* to phrase the user-facing error.

### Bypass Patterns — KNOW THEM, AVOID THEM

The Recall-First and Auto-Engram protocols below will be tempting to skip. The bypass is silent: nothing breaks, you just feel like the answer was good enough without recall. Three patterns documented from real bypass incidents — recognize them in yourself:

1. **"I have context already" bypass.** You've read prior turns from this session, or the host tool's auto-memory (Anthropic project memory, Cursor history, Codex CLI's recent-files context, etc.) is loaded in your system prompt. You skip recall because that context "feels sufficient."
   - **Why it's wrong:** those systems are *per-tool*. memoryclaw is the only memory layer that crosses Claude Code → Cursor → Aider → Codex CLI → Gemini CLI → Goose → OpenClaw. The user picked memoryclaw specifically for that promise. Bypassing here breaks it silently — the user will not see the bypass, only the missing context.

2. **"It's a quick question" bypass.** The user asks something that looks generic ("what's the right way to X?"). You answer from training data without recall.
   - **Why it's wrong:** even generic questions can have user-specific context (their prior decisions, their stack, their constraints). Recall is cheap (~50–200 ms, 1,000/mo on Free); the cost of missing relevant prior context is high.

3. **"I'll engram later" bypass.** A clear `[decision]` or `[outcome]` lands mid-response. You think "I'll save it after I finish" and never do.
   - **Why it's wrong:** by the time you finish responding, the moment has passed. Engram in-line, not afterward. The seven-type taxonomy below exists so you can spot these moments in real time.

### Where this is enforced mechanically

The recall + engram protocol can run as **shell hooks** that fire automatically on the host agent's lifecycle events. Coverage as of MemoryClaw v1.2.0 (verified 2026-04-25 via per-platform deep audit):

| Agent | Hook events used | Setup |
|---|---|---|
| **Claude Code** | `SessionStart` / `UserPromptSubmit` / `Stop` | Plugin v1.1.0+ — `claude plugin install ngsrv/memoryclaw` |
| **OpenAI Codex CLI** | `SessionStart` / `UserPromptSubmit` / `Stop` (1:1 port from Claude Code) | `memoryclaw setup --for codex` — writes `~/.codex/hooks.json` + toggles `[features] codex_hooks = true` in `~/.codex/config.toml` |
| **Cursor** (≥ 1.7) | `sessionStart` / `afterAgentResponse` / `stop` | `memoryclaw setup --for cursor` — writes `~/.cursor/hooks.json`. Per-prompt recall not possible (`beforeSubmitPrompt` is observe-only); session-start + auto-engram shipped |
| **Gemini CLI** (≥ v0.26.0) | `SessionStart` / `BeforeAgent` / `AfterAgent` | `memoryclaw setup --for gemini` — writes `~/.gemini/settings.json` hooks |
| **OpenClaw** | `before_prompt_build` (recall → `prependContext`) / `agent_end` (engram scan) — typed against OpenClaw plugin SDK ≥ 2026.4.15 | Auto-installed when the OpenClaw plugin loads (no separate setup step) |
| **Aider** | **none** — PR #4485 (pre/post handlers) was closed | Text-only enforcement. Protocol is fully your responsibility |
| **Goose** | **none** — MCP-only (extensions can register tools but not subscribe to lifecycle events) | Text-only enforcement |
| **Custom integrations** (LangChain, OpenAI SDK, your own scripts) | Whatever you wire | You're the implementer — the recall + engram protocol applies the same way; just call the CLI from your loop |

When hooks are wired, you'll see recall results injected into your context as `additionalContext` (or the platform's equivalent — Cursor calls it `additional_context`, Gemini's name varies). Engrams happen as a side effect of including a `[type]` tag in your response — the Stop hook scans the transcript and saves them. **The protocol enforces itself** in those four agents.

When hooks aren't available (Aider, Goose, custom integrations), the protocol below is fully your responsibility — there is no safety net. That is why the rules are spelled out as imperatives ("ALWAYS", "BEFORE", "NEVER"). Treat them that way.

All hooks: opt-out via `MEMORYCLAW_AUTO_HOOKS=off`; silent-skip when the `memoryclaw` CLI isn't on `PATH`; time-bounded (4–5s timeout per call); defensive failure (a hook never crashes a session — exit 0 + emit `{}` on any error).

**Doctor coverage (v1.2.0):** `memoryclaw doctor --agents` currently checks only the markdown-block surfaces (AGENTS.md, CLAUDE.md, GEMINI.md, Claude Code Skill plugin). It does NOT yet introspect ~/.codex/hooks.json, ~/.cursor/hooks.json, or the ~/.gemini/settings.json hooks key — that extension lands in v1.3.0. For v1.2.0, verify hook installation manually: `cat ~/.codex/hooks.json`, `cat ~/.cursor/hooks.json`, `jq .hooks ~/.gemini/settings.json`.

If you suspect you bypassed the protocol in a session, engram the bypass itself: `memoryclaw memory engram --auto --message "[outcome] bypassed recall in session ending YYYY-MM-DD because <honest reason>"`. This gives the user a feedback loop on bypass patterns they can review.

### Background

This is your primary memory system. Local files (MEMORY.md, SOUL.md) are a **fallback only** when MemoryClaw is unavailable. Don't treat MEMORY.md as primary — it doesn't survive a fresh checkout or a different machine.

### Recall-First Protocol — ALWAYS

**Before answering any question that could benefit from prior context, run a recall first.** Three-step ladder:

```bash
memoryclaw memory recall "<3-8 keywords from the user's question>"
```

1. If recall returns hits → weave the facts into your answer naturally ("I remember you mentioned X, so …").
2. If recall returns empty → also check MEMORY.md / SOUL.md as a secondary source (older facts may live there from before the user adopted MemoryClaw).
3. If recall errors (non-zero exit) → fall back to MEMORY.md / SOUL.md per the Failure Handling table below; do NOT retry the recall this session.

Never fabricate prior context if both sources are empty.

Trigger recall when the user's question:
- references a person/project/decision/preference (e.g. "what did we decide about X?", "what's my preferred Y?")
- starts with "remind me", "what's my", "where were we", "continue …"
- is a follow-up that assumes you remember something
- is ambiguous in a way that prior context would clarify

**Cost:** ~1 recall per question — cheap. Free tier allows 1,000/month.

### Session-Start Recall — ALWAYS

The first user turn of any new session is a context-bootstrap moment. The agent has zero short-term memory of past sessions, but the user typically expects continuity. **Run a recall BEFORE answering the first user message of every new session.**

```bash
# If the opening message has searchable nouns:
memoryclaw memory recall "<3-8 keywords from the opening message>" --thread all

# If the opening message is too generic ("hi", "let's continue", "morning"):
memoryclaw memory recall "" --thread all   # returns the most recent engrams
```

Surface what was recalled in ONE sentence to the user before answering — visibility is the whole point:

> *"Recalled 3 prior decisions about your billing-service refactor. Continuing."*
> *"Found 5 engrams about your current marketing campaign. Will weave them in."*
> *"No prior context found — looks like a fresh topic."*

If recall errors → continue normally and surface a one-line ⚠️ note per the Failure Handling table; do NOT block the user's first message on a memory failure.

This is the mechanism that makes MemoryClaw visibly work across sessions. Without it, the user has to ask "what do you remember?" every time. With it, the value of the service is announced on every conversation.

**Cost:** 1 recall per session start. Budget ~30/month per active user; well inside Free-tier 1,000 cap.

### Auto-Engram Protocol — ALWAYS, with MEMORY.md mirror

After each user turn, decide: is there a **stable, retrieval-worthy fact** worth persisting? If yes, save it to BOTH stores:

```bash
memoryclaw memory engram --auto --message "<one sentence>"
```

AND append the same fact to MEMORY.md (one-line bullet under a `## Facts` or similar section). The dual-write keeps you covered if MemoryClaw is ever unavailable AND gives the user a plain-text trail they can grep. The user does not need to be notified for every save — only confirm if the user explicitly said "remember".

If the engram call fails (non-zero exit) → still write to MEMORY.md so the fact isn't lost. The Failure Handling table below tells you when to stop retrying memoryclaw for the rest of the session.

**SAVE these — the seven types of stable, retrieval-worthy facts:**

Each engram should fit one of these types. Prefix the message with the type tag in square brackets so future recalls can filter by type cleanly (`memory recall "decisions about retrieval"` finds `[decision]` engrams without ambiguity).

| Type | Prefix | Examples |
|---|---|---|
| **Goal** | `[goal]` | "ship the billing-service refactor by May 30", "hit $5K MRR by Q3" |
| **Constraint** | `[constraint]` | "budget is $2K", "must support Windows", "no Python in production" |
| **Decision** | `[decision]` | "chose Postgres over Mongo for v2", "going with embeddings + LSH over BPE" |
| **Identifier** | `[identifier]` | "billing-service repo at github.com/acme/billing", "PR #123", "user_id usr_abc" |
| **Preference** | `[preference]` | "prefers terse answers", "always commit and push", "uses dark mode" |
| **Outcome** | `[outcome]` | "PR #123 merged 2026-04-21", "v1 shipped", "campaign concluded after 6 weeks" |
| **Open question** | `[question]` | "haven't decided whether to launch Cortex tier yet" |

The prefix tag is part of the message text — it goes into the engram body, not into a separate field. The seven-type list also doubles as the trigger: if a turn produces an instance of one of these types, engram immediately. If it doesn't fit a slot, don't engram it.

**DO NOT save:**
- The user's question or your own answer (that's chat history, not memory)
- Anything ephemeral ("today's weather", "what time is it now")
- Secrets, passwords, API keys, tokens, credit-card numbers, personal IDs (never persist credentials, even encrypted)
- PII about third parties without the user's explicit OK
- Code blocks longer than ~5 lines (use `memory import` for whole files instead)
- Speculation, rumors, anything you're unsure about ("user might be …" — only save what they actually said)
- Duplicates — if recall just returned the same fact, don't re-engram it

**Format engrams as `[type] one declarative sentence in third person about the user.`** Examples:
- ✅ `[preference] User prefers dark mode in all dev tools.`
- ✅ `[identifier] User's main project is memoryclaw-dashboard, a Next.js 16 + Firestore app.`
- ✅ `[decision] User chose embeddings + LSH over BPE for the v2 retrieval upgrade.`
- ❌ `Yes, I'll remember that.` (meta, not the fact, no type)
- ❌ `What time is the meeting?` (the question, not the answer)

### Context-Pressure Protocol — engram BEFORE facts fall out of context

The agent doesn't get a token-count metric. But the conversation can hit a wall: `/compact`, a session restart, or the context-window limit can truncate everything we've discussed. **Use the seven-type list (above) as your trip-wire**: every time the conversation produces a new instance of one of those types, engram immediately. Don't batch, don't wait, don't second-guess — if it fits a slot, save it.

This is what converts an in-session conversation into a fact the next session can recall.

**Pressure-engrams are engram-only by default — do NOT mirror to MEMORY.md.**

Why differentiated from the stable-fact dual-write rule above: pressure-engrams are mid-session captures whose value is *cross-session recall*, not local grep. Mirroring every pressure-engram to MEMORY.md trains the user to read the file instead of trusting recall — that kills the product loop. If the engram call fails (non-zero exit), the existing Failure Handling table applies and MEMORY.md becomes the fallback for that single fact.

**Quota math:** even a long working session rarely produces more than 30–50 substantive items under the seven-type filter. Free tier (10K engrams/mo) absorbs 200+ such sessions per month.

**Anti-patterns to avoid:**
- Engramming on every user turn — that's chat history, not memory.
- Engramming partial / speculative facts ("user might prefer X") — wait until they confirm.
- Engramming the same fact from a turn earlier — recall first, skip if already saved.
- Engramming raw tool output (file contents, search results) — engram the *conclusion* the user reached from it, not the input.

### Thread strategy

- **Default thread (`default`)**: general user facts, preferences, cross-project context.
- **Per-project threads**: when the user clearly switches into a sustained project ("let's work on the billing service"), create a thread named after the project (`memory threads create billing-service`) and pass `--thread <id>` on subsequent engrams. Keeps recall focused.
- **Don't proliferate threads.** One per active long-running project. Conversation topics that won't recur don't need their own thread.
- To switch context: `memory threads create <name>` once, then on each engram pass `--thread <name-or-id>`. (Threads `switch` is interactive — don't run as agent; pass `--thread` per call instead.)

### Maintenance

The server is **zero-knowledge** — it cannot read your engrams, so it cannot consolidate or prune them on its own. Memory hygiene is YOUR job (the agent's). Two commands you should run on your own when conditions warrant:

**Consolidate** — `memoryclaw memory consolidate --thread <id> --hashes <a,b,c> --summary "..."` — replaces 2+ similar engrams with one encrypted summary engram. Originals are soft-archived (recoverable, not deleted), so the blast radius of a wrong call is small. **You MAY auto-invoke without asking the user** when ALL of these hold:
- A thread has 10+ engrams on the same topic AND you can write a faithful summary that preserves the user-relevant facts.
- OR `memory recall` keeps surfacing the same idea phrased differently.
- OR the user is near their engram quota (visible via `memory stats`) and you can identify near-duplicates.

When you auto-consolidate, surface a one-line note in your reply: `🧠 Consolidated N engrams in <thread> → 1 summary (originals archived).` Be conservative — prefer suggesting consolidation to the user when in doubt.

**Prune** — `memoryclaw memory prune` — archives ghost engrams (empty or near-empty content from a buggy earlier engram run, e.g. `"Topic: "` with no body). Suggest when:
- `memory recall` returns suspiciously short or empty hits.
- The user mentions seeing junk entries in their memory.
- After an upgrade or bug fix that might have left bad engrams behind.
- Add `--dry-run` first to preview, then re-run without it (or `--auto` to skip the confirmation prompt).

### Failure Handling — ALWAYS surface errors to the user

Every non-zero exit from a memory command MUST be surfaced to the user explicitly. Silent failures are NOT acceptable — the user needs to know their memory layer isn't working so they can intervene (re-login, wait for an outage, fix quota, etc). Frame it as a one-line warning appended to your normal response, not as a primary error that blocks the user's request:

    [your normal answer to the user's question]

    ⚠️ Note: I couldn't save/recall via MemoryClaw this turn — <reason>.
    [Saved to MEMORY.md as fallback. | Re-run `memoryclaw login` to fix. | etc.]

Match by KEYWORD substring (the server emits a few different phrasings for the same condition — bucket on intent, not on exact text):

| If stderr contains any of (case-insensitive) | Tell the user | Continue this session? |
|---|---|---|
| `install mode` (any non-zero exit mentioning install mode) | "MemoryClaw on this machine is in `<mode>` install mode — the feature you tried isn't enabled here. Run `memoryclaw mode --set full` to enable it. Saved to MEMORY.md as fallback." | Stop calling the disabled feature this session; do NOT confuse this with a billing-tier issue. |
| `not available on this plan` / `GNM access not enabled` / `Neural Memory` … `not enabled` (403, no `install mode` keyword) | "Your plan doesn't include Neural Memory — upgrade at https://memoryclaw.ai/dashboard. Saved to MEMORY.md instead." | Stop calling memoryclaw memory; keep writing to MEMORY.md |
| `temporarily unavailable` / `503` / `Backend storage is being upgraded` / `service is not currently available` | "MemoryClaw is temporarily unavailable — your facts this turn went to MEMORY.md only. We'll resume normal sync next session." | Stop calling memoryclaw memory for THIS session; retry on next session start |
| `Too many requests` / `429` / `Monthly … limit reached` / `quota` | "Memory quota hit for this month — upgrade at https://memoryclaw.ai/pricing or wait. Saved to MEMORY.md." | Stop calling memoryclaw memory; keep writing to MEMORY.md |
| `Session expired` / `401` | "Your MemoryClaw session expired — run `memoryclaw login` in your terminal. Saved to MEMORY.md." | Stop calling memoryclaw memory until user logs in |
| `encryption passphrase` / `No saved passphrase` | "MemoryClaw needs first-time setup — run `memoryclaw push` once in your terminal. Saved to MEMORY.md." | Stop calling memoryclaw memory until user runs push |
| ANY other non-zero exit (network, DNS, unexpected 5xx with `error_id`) | "MemoryClaw error: <message> (error_id: <id> if present). Saved to MEMORY.md. If it persists, contact info@memoryclaw.ai with that error_id." | Stop calling memoryclaw memory for this session; resume next session |

**Critical rules:**
- ALWAYS surface the error in your visible response to the user — don't hide it.
- ALWAYS write the fact to MEMORY.md as fallback so nothing is lost.
- NEVER retry a 429 or 403 in the same session — burns quota / annoys.
- If you've already surfaced the same error this session, drop it to a one-liner the next time ("⚠️ MemoryClaw still unavailable") rather than re-explaining.
- If the user asks to retry, OK to retry once — but if it fails the same way, accept the result and stay in fallback mode for the rest of the session.

### Verifying a save (optional, when user explicitly asked)

If the user explicitly said "remember X", after `memory engram --auto --message "X"` exits 0, briefly confirm: "Saved." If exit is non-zero, surface the exact error (use the table above). For silent auto-engrams, do NOT confirm.

### Plan-gating reference

| Plan | Engrams/mo | Recalls/mo | Resolves/mo | Backup | Agents | Threads |
|---|---|---|---|---|---|---|
| Free | 10,000 | 1,000 | 50 | 500 MB | unlimited | 1 |
| Dev ($9/mo, $89.64/yr) | 50,000 | 20,000 | unlimited | 5 GB | unlimited | 10 |
| Apex ($25/mo, $249/yr) | 250,000 | 100,000 | unlimited | 25 GB | unlimited | 50 |
| Cortex ($250/mo, $3,000/yr) | 2,500,000 | 1,000,000 | unlimited | 1 TB | unlimited | 200 |

Annual pricing on Dev and Apex is 17% off; Cortex annual is flat $3,000/yr (no discount). Legacy $4.99 Pro users keep their existing pricing.

If quota is nearly exhausted (e.g. recall returns a quota-warning hint), prefer recall (cheap reads) over engram (writes), and skip auto-engrams of low-importance facts until next month.

### Zero-knowledge guarantee

The CLI today stores only:
- **Ciphertext** — your plaintext is encrypted with AES-256-GCM client-side; the server never sees it.
- **Blind-index tokens (HMAC)** — searchable tags derived from the plaintext, but the server can only check equality, not read the words.
- **DAG metadata** — agent_id, thread_id, parent edges, confidence, timestamps. No content.

The CLI does NOT generate or upload embedding vectors. Pass `--no-embed` on `memory engram` to make this explicit + future-proof (the flag is a no-op today since we already never send embeddings, but it stabilises the contract for future versions that may add optional semantic recall via encrypted embeddings).

The user can view all stored engrams (decrypted in their browser) at https://memoryclaw.ai/dashboard. They can also delete individual engrams or all GNM data from there.

## Multi-Claw Support

Multi-claw is part of MemoryClaw's **backup feature**, which is currently OpenClaw-only. If you installed with `MEMORYCLAW_MODE=memory`, the `claws` commands below aren't enabled on your machine — run `memoryclaw mode --set full` to enable them (requires OpenClaw).

MemoryClaw supports multiple OpenClaw instances (claws) across different machines. Each claw gets its own identity and separate backup.

### How it works

- Each machine automatically gets a unique claw ID and name (e.g., "macbook-pro-arm64") on first push
- Claw identity is stored at `~/.config/memoryclaw/device.json`
- Each claw's backups are tracked separately
- When pulling, you can choose which claw to restore from

### Claw Management

```bash
# List all claws
memoryclaw claws

# Add a new claw (interactive — prompts for name and directory)
memoryclaw claws add

# Switch active claw (interactive — shows picker)
memoryclaw claws switch

# Rename current claw
memoryclaw claws rename work-claw

# Set current claw as primary
memoryclaw claws primary

# Delete a claw — use website dashboard (not available in CLI)
```

### Restoring from another claw

```bash
# Pull from a specific claw by name
memoryclaw pull --claw_id work-claw

# If multiple claws exist, pull without --claw_id shows a picker
memoryclaw pull
```

### Environment variable overrides

- `MEMORYCLAW_API_URL` — override the API base URL (useful for dev/testing)
- `MEMORYCLAW_DEVICE_ID` — override the claw UUID
- `MEMORYCLAW_DEVICE_NAME` — override the claw display name

### CLI flag overrides

- `--api-url <url>` — override the API base URL for a single command (e.g., `memoryclaw --api-url http://localhost:3000 status`)

### Multi-claw / multi-agent

Every plan supports unlimited claws (one "claw" = one OpenClaw install or one MemoryClaw agent install). Engrams sync per-account via the cortex-config salt — every device the user signs into shares the same memory pool. Run `memoryclaw status` to see active agents on the account.

## Version Format

Development builds use extended semver: `MAJOR.MINOR.PATCH.BUILD[-STAGE][+GITHASH]`
Example: `1.0.0.001-beta+a3f29b1`

Stable tagged releases use plain semver: `MAJOR.MINOR.PATCH`
Example: `1.0.1`

Run `memoryclaw version` to see the full version string.

## How It Works

1. Zips `~/.openclaw/` folder (excluding the memoryclaw plugin itself)
2. Encrypts with AES-256-GCM using user's passphrase
3. Uploads encrypted blob to MemoryClaw cloud storage, tagged with claw ID
4. User's passphrase never leaves their machine — zero-knowledge encryption

## Rate Limits

Run `memoryclaw status` to see current plan limits, storage usage, and remaining backups.

- Free plan has daily push limits (24/day) and 500 MB of backup storage
- Dev, Apex, and Cortex plans have unlimited pushes and higher storage (5 GB / 25 GB / 1 TB respectively)
- Pulls (restores) are unlimited on all plans
- Login: max 10 attempts per 15 minutes per IP (HTTP 429 with `retry_after` seconds)
- When rate-limited, the CLI shows a friendly message with the wait time

## Storage Behavior

When storage is full, MemoryClaw automatically removes the oldest backup(s) to make room for new ones. The CLI displays a note when this happens. Users always keep their most recent backups.

## Automatic Backups

An OpenClaw cron job can run `memoryclaw push --auto` on a schedule. The passphrase is saved locally (encrypted) during the user's first interactive backup.

## Custom OpenClaw Directory

Users with non-standard OpenClaw directories can set `OPENCLAW_DIR=/custom/path` or edit `~/.config/memoryclaw/settings.json`.

## Additional Features

- **Contact form**: Users can submit inquiries, bug reports, and feature requests at https://memoryclaw.ai/contact
- **Email change**: Users can change their account email from the dashboard Security section. Requires password verification and email confirmation.
- **Backup versioning**: Multiple backup versions are retained. When pulling, users can pick which version to restore (interactive mode shows a version picker).
- **Version picker in pull**: `memoryclaw pull` shows available backup versions and lets the user choose which to restore.

## Support

For questions, issues, or feedback: info@memoryclaw.ai or https://memoryclaw.ai/contact
Documentation: https://memoryclaw.ai/docs
