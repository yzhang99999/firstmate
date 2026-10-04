# Gemini CLI dialogs

Load this through the router's `dialog` situation for Gemini, beside `references/harness/gemini.md`, which owns the operating facts.

## Trust, and why the two documented options are not equivalent

Every task worktree is a path Gemini has never seen, so an unhandled launch refuses outright:
`Gemini CLI is not running in a trusted directory. To proceed, either use --skip-trust, set the GEMINI_CLI_TRUST_WORKSPACE=true environment variable, or trust this directory in interactive mode.`
Headless, that refusal exits 55.

The CLI presents those two options as equivalents and they are not.
A controlled A/B on one worktree - same config home, same prompt, only the trust mechanism changed - showed `--skip-trust` runs the turn while leaving PROJECT configuration unloaded, so the project's own hooks never fire and its `.agents/skills` are never discovered, while `GEMINI_CLI_TRUST_WORKSPACE=true` loads both.
A firstmate-repo task needs exactly those workspace skills, which is why the launch uses the environment variable.
Firstmate's OWN busy hooks do not depend on this, because they ride the system settings layer that `references/harness/gemini.md` describes under worker busy state.
Trusting the workspace loads that project's `.gemini/settings.json`, hooks, MCP servers, and skills, which is the same posture the other adapters already run under in a task worktree.

The interactive trust dialog is `Do you trust the files in this folder?` with three choices.
Unlike Claude's, its default selection is the SAFE one: `● 1. Trust folder (<name>)`, with `2. Trust parent folder (<parent>)` and `3. Don't trust` unselected.
Accepting persists to `~/.gemini/trustedFolders.json`, so the spawn's environment variable is preferred: it is per-session and leaves no growing global record of disposable worktree paths.

## Credential dialogs

A first run also shows an auth-method picker (`How would you like to authenticate for this project?`, default `● 2. Use Gemini API Key`); answering it once writes `security.auth.selectedType` to the user `settings.json` and it does not return.

With no credential the pane wedges on an `Enter Gemini API Key` dialog, and that dialog is dangerous in two distinct ways.
It RENDERS THE KEY IN PLAINTEXT in the pane once a value is present, where any capture or debug log would retain it, and the launch brief fails behind it with `API Error: Content generator not initialized`.
Worse, it is a credential field that accepts whatever is typed next: sending the ordinary exit command to a wedged pane submits `/quit` INTO it and persists it as a stored credential in `~/.gemini/gemini-credentials.json`.
That poisons the machine for every later run - a credential-less run then stops failing cleanly with exit 41 and instead reaches the API and fails per request with `API key not valid` - and it is repairable only by clearing that stored credential.
So never drive lifecycle text into a gemini pane that is showing this dialog.
Treat it as a credential blocker under `../../../../../AGENTS.md` section 9, fix the environment, and retire the endpoint rather than typing into it.
