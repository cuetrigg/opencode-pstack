---
name: setup-pstack
description: Configure which models pstack prefers per role and at what reasoning budget. Detects your available OpenCode models and writes a rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.config/opencode/pstack-models.md`. The opencode-pstack plugin reads that file and adds it to the model context. It is a plain Markdown rule with no frontmatter.

OpenCode does not let the `subagent` tool pick a model per call. Role lines are preferences: use them when you choose an agent or a session model, and say when OpenCode cannot honor one. `inherit-parent` means the session model.

## Steps

### 1. Detect available models

Run `opencode models`. Each line is a real model ID in `provider/model` form. If the user configured variants, a model can also be written `provider/model#variant`; only use a variant you have confirmed in the OpenCode configuration. If the command lists nothing, ask the user to paste the model IDs they can use. Never write a real model ID you have not confirmed. The aliases `inherit-parent` and `auto` are always valid.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.config/opencode/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer the `question` tool over free text. Offer these four options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed. `unlimited`, `large`, `medium`, and `small` set the effort token of every real model to `max`, `xhigh`, `high`, or `medium`. The effort token is the variant after `#` when one is present, or the trailing `-max`, `-xhigh`, `-high`, `-medium`, `-low`, or `-fast` part of a model ID. If the result is not a detected model ID, use the same family's detected model with the highest effort at or below the target, else mark the role as needing a choice. `inherit-parent` and `auto` do not change. When a model has no variants or effort suffixes, leave the model ID as it is and record the budget label.

**(c) Show the roles and confirm.** Show every role with its model, marking any real model not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` and `auto` as the options. Prefer the `question` tool over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries entries in list order. `inherit-parent` or `auto` means the parent model. `swarm workers` is the default model preference for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real model ID written must be in the detected set. `inherit-parent` and `auto` always pass. If a chosen real model is not available, stop and ask again.

### 5. Write the rule

Write `~/.config/opencode/pstack-models.md` with a `# budget` line with the chosen label and its target effort, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# pstack model configuration for OpenCode. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role uses the session model. OpenCode cannot set a model per subagent call.
# budget: large (xhigh)
feature, refactoring: inherit-parent
bug-fix: inherit-parent
perf-issue: inherit-parent
hillclimb: inherit-parent
judgment and prose: inherit-parent
hardest tasks: inherit-parent
how explorer: inherit-parent
how explainer: inherit-parent
why investigators: inherit-parent
why synthesizer: inherit-parent
reflect tooling: inherit-parent
reflect judgment, divergent, synthesizer: inherit-parent
arena runners: inherit-parent, inherit-parent
arena cross-judge pool: inherit-parent, inherit-parent
swarm workers: inherit-parent
architect runners: inherit-parent, inherit-parent
interrogate reviewers: inherit-parent, inherit-parent
```

### 6. Confirm

Tell the user the rule was written. The plugin adds it to the model context on new requests; re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, load the `create-verification-skill` skill with the `skill` tool. On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, load the `create-formal-verification-skill` skill with the `skill` tool. On no, move on without pushing.
