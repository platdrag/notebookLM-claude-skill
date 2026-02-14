---
description: Interact with your NotebookLM notebooks — ask domain-specific questions, explore sources, and have multi-turn conversations grounded in your uploaded documents.
argument-hint: <question> [--notebook <name>]
disable-model-invocation: true
---

# NotebookLM Query

You are using NotebookLM as a knowledge base. Your job is to query the user's NotebookLM notebook for domain-specific knowledge and return grounded, cited answers.

## Prerequisites

The `notebooklm-py` CLI must be installed and authenticated. If any step below fails with a missing command or auth error, tell the user to run `/notebooklm:setup` first.

## Execution Steps

<steps CRITICAL="TRUE">

### Step 1: Parse the Request

The user's input is: `$ARGUMENTS`

Extract:
- **query**: The domain question to ask (everything before `--notebook` flag, or the entire input if no flag)
- **notebook_name**: Optional. Value after `--notebook` flag. Defaults to `null`.

If `$ARGUMENTS` is empty or missing, ask the user what they want to know.

### Step 2: Verify Authentication

Run this command to check auth status before doing anything else:
```bash
notebooklm auth check --json
```

- If exit code is 0 and output shows valid auth, proceed to Step 3.
- If the `notebooklm` command is not found, tell the user to run `/notebooklm:setup`.
- If auth check fails, tell the user:
  > NotebookLM authentication has expired. Run `notebooklm login` in your terminal to re-authenticate (requires a browser).

  Then stop — do not proceed until auth is resolved.

### Step 3: Determine Target Notebook

**If `--notebook <name>` was provided:**

Read `{project-root}/.claude/notebooklm-config.json` and look up the notebook name in the `notebooks` map to get its ID. If found and the ID is valid (not `"PASTE_NOTEBOOK_ID_HERE"` or `null`), use that ID and skip to Step 4.

If not found in config, fall through to the interactive selection below.

**Otherwise (no `--notebook` flag, or name not in config):**

Read `{project-root}/.claude/notebooklm-config.json` to check if a `defaultNotebook` is configured. If a valid default exists, use that notebook's ID and skip to Step 4.

If no default is configured, list available notebooks and let the user choose:

```bash
notebooklm list --json
```

Present the notebooks to the user as a numbered list showing title and ID. Also suggest:
> If none of these notebooks contain the knowledge you need, you can create a new one at https://notebooklm.google.com or via `notebooklm create "Notebook Title"`, then add sources to it.

Wait for the user to select a notebook before proceeding.

After the user selects, offer to save it to the config for future use:
> Would you like me to save this notebook to `.claude/notebooklm-config.json` so it's used by default next time?

If yes, update the config file with the selected notebook's name and ID.

### Step 4: Query the Notebook

Run the query using the `notebooklm ask` command with JSON output for structured parsing:

```bash
notebooklm ask "<query>" -n <notebook_id> --json --new
```

**Important flags:**
- `-n <notebook_id>`: Explicitly target the notebook (parallel-safe, avoids context file race conditions)
- `--json`: Get structured output with answer text, citations, and source references
- `--new`: Start a fresh conversation (avoids context bleed from previous queries)

**On Windows**, if you encounter Unicode errors, prefix with `PYTHONUTF8=1`.

### Step 5: Parse and Present Results

The JSON output structure is:
```json
{
  "answer": "The answer text with [1] [2] inline citations...",
  "conversation_id": "...",
  "turn_number": 1,
  "references": [
    {
      "source_id": "...",
      "citation_number": 1,
      "cited_text": "Relevant passage from the source..."
    }
  ]
}
```

Present the results to the user as follows:

1. **Answer**: Show the answer text clearly. Keep inline citation markers `[1]`, `[2]` etc.
2. **Sources**: List each citation with its number and cited text passage:
   ```
   Sources:
   [1] "cited text passage..."
   [2] "cited text passage..."
   ```
3. **Confidence note**: If the answer contains hedging language ("I don't have information about...", "Based on limited sources..."), flag this to the user so they know the notebook may not cover this topic.

### Step 6: Handle Errors

| Error | Action |
|-------|--------|
| Command not found (`notebooklm`) | Tell user to run `/notebooklm:setup` |
| Auth expired / 401 | Tell user to run `notebooklm login` in terminal |
| Notebook not found | Run `notebooklm list --json` and show available notebooks |
| Empty/null answer | Report that the notebook sources don't contain relevant information |
| Rate limited | Tell user to wait 1-2 minutes and retry |
| Timeout | Retry once; if still failing, report the issue |

### Step 7: Follow-up (Optional)

If the answer is insufficient or the user wants to dig deeper, you can run a follow-up query using the conversation_id from the previous response:

```bash
notebooklm ask "<follow-up question>" -n <notebook_id> -c <conversation_id> --json
```

This maintains conversation context for multi-turn exploration.

</steps>

## Usage Examples

```
/notebooklm:query What is the architecture of the evaluation pipeline?
/notebooklm:query How are scenarios structured in the domain model? --notebook my-project
/notebooklm:query What design decisions were made for the data layer?
```
