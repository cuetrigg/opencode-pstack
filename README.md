# opencode-pstack

[pstack](https://github.com/cursor/plugins/tree/main/pstack) ported to [OpenCode](https://opencode.ai) as a native plugin.

This is pstack 0.15.36: every skill, playbook, reference, principle, and both subagents, with the Cursor-specific tool and path names adapted for OpenCode. It does not add pstack skills or commands that do not exist upstream.

## What it registers

- **58 skills**, including `poteto-mode`, its 23 playbooks, the 24 principles, `setup-pstack`, and every other pstack skill.
- **58 slash commands**, one per skill. Running `/poteto-mode <task>` attaches the skill to the prompt for that turn, the way Cursor attaches a skill.
- **2 subagents**: `poteto-agent` and `comment-sicko`.
- **One context rule**: the per-role model preferences in `~/.config/opencode/pstack-models.md`, when that file exists.
- **One session hint**: the current OpenCode session ID, so pstack's transcript skills can run `opencode session export <id>`.

No Cursor-only feature is invented. Bugbot, Cursor Automations, cloud agents, Custom Modes, `/loop`, and the `cursor-team-kit` skills are skipped with a one-line note. The mapping is in [`content/docs/harness/opencode.md`](content/docs/harness/opencode.md) and is prepended to every skill.

## Install

OpenCode 2.2.5 or newer is required. This plugin uses the V2 plugin API.

```bash
git clone https://github.com/<owner>/opencode-pstack.git ~/Projects/opencode-pstack
cd ~/Projects/opencode-pstack
bun install
```

Add the plugin directory to your global OpenCode configuration, `~/.config/opencode/opencode.json(c)`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "plugins": [
    "/home/you/Projects/opencode-pstack"
  ]
}
```

Reload OpenCode. The plugin auto-installs the two agents into `~/.config/opencode/agents/` on load. To install them by hand instead, run:

```bash
bun run install-agents
```

The installer writes a marker at the end of each generated agent. Remove the marker if you want to keep your own edited copy; the plugin then leaves the file alone.

The runtime has no npm dependencies. `bun install` is only needed for type checking and the check script. OpenCode loads `src/index.ts` with Bun and resolves the type-only `@opencode/plugin` import at build time.

## Use

Start a task with `poteto-mode`:

```text
/poteto-mode fix the scroll drift. reproduce it first, then fix and verify.
```

OpenCode has no Custom Mode, so the skill applies to that turn. Start each new task with the command, or load the skill explicitly with the `skill` tool (`poteto-mode`). Every other pstack skill has the same name as its command:

```text
/setup-pstack
/how do we cancel runs?
/why is this flag off?
/interrogate review this diff.
```

`/setup-pstack` writes `~/.config/opencode/pstack-models.md`. The plugin adds that file to the model context when it exists. Real OpenCode model IDs come from `opencode models`.

## Differences from the Cursor plugin

These are limits of the OpenCode harness, not new pstack behavior:

- **No per-subagent model choice.** OpenCode's `subagent` tool takes an agent, not a model. pstack role lines are preferences used when choosing an agent or session model; they are not enforced per call.
- **No Custom Mode.** Running a pstack command attaches the skill for one turn.
- **No `/loop`.** The autonomous-run, babysit, and shipping playbooks re-check in the session instead of arming a Cursor loop.
- **No cloud agents.** Workers run locally, or in an OpenCode worktree when isolation is needed.
- **No Cursor Automations or Bugbot.** The benny automation pack is included in `content/automations/` for reference, but it is not registered. Ordinary GitHub review threads are triaged with `gh`.
- **No `cursor-team-kit`.** `deslop`, `control-ui`, and `control-cli` are not bundled. The skills use `unslop`, `no-comments`, and the repository's own drivers instead.

## Development

```bash
bun install
bun run check
```

`bun run check` type-checks the plugin and runs `scripts/check.ts`. The check loads all skills and commands with a fake plugin context, verifies the frontmatter and agent install, and confirms a command attaches its skill to the prompt.

The pstack content lives in `content/` with the upstream layout preserved so relative links inside skills keep working:

```text
content/
  agents/
  docs/
  skills/
  automations/
  README.md
  LICENSE
```

## Upstream

pstack is by [Lauren Tan](https://x.com/poteto), MIT licensed. The original plugin and guide live at [cursor/plugins/pstack](https://github.com/cursor/plugins/tree/main/pstack). This port tracks pstack 0.15.36 and keeps the upstream license and README in `content/`.
