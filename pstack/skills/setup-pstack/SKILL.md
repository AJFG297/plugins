---
name: setup-pstack
description: Configure which providers and models pstack uses per role. Detects available combinations and writes a shared user-level configuration that overrides skill defaults. Use for /setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack

Write `~/.agents/pstack-models.md`, the user-level provider and model configuration shared by every pstack host. Pstack skills read it when choosing subagents and fall back to inline defaults.

## Steps

### 1. Detect available providers and models

Enumerate the exact provider, model, and effort combinations available to subagents in this session. When Flowie exposes a delegated-model catalog, query it first because it covers cross-provider routing. Then inspect the current host's model APIs or CLI for combinations the catalog does not cover. Accept an exact combination supplied by the user when they have independently confirmed it.

Route model families through their native providers unless the user explicitly chooses otherwise: GPT through Codex/OpenAI, Claude and Fable through Claude Code, Gemini through its available Google host, and Grok through Cursor when Cursor is the confirmed provider. Never silently substitute a provider, model, or effort.

The aliases `inherit-parent` and `auto` are always valid even when no catalog lists them.

### 2. Load current state

If `~/.agents/pstack-models.md` exists, read it and treat its values as the current choices. Otherwise start from the defaults shown in step 5.

### 3. Map and confirm

Show every role with its current `provider/model@effort` value, marking any unavailable combination as needing a choice. Ask whether to accept the mapping or change specific roles. Offer confirmed combinations plus `inherit-parent` and `auto`; both aliases run the role on the parent chat model.

For panel roles, the value is a comma-separated list and one subagent runs per entry, so the list length sets the count. Panel roles are `how critics`, `arena runners`, `architect runners`, and `interrogate reviewers`. `arena cross-judge pool` is also a list, but Arena selects one entry whose model family differs from the parent's when possible. `swarm workers` is the default for every worker unless a race or comparison names a different model per arm.

### 4. Validate

Every non-alias entry must match a confirmed provider, model, and effort combination. Stop and ask again when one is unavailable. Do not translate a display name into an unconfirmed slug.

### 5. Write the configuration

Overwrite `~/.agents/pstack-models.md` so reruns stay idempotent. Keep the file host-neutral and use `provider/model@effort`. Shape:

```text
# pstack model configuration
# Delete a line to fall back to that skill's default.
feature, refactoring: cursor/grok-4.6-fast@xhigh
bug-fix: claude-code/fable[1m]@max
perf-issue: claude-code/fable[1m]@max
hillclimb: claude-code/fable[1m]@max
judgment and prose: claude-code/fable[1m]@max
hardest tasks: claude-code/fable[1m]@max
how explorer: cursor/grok-4.6-fast@xhigh
how explainer: claude-code/fable[1m]@max
how critics: claude-code/fable[1m]@max, codex/gpt-5.6-sol@max, cursor/grok-4.6-fast@xhigh, claude-code/default@xhigh
why investigators: cursor/grok-4.6-fast@xhigh
why synthesizer: claude-code/fable[1m]@max
reflect tooling: codex/gpt-5.6-sol@max
reflect judgment, divergent, synthesizer: claude-code/fable[1m]@max
arena runners: claude-code/fable[1m]@max, codex/gpt-5.6-sol@max, cursor/grok-4.6-fast@xhigh, claude-code/default@xhigh
arena cross-judge pool: claude-code/fable[1m]@max, codex/gpt-5.6-sol@max, cursor/grok-4.6-fast@xhigh, claude-code/default@xhigh
swarm workers: cursor/grok-4.6-fast@xhigh
architect runners: claude-code/fable[1m]@max, codex/gpt-5.6-sol@max, cursor/grok-4.6-fast@xhigh, claude-code/default@xhigh
interrogate reviewers: claude-code/fable[1m]@max, codex/gpt-5.6-sol@max, cursor/grok-4.6-fast@xhigh, claude-code/default@xhigh
```

### 6. Confirm

Tell the user the shared configuration was written and that pstack hosts will use it in new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` wherever pstack is installed. On no, move on without pushing.
