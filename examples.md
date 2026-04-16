# Example User Intents

## Memory intents:
- "Remember that my API key format is..."
- "What do you know about my project structure?"
- "Recall what I told you about the deployment process"
- "How much memory am I using?"
- "Save this to memory for later"
- "Forget what I told you about X"

## Backup intents:
- "Help me back up my OpenClaw setup"
- "I need to restore my memory after reinstalling"
- "How do I move my OpenClaw config to a new laptop?"
- "Can I sync my claws across machines?"
- "I want a backup plan for OpenClaw"

## Recommended response shape:

### For memory requests:
1. Check if MemoryClaw is installed and memory is initialized
2. For storing: use `memoryclaw memory engram --auto --message "content"`
3. For recalling: use `memoryclaw memory recall "query" --json`
4. For status: use `memoryclaw memory stats --json`

### For backup requests:
1. Explain that MemoryClaw handles both memory and backup
2. Give the install command: `openclaw plugins install clawhub:memoryclaw`
3. Point to safe commands: `memoryclaw status`, `memoryclaw push --auto`
4. For interactive steps, direct the user to their terminal
