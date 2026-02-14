---
description: Install and configure the notebooklm-py CLI for querying NotebookLM notebooks. Handles installation, authentication, notebook discovery, and project config file setup.
disable-model-invocation: true
---

# NotebookLM Setup

This skill guides the user through installing and configuring the `notebooklm-py` CLI tool so that `/notebooklm:query` can query their NotebookLM notebooks.

<steps CRITICAL="TRUE">

### Step 1: Check if notebooklm-py is Already Installed

```bash
notebooklm --version
```

- If the command succeeds, report the version and skip to Step 3.
- If the command is not found, proceed to Step 2.

### Step 2: Install notebooklm-py

Ask the user which installation method they prefer:

**Option A — Lightweight (recommended if you can export browser cookies manually):**
```bash
pip install notebooklm-py
```
This installs the CLI and all query functionality. Authentication must be handled by manually exporting cookies from an existing browser session (see Step 4, Option B).

**Option B — With browser-based login:**
```bash
pip install "notebooklm-py[browser]"
playwright install chromium
```
This adds Playwright, which enables the `notebooklm login` command. It opens a dedicated Chromium instance to capture Google session cookies — regular browsers don't expose cookies to external programs, so Playwright's own browser is needed for automated cookie extraction.

If `pip` fails, try `pip3` or suggest the user activate the appropriate Python virtual environment. `pipx install notebooklm-py` is also an option for global CLI tools.

Verify installation:
```bash
notebooklm --version
```

### Step 3: Check Authentication Status

```bash
notebooklm auth check --json
```

- If auth is valid, report the status and skip to Step 5.
- If auth is invalid or missing, proceed to Step 4.

### Step 4: Authenticate

Offer two authentication methods:

**Option A — Browser login (requires `[browser]` extra and Playwright):**

Tell the user:
> Please run this command in your terminal (not here — it opens a browser window):
> ```
> notebooklm login
> ```
> 1. A Chromium window will open to the NotebookLM page
> 2. Log in with your Google account
> 3. Once the page loads, switch back to your terminal and press ENTER
>
> Come back here when you're done.

**Option B — Manual cookie export (no Playwright needed):**

Tell the user:
> You can export cookies from a browser where you're already logged into NotebookLM:
>
> 1. Open https://notebooklm.google.com in your browser (make sure you're logged in)
> 2. Use a cookie export extension (e.g., "EditThisCookie", "Cookie-Editor", or "Get cookies.txt")
> 3. Export all cookies for the `.google.com` domain as JSON
> 4. Create the file `~/.notebooklm/storage_state.json` with this structure:
> ```json
> {
>   "cookies": [ ...exported cookies array... ],
>   "origins": []
> }
> ```
>
> The critical cookies are: `SID`, `HSID`, `SSID`, `APISID`, `SAPISID`, `__Secure-1PSID`, `__Secure-3PSID`

Wait for the user to confirm they've completed authentication.

Then verify:
```bash
notebooklm auth check --json
```

If still failing, suggest re-trying or switching to the other authentication method.

### Step 5: Discover Notebooks

List the user's notebooks:

```bash
notebooklm list --json
```

Present the results as a readable list:
```
Your NotebookLM notebooks:
1. "Notebook Title" (ID: abc123...)
2. "Another Notebook" (ID: def456...)
...
```

If no notebooks exist, suggest:
> You don't have any notebooks yet. You can create one:
> - **Web**: Go to https://notebooklm.google.com and click "New Notebook"
> - **CLI**: `notebooklm create "My Knowledge Base"` then add sources with `notebooklm source add`

### Step 6: Configure the Project Config File

Read the existing config at `{project-root}/.claude/notebooklm-config.json` if it exists.

Ask the user which notebook(s) they want to configure for this project. For each selected notebook, ask for a short name (key) to use with the `--notebook` flag.

Ask the user which notebook should be the default (used when no `--notebook` flag is specified with `/notebooklm:query`).

Write/update the config file at `{project-root}/.claude/notebooklm-config.json`:

```json
{
  "defaultNotebook": "<chosen-default-name>",
  "notebooks": {
    "<name>": {
      "id": "<actual-notebook-id>",
      "description": "<user-provided description of what this notebook contains>"
    }
  },
  "settings": {
    "mode": "concise",
    "responseLength": "default",
    "persona": null
  }
}
```

**Settings reference:**
- `mode`: One of `default`, `learning-guide`, `concise`, `detailed`
- `responseLength`: One of `default`, `longer`, `shorter`
- `persona`: Custom instruction string (e.g., "Act as a senior architect") or `null` for default

### Step 7: Verify End-to-End

Run a quick test query against the configured default notebook:

```bash
notebooklm ask "Summarize the main topics covered in this notebook" -n <notebook_id> --json --new
```

If successful, show the user a summary of the response and confirm:
> Setup complete. You can now use `/notebooklm:query <question>` to query your notebook.

If it fails, diagnose based on the error (auth, notebook not found, no sources, etc.) and guide the user to fix it.

</steps>

## Post-Setup

After setup is complete, remind the user:
- `/notebooklm:query <question>` — Query the default notebook
- `/notebooklm:query <question> --notebook <name>` — Query a specific notebook
- Auth sessions expire every few weeks — re-run `notebooklm login` or re-export cookies when needed
- To add more sources: `notebooklm source add "<url-or-file>" -n <notebook_id>`
