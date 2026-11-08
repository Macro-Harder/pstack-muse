# pstack in Muse

pstack skills use Cursor names. In Muse, read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack-muse>` means the folder that holds `skills/` and `docs/`. If a Cursor field has no row, drop it and say so. Never pass a field or value the tool schema rejects.

## Tools

| Cursor | Muse |
| --- | --- |
| `Task` | `subagent.spawn` |
| `AskQuestion` | Ask in chat text. A bounded choice uses `muse.create_options`. |
| Open a todolist | `todo.write` |
| Run another pstack skill (`/name` or "the **name** skill") | `muse.read` `<pstack-muse>/skills/<name>/SKILL.md` and follow it. When you tell the user to invoke it, give its name. |
| `Grep`, `Glob` | `muse.exec` with `rg` |
| Shell, file reads/writes, diffs | `muse.exec`, `muse.read`, `muse.write`, `muse.edit` |
| Memory / chat history | `muse.memory_search`, `muse.memory_get` |
| MCP tools | `muse.connector_search`, then the connector's own tools |

## Subagent fields

| Cursor | Muse |
| --- | --- |
| `subagent_type: generalPurpose` | `subagent.spawn` (no type) |
| `poteto-agent`, `comment-sicko` | The brief files in `agents/`. Paste the brief text into the `subagent.spawn` call. |
| `readonly: true` | The brief says "do not edit files", and you check `git status` afterward. |
| `readonly: false` | Omit. |
| `environment: "cloud"` | A git worktree. Name the branch or commit the worker checks out. |
| `environment: "local"` | Omit. |
| `cloud_base_branch: <b>` | Tell the worker to check out `<b>` first. |
| `run_in_background: true` | Omit. Subagents run in the background by default and report back when they finish. |
| `resume` | `subagent.resume`, or a fresh spawn with the earlier result pasted into its brief. |
| `file_attachments` | List the paths in the brief. |
| `model: <slug>` | Omit. Muse runs one model; there is no per-role model choice. |

## Models

There is no model table for Muse: every role runs the one model. Upstream pstack
routes code delegates to Grok and judgment to Opus 5.5; in this port those panels
run as same-model independent runs. A role value in the old `pstack-models.mdc`
rule has no meaning here. Effort and panel sizes live in
`~/workspace/user/pstack-muse-config.md`, written by `setup-pstack`. Read it when
it exists; missing means defaults.

## setup-pstack

Follow its steps with these changes: there is no model detection and no rule file.
Ask for an effort level and panel sizes, write them to
`~/workspace/user/pstack-muse-config.md`, and confirm. The verification-skill
offers at the end are unchanged.

## Paths

| Cursor | Muse |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `~/workspace/custom-skills/` |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | This repo; each skill carries a `skill.yaml` |
| `.cursor/rules/*.mdc`, always-applied rules | No equivalent. Standing instructions live in `~/SOUL.md`, `~/USER.md`, and `~/MEMORY.md`. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `https://github.com/Macro-Harder/pstack-muse/blob/main/<path>` when that file exists. The port ships the skills, `docs/guide/`, and `README.md`. Read the local copy and never fetch it. Give the user the URL when they need a link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | `muse.memory_search` over `~/MEMORY.md` and `~/memory/`. |

## Cursor-only features

Muse has no Cursor Automations, Bugbot, Cursor cloud agents, `/loop`, Custom Modes, or built-in PR tool. When a step needs one, put one line in your reply, `Skipped in Muse: <step>. <reason>.`, then go on to the next step. Do not stop, ask, or retry. When a later step needs what the skipped step made, such as a URL or a key, leave a placeholder and name it in the line.

| Cursor | Muse |
| --- | --- |
| Cursor Automations: routines, `update_state` routine calls, `api2.cursor.sh/automations` webhooks, routine wakes, and `SendToUser` secret requests | Skip with the line. |
| Bugbot | Bugbot comments on a GitHub PR are ordinary review threads, so triage them with the forge command that the step gives. Skip a step that starts, waits for, or configures Bugbot when the PR has no Bugbot check or comment. |
| Cursor cloud agents | A step that needs the cloud agent itself, such as its dashboard status or its own machine, is a skip. A step that only needs an isolated worker maps to a git worktree, as in Subagent fields. |
| A built-in PR tool | Use the forge command that the step gives, such as `gh pr create` via `muse.exec`. That is not a skip, so write no line. |
| `/loop` | Skip with the line. A long run in Muse uses subagents and scheduled work, not a loop command. |
| Custom Modes | Skip with the line. Invoke the skill by name at the start of each task. |
