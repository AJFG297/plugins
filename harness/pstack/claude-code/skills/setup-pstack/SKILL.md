---
name: setup-pstack
description: Configure which providers and models pstack uses per role and at what reasoning budget. Detects your available combinations and writes a shared user-level configuration that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

Write `~/.agents/pstack-models.md`, a shared user-level configuration that sets pstack's provider, model, and effort per role.

## Steps

### 1. Detect available models

Enumerate the provider, model, and effort combinations you can pass to a subagent in this session. That is the dependable source. If the host exposes a delegated-model catalog, models API, or CLI that lists the user's entitled models, prefer it for completeness. If you cannot detect any, ask the user to paste the combinations they have access to. Never write a real combination you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not detected combinations.

Use `provider/model@effort` for real entries. Route through that provider and pass model and effort separately. Resolve the upstream Cursor slugs below to confirmed native-provider combinations; never guess model IDs. When a default is unavailable, use `inherit-parent` unless the user chooses another confirmed combination. Preserve configured providers, model identities, panel sizes, and fallback comments. If a configured combination is unavailable, use its explicit fallback, or inherit the parent model and report the fallback when none is declared.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.agents/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how explorer` or `why investigators`, is from a retired role. Drop it.

A rule without the `# defaults: 2` line comes from an older setup. That setup wrote a line for every role, so some of its values are old defaults, not choices. An old slug is one that matches `grok-*`. This migration applies only to legacy bare Cursor slugs; explicit `provider/model@effort` entries are user choices and stay. Before step 3, replace these values:

- An old slug in `feature, refactoring`, `bug-fix`, `perf-issue`, `hillclimb`, `reflect tooling`, or `swarm workers` takes the role's step 5 default.
- A `claude-opus-*` value in `hardest tasks` takes the role's step 5 default.
- In a panel role (`arena runners`, `arena cross-judge pool`, `architect runners`, `interrogate reviewers`), a list of exactly one `claude-opus-*` entry and one old slug, in either order, takes the role's step 5 list. In any other list, each old slug becomes `claude-opus-5-5-high-fast`.

Every other value is the user's choice and stays, including an old slug in any other role and every `inherit-parent` or `auto` value. Tell the user which values you replaced.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer the host's structured question tool over free text. Offer these four options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by family, list, or alias (`inherit-parent`, `auto`). Families are the ones Phase A of the **arena** skill defines, so Fable is its own family. Each role has a target effort, which is its default's effort moved by the budget. `large` keeps it, `unlimited` raises it one step, `medium` lowers it one step, and `small` lowers it two steps, on the ladder `max` > `xhigh` > `high` > `medium` > `low`. A target at either end of the ladder stays there. Most defaults are at `xhigh`, and each budget label names where those land. Set the effort token of every real entry in the role, panel entries included, to the role's target. For `provider/model@effort`, change only `@effort`; the provider and model stay. In the upstream Cursor slugs below, the effort token is the last token, or the one before a trailing `fast`. If the result is not a detected combination, use the same provider and model's highest confirmed effort at or below the target, else mark the role as needing a choice. `inherit-parent` and `auto` do not change. So `unlimited` turns `claude-opus-5-5-xhigh` into `claude-opus-5-5-max` and `claude-opus-5-5-high-fast` into `claude-opus-5-5-xhigh-fast`, and it keeps `claude-fable-5-2-max`. Gemini tops out at `high`, so under `unlimited` the fallback keeps `gemini-4-carbon-high` as it is. `small` turns the first two into `claude-opus-5-5-medium` and `claude-opus-5-5-low-fast`. Fable 5.2 and Gemini have no fast slug, so never write one.

**(c) Show the roles and confirm.** Show every role with its model, marking any real combination not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` and `auto` (both mean: this role runs on the parent chat model, which is how Auto users stay on Auto) as the options. Prefer the host's structured question tool over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries only values from a vendor other than the parent's, in list order, so order it by preference. Fable and the other Claude models are one vendor. An `inherit-parent` or `auto` value never judges. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real provider, model, and effort combination written must be in the detected set. `inherit-parent` and `auto` always pass. If a chosen real combination is not available, stop and ask again.

### 5. Write the rule

Write `~/.agents/pstack-models.md` with no frontmatter, the `# defaults: 2` line, a `# budget` line with the chosen label and the effort that label names, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent, preserving fallback comments. Shape below shows the upstream defaults as Cursor slugs; write the confirmed `provider/model@effort` combinations from step 3 in their place:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit Task `model`). Alias entries in a panel list still count toward its fan-out.
# defaults: 2
# budget: large (xhigh)
feature, refactoring: claude-opus-5-5-xhigh
bug-fix: claude-opus-5-5-xhigh
perf-issue: claude-opus-5-5-xhigh
hillclimb: claude-opus-5-5-xhigh
judgment and prose: claude-opus-5-5-xhigh
hardest tasks: claude-fable-5-2-max
reflect tooling: claude-opus-5-5-high-fast
reflect judgment, divergent, synthesizer: claude-opus-5-5-xhigh
arena runners: claude-fable-5-2-high, claude-opus-5-5-high-fast, gemini-4-carbon-high
arena cross-judge pool: claude-fable-5-2-high, claude-opus-5-5-high-fast, gemini-4-carbon-high
swarm workers: claude-opus-5-5-high-fast
architect runners: claude-fable-5-2-high, claude-opus-5-5-high-fast, gemini-4-carbon-high
interrogate reviewers: claude-fable-5-2-high, claude-opus-5-5-high-fast, gemini-4-carbon-high
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
