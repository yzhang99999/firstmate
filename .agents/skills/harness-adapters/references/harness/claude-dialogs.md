# Claude startup dialogs

Load this through the router's `dialog` situation for Claude, beside `references/harness/claude.md`, which owns the operating facts.

## Workspace trust

Claude gates a folder it has never seen behind an interactive workspace-trust dialog (titled "Quick safety check: Is this a project you created or one you trust?"), so every fresh task worktree would hit it, and so would every secondmate home no operator has opened by hand.
`--dangerously-skip-permissions` does not cover that gate: `claude --help` records that the dialog is skipped only in non-interactive mode, through `-p` or a non-TTY stdout, and a spawned pane is interactive.
Every claude spawn therefore pre-registers the directory its pane starts in before launch, and the dialog does not appear: the task worktree for a ship or scout, and the home itself for a `--secondmate` spawn, in either seeded shape (a leased worktree or a standalone clone).

A second, separate dialog - "Allow external CLAUDE.md file imports?" - renders whenever a loaded CLAUDE.md chain reaches outside the project tree, which every crewmate's does through the captain's own `~/.claude/CLAUDE.md` importing `~/.claude/RTK.md`.
`--setting-sources project,local` (the minimal worker tool surface) does not suppress it either, and it gates the pane exactly like the trust dialog: cursor on "No, disable external imports", no way to move the selection from firstmate's steering plane.

`../../../bin/fm-claude-trust.sh` records `hasTrustDialogAccepted` for both the worktree and its primary checkout in `${CLAUDE_CONFIG_DIR:-$HOME}/.claude.json`, where a home's worker account pin decides `CLAUDE_CONFIG_DIR` (`../../../docs/configuration.md` "Worker account pin"), for a ship or scout spawn; a secondmate spawn registers only its own home entry, since a secondmate home has no separate primary-checkout entry to carry import consent forward from.
For a ship or scout spawn, the external-imports flags (`hasClaudeMdExternalIncludesApproved`, `hasClaudeMdExternalIncludesWarningShown`) are carried forward alongside the trust flag only when the primary checkout's project entry already carries an explicit `hasClaudeMdExternalIncludesApproved===true` from a prior interactive session - the common first-spawn case is a project claude has never been asked about, so those two flags are left unwritten and the import dialog still renders, even though trust registers normally.
When the project entry instead already carries an explicit decline (`hasClaudeMdExternalIncludesApproved===false` with `hasClaudeMdExternalIncludesWarningShown===true`), the whole registration refuses - including the trust flag - rather than manufacture consent the human never gave, so that spawn wedges on the trust dialog before it would even reach the import one.
Both flags `false` is Claude Code's default entry for a project never asked, not a decline, and is treated like an absent flag: trust registers and the import dialog still renders.
The why-two-entries mechanism and the consent-gating logic live in the script's own header comment, which is the one owner for that contract; the fact worth repeating here is that `../../../bin/fm-spawn.sh` refuses the spawn when the trust flag fails to land, rather than launching a worker that would wedge on that dialog.

Never try to answer either dialog with a key.
Firstmate's key plane carries only Enter, Escape, and C-c with no arrow navigation, so it cannot move a dialog's selection at all, and both dialogs render with the cursor on their declining option, which means a sent Enter ends the session instead of accepting.
A visible trust dialog means pre-registration did not take effect (or the project entry already carries an explicit decline) - inspect the store and the spawn's error output rather than sending keys.
A visible external-imports dialog is expected, not a failure signal, whenever the project entry has no prior explicit approval on record - the common first-spawn case.
`fm-control.sh <id> interrupt` delivers Escape, which is the safe way to clear a wedged workspace-trust dialog for inspection without answering it.
Escape on the external-imports dialog is different: it records a permanent decline (`hasClaudeMdExternalIncludesApproved: false`, `hasClaudeMdExternalIncludesWarningShown: true`) that `../../../bin/fm-claude-trust.sh` then correctly refuses to override on every later spawn for that project.
Leave a pane showing the external-imports dialog alone and have a person answer it interactively instead of interrupting it.
To recover from an already-recorded decline, remove both flags from the project's entry in `~/.claude.json` and approve the imports dialog once by hand.

The once-per-machine bypass-permissions confirmation is a third, separate dialog, scoped to the machine rather than the path, and pre-registration does not address it.
Never send Enter to that one either: it was observed rendering in the same shape as the trust dialog, with the selection on `No, exit` and the footer `Enter to confirm . Esc to cancel`, so Enter ends the session rather than accepting.
Firstmate cannot move a selection with Enter, Escape, and C-c alone, so it cannot accept this dialog at all, and an operator accepts it once per machine instead.
Inspect the pane to identify which dialog is on screen, and report it rather than answering it.
A launch under `config/claude-permission-mode=auto` never meets the bypass confirmation, because it does not request bypass mode: on 2.1.269 `claude --permission-mode auto` reached the composer directly with the footer `⏵⏵ auto mode on (shift+tab to cycle)`, so a captain who refuses the bypass dialog selects `auto` there instead of accepting it.
The workspace-trust dialog is unaffected by the permission mode and still needs the pre-registration above.
