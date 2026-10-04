# Rovo CLI troubleshooting

Load this through the router's `troubleshoot` situation for Rovo, beside `references/harness/rovo.md`, which owns the operating facts.

## Composer ghost text: a known, unfixed gap

rovo's empty composer renders an inline placeholder chip (e.g. `Summarize my open tasks`) directly inside the bordered content row, not merely as a separate suggestion list below it.
Measured live, that placeholder's foreground is `38;2;162;163;165` (luminance ~163), while real typed text in the same box is `38;2;206;207;210` (luminance ~207) - a real gap, but one that sits entirely above `../../../bin/fm-composer-lib.sh`'s default `FM_COMPOSER_GHOST_LUMA_MAX` of 128, so `fm_composer_strip_ghost` does not strip it and a fresh rovo composer can misclassify as `pending` instead of `empty`.
Raising the shared default to catch it is not safe: muse's own real, must-not-be-stripped prompt glyph measures luminance ~149.9, below rovo's ghost luminance, so no single global threshold can keep muse's real glyph while dropping rovo's ghost chip.
This is deliberately left unfixed rather than patched with a threshold change that would risk muse's already-verified behavior; a real fix needs a harness-scoped signal the shared composer classifier does not currently carry.
The practical consequence is bounded to composer-emptiness consumers - steering into an idle rovo pane may see a non-empty verdict and retry through the normal doorbell ladder rather than deliver on the first try.
It does not block the launch-then-send gates: readiness leads with the `Welcome to Rovo!` banner (not composer-empty), and while the delivery gate does require composer-empty as one conjunct, it runs while rovo is actively processing the just-delivered brief - the placeholder chip renders only at idle rest, not mid-turn - so the composer reads genuinely empty during the delivery window.

## Interrupt: confirmed under real tmux

The original verification scout (`fm-rovo-smoke-s1`, PTY smoke) observed a single Escape print `Agent cancelled` during a running tool call.
A follow-up live check under real tmux 3.6a - an isolated `tmux -L <private-socket>` session/window, not the shared fleet session - reproduced the scout's exact finding: a single Escape sent during a genuine mid-flight bash tool call printed `Agent cancelled` in the captured pane.
The launch-then-send live guard (`../../../../tests/fm-rovo-signals-live-e2e.test.sh`) now reproduces it over a raw PTY too: an earlier single fixed-timer Escape landed unreliably (the interrupt instant is timing-sensitive over a bare PTY), so the guard sends Escape across the live tool-call window until the cancel renders - a deterministic way to reproduce a timing-sensitive interrupt, and confirmed to print `Agent cancelled` every run.
Escape is the interrupt key and is what `fm_control_interrupt_key` returns.
`fm_control_interrupt_ack_source` still records `none` for rovo - the same conservative choice already made for claude/codex/grok/kimi/cursor, a control-plane fact independent of whether the render happens to appear - so the control plane sends the key and lets its own postcondition, not a parsed string, decide whether the agent actually stopped.
The interrupt key and its rendered evidence are now fully corroborated rather than in tension with the code.

## OAuth token lifetime

The access token lasts about one hour, but `rovo` refreshes it silently and non-interactively from a stored refresh token (about four weeks' lifetime) with no browser prompt and no visible interruption - this is standing captain-corrected guidance, not this task's own discovery, and this task's own live checks corroborated it empirically: `rovo auth status` showed `Access token expired ... but a refresh token is present`, then a plain `rovo run` completed successfully and a follow-up `rovo auth status` showed a freshly valid token with no interactive step in between.
Treat the ~1h access-token lifetime as an ordinary operational fact, not a non-negotiable-safety blocker: a rovo worker does not need to be scoped short to survive it.
`rovo auth login` (interactive browser OAuth) is needed only after roughly four weeks of disuse or if the refresh token itself is invalidated.

## ACP as a future upgrade

`rovo acp` (Agent Client Protocol) and `rovo serve --non-interactive` expose a fully structured, machine-readable turn lifecycle: `session/prompt` returns a real `{"stopReason":"end_turn"}`, and `session/cancel` is a protocol-native interrupt.
This is a cleaner done-signal than any current adapter has, but consuming it means firstmate runs a JSON-RPC client and owns the session lifecycle itself - a new backend-shaped surface, not a drop-in TUI adapter - so it is out of scope here.
It remains a deliberate future upgrade for a rovo-as-structured-backend follow-up, not a near-term path; do not build it as part of this TUI-path adapter.
