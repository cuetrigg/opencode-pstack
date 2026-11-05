# pstack in OpenCode

pstack skills use Cursor names. You are running in OpenCode. Read this mapping once per session before following a skill. `<pstack>` means the `content/` directory that holds the installed plugin's `skills/`, `agents/`, and `docs/`. If a Cursor field has no row here, drop it and say so. Never pass a field or value that the OpenCode schema rejects.

## Tools

| Cursor | OpenCode |
| --- | --- |
| `Task` | The `subagent` tool. Pass `agent`, `description`, `prompt`, and optionally `background` or `sessionID`. |
| `subagent_type: "generalPurpose"` | `agent: "general"`. |
| `subagent_type: "poteto-agent"` | `agent: "poteto-agent"`. |
| `subagent_type: "comment-sicko"` | `agent: "comment-sicko"`. |
| `subagent_type: "Explore"` | `agent: "explore"`. It reads and searches without editing. |
| `readonly: true` | Use `agent: "explore"`. It keeps shell access, so the brief also says "do not edit files", and you check `git status` afterward. |
| `readonly: false` | Use `agent: "general"`. It keeps the session's MCP servers. |
| `AskQuestion` | The `question` tool. If the client cannot show a question, ask in plain text. |
| Open a todolist | Use `todowrite` when the session offers it. Otherwise keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | Load it with the `skill` tool by its exact ID, or run its slash command. OpenCode prefixes nothing; `/poteto-mode` is `/poteto-mode`. |
| `Grep`, `Glob`, `Read`, shell | The built-in `grep`, `glob`, `read`, and `shell` tools. |
| `run_in_background: true` | `background: true` on `subagent`. It returns immediately and notifies the parent when the child finishes. |
| `resume` | Pass the prior child's `sessionID` to `subagent`. |
| `file_attachments` | List the paths in the `prompt`. |
| `model` per subagent | Not available on the `subagent` tool. OpenCode chooses the model from the agent or the session. Use the agent that matches the role and say when a role's model cannot be honored. |
| `environment: "cloud"` | Not available. Run the worker locally, or create an OpenCode worktree first when isolation is needed, and say so. |
| `cloud_base_branch: <b>` | Create the worktree from `<b>` first, or check out `<b>` in the worker. |
| `/loop` | No built-in loop tool. Do not invent one. Continue in the session, re-check with the available tools, and say when the run needs a new turn. |
| Custom Mode | Not available. Start each task with the matching slash command, such as `/poteto-mode`; the command attaches the skill for that turn. |

## Subagents

OpenCode only allows subagent-mode agents in `subagent`, and the default nesting depth is one. If a pstack playbook needs a subagent to start its own subagents, set `experimental.subagent_depth` in OpenCode configuration or collapse that level into the parent.

The pstack agents are installed as global OpenCode agents:

- `poteto-agent` is the delegate wrapper for playbook steps.
- `comment-sicko` is the read-only comment reviewer used by `/no-comments`.

Do not substitute `general` for `poteto-agent` inside a playbook step. The wrapper loads the `poteto-mode` skill, including its principles index, before doing work.

## Models

pstack's Cursor model slugs, such as `claude-opus-5-5-xhigh` and `grok-4.7-xhigh-fast`, are not OpenCode model IDs. In OpenCode:

- Detect real models with `opencode models`. Each line is `provider/model`.
- A variant is appended as `#variant`, such as `openai/gpt-6.1-sol#high`, when the model exposes variants.
- `~/.config/opencode/pstack-models.md` holds the per-role choices. The plugin injects that file into the model context when it exists.
- `inherit-parent` and `auto` mean "use the session model".
- The `subagent` tool cannot select a model per call. Apply the role's intent when you choose the agent or switch the session model, and say when OpenCode cannot honor a role line.

The role defaults in pstack assume Cursor models. On a first run, use `setup-pstack` to replace them with models you actually have. When a role has no line, keep the skill's default behavior and use the session model.

## setup-pstack

`setup-pstack` writes `~/.config/opencode/pstack-models.md`, a plain Markdown rule with a `# budget` line and one line per role. The plugin reads that file and adds it to the model context. There is no always-applied `.mdc` rule and no frontmatter.

Detect models with `opencode models`. Never write a model ID you have not confirmed. The aliases `inherit-parent` and `auto` are always valid.

## Paths

| Cursor | OpenCode |
| --- | --- |
| `.cursor/skills/` | `.opencode/skills/` |
| `~/.cursor/skills/` | `~/.config/opencode/skills/` |
| `~/.cursor/plugins/` | The installed `opencode-pstack` plugin. Load a skill with the `skill` tool instead of searching this directory. |
| `.cursor/worktrees/` | `.opencode/worktrees/`, or the directory configured by OpenCode's `worktree.directory`. |
| `~/.cursor/rules/pstack-models.mdc` | `~/.config/opencode/pstack-models.md` |
| `.cursor/rules/*.mdc` | OpenCode instructions or `AGENTS.md`. The model rule is the exception above. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `<pstack>/<path>` when the file exists in the installed plugin. Read the local copy. Give the public URL only when the user needs a link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | The current session ID is in your context as `opencode-pstack:session`. Run `opencode session export <id>`. For another session, run `opencode session list`, pick it, then export it. Do not read `~/.cursor/projects`. |

Skill paths written in a skill body are relative to the directory that holds that skill's `SKILL.md`, as OpenCode documents. Read supporting files from there.

## Cursor-only features

OpenCode has no Cursor Automations, Bugbot, Cursor cloud agents, Custom Modes, or `/loop`. When a step needs one, put one line in your reply, `Skipped in OpenCode: <step>. <reason>.`, then go on to the next step. Do not stop, ask, or retry. When a later step needs what the skipped step made, such as a URL or a key, leave a placeholder and name it in the line.

| Cursor | OpenCode |
| --- | --- |
| Cursor Automations: routines, `update_state`, `api2.cursor.sh/automations` webhooks, routine wakes, `SendToUser` | Skip with the line. The benny automation pack is not registered as an OpenCode plugin feature. |
| Bugbot | Bugbot comments on a GitHub PR are ordinary review threads. Triage them with `gh` when it is available. Skip a step that starts, waits for, or configures Bugbot when the PR has no Bugbot check or comment. |
| Cursor cloud agents | Run locally. Create an OpenCode worktree only when the work must not touch the current checkout. |
| Cursor's built-in `create-skill` | Use the `authoring-a-skill` playbook or write the `SKILL.md` directly. Follow the authoring rules in the `poteto-mode` skill. |
| `cursor-team-kit` skills: `deslop`, `control-ui`, `control-cli` | Not bundled. Use `unslop` and `no-comments` for the slop pass. Prove UI or CLI behavior with the repository's own driver, tests, or a project verification skill. Say when a control skill was unavailable. |
| A built-in PR tool | Use `gh` or the forge command the step names. That is not a skip, so write no line. |

## Slash commands

Every pstack skill is registered as an OpenCode command with the same name. Running `/poteto-mode <task>` attaches the skill to the prompt for that turn. To keep the style across turns, run the command again at the start of each task, or load the skill with the `skill` tool.
