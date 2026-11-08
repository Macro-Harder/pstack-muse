# Porting notes: pstack 0.15.36 → Muse

Source: upstream `pstack/` at cursor/plugins@6b03002 (version 0.15.36), 58 skills,
23 playbooks, 24 principles, 10 guide pages, 2 agents, benny automation pack, scripts.
This repo is the Muse port. The upstream original lives at
https://github.com/cursor/plugins/tree/6b0300214a09fd6029cfb7077043125758cd1fd2/pstack.
Credit: all of pstack is by Lauren Tan (@poteto),
MIT, Copyright (c) 2026 Lauren Tan. The LICENSE file is kept verbatim.

Rule for the whole port: change the tooling references, not the ideas. The methodology
(go deep first, smallest verifiable units, test-first where cheap, prove it works,
delete before adding, parallelise only when trustworthy) is unchanged everywhere.

## Cursor / Claude Code → Muse mapping

| Upstream | Muse equivalent |
|---|---|
| `Task` tool (`subagent_type`, per-role `model`, `run_in_background`) | `subagent.spawn` (async by default; no `model` parameter — every delegate runs Muse) |
| `subagent_type: "poteto-agent"` | A subagent brief that says "read pstack-muse `poteto-mode` SKILL.md in full first" plus its absolute path. Not a type. |
| `generalPurpose` subagent | `subagent.spawn` |
| Model slugs (`grok-4.7-xhigh-fast`, `claude-opus-5-5-xhigh`) and per-role model routing | N/A. Muse runs one model. `setup-pstack` was rewritten to configure effort and fan-out preferences instead. |
| Multi-model review panels (arena runners, interrogate reviewers, reflect reviewers) | Same-model independent runs via `subagent.spawn`. Agreement between two independent runs is still high signal. |
| `AskQuestion` tool | Ask in chat text. A bounded choice uses `muse.create_options`. |
| Todo list | `todo.write` |
| Shell, file reads/writes, diffs | `muse.exec`, `muse.read`, `muse.write`, `muse.edit` |
| Memory / chat history | `muse.memory_search`, `muse.memory_get` |
| MCP tools / connectors | `muse.connector_search`, then the connector's own tools |
| `gh` CLI, git | Same commands, run with `muse.exec` |
| `~/.cursor/rules/pstack-models.mdc` (per-role model rule) | N/A. No model rule file in Muse. |
| `~/.cursor/plugins/` skill search path | This repo, or the installed copy under `~/workspace/custom-skills/` |
| `~/.claude/rules/` | N/A |
| Slash commands (`/poteto-mode`, `/how`, ...) | Invoke the skill by name. Muse has no slash-command menu; `skill_search` finds them. |
| Custom Modes (option+enter keeps a skill on) | N/A. Invoke the skill by name at the start of each task. |
| `/loop` | N/A. No loop command in Muse; long runs use subagents and scheduled work. |
| `create-skill` (Cursor built-in) | Skill template at `/opt/muse/skills/custom-connector-setup/templates/skill/` |
| Cursor / Claude Code hooks | Muse `hooks` namespace for event-driven automation where one fits; otherwise marked N/A in place |
| Bugbot / agentic security review | N/A (Cursor-only). The skeptical review posture is kept; Bugbot-specific steps are marked N/A. |
| Cursor Automations, `/automate` (benny runtime) | N/A. Benny's triage / reproduce-and-fix skills are kept as manual skills; there is no automation runtime for them in Muse. |
| `deslop`, `control-cli`, `control-ui` | Not shipped. They belong to `cursor-team-kit`, which is not ported here. |
| `harness/pstack` (Claude Code generated copy) + `generate-pstack-harness-trees.mjs` | N/A for Muse. This directory IS the Muse copy. |
| `docs/harness/claude-code.md` | Replaced by `docs/harness/muse.md`. The Claude adapter is unchanged upstream at `https://github.com/cursor/plugins/blob/main/pstack/docs/harness/claude-code.md`. |
| Frontmatter `disable-model-invocation`, `icon`, `color`, `is_background` | Dropped. Muse skill metadata lives in each skill's `skill.yaml` (name, title, description, category). |
| `.cursor-plugin/plugin.json` | Removed. Root `plugin.json` is adapted (name `pstack-muse`, version 0.15.36, author Lauren Tan, MIT); `$schema` removed, informational only. |

## Per-group notes

### Core mode (`poteto-mode`, `poteto-help`, `setup-pstack`)
- `poteto-mode`: Subagents section rewritten for `subagent.spawn`; the model-matching
  paragraphs replaced with the one-model rule; `/loop`, Custom Modes, and `deslop` /
  `control-cli` / `control-ui` references marked N/A; Bugbot triage kept as posture.
- `setup-pstack`: rewritten. The 8-step model-detection flow is gone; it now records
  effort and fan-out preferences (panel sizes, verification strictness) and still
  offers the verification-skill and formal-verification-skill steps.
- `poteto-help`: routing table kept; install, Custom Mode, `/loop`, and model-rule
  answers rewritten for Muse; public links now point at this repo.

### Principles (24 `principle-*` skills)
- Methodology only, almost no tooling references. Kept verbatim except skill.yaml
  metadata and any Cursor tool name that appeared. No idea changed.

### Situational skills (31 non-principle, non-mode skills)
- Tool references adapted per the mapping above. Skills that existed only to drive a
  Cursor feature (`make-bot-ui` needs Grok Bot + Tailscale webhook) are kept with a
  Muse note on what still works and what needs the user's own bot setup.

### Playbooks (23, under `poteto-mode/playbooks/`)
- Steps kept verbatim where they are methodology. Tool calls rewritten to Muse tools.
  Shipping / babysit playbooks: `gh` stays (via `muse.exec`). The `origin` forge CLI
  is kept as a conditional (`if command -v origin succeeds ... else gh`); it is not
  present in Muse's environment, so the `gh` fallback is what runs. Cursor's built-in
  babysit / autopilot loop commands are replaced by subagents plus scheduled work.

### Guide (`docs/guide/`, 10 pages + README)
- Rewritten for Muse throughout: install, setup, mode invocation, overnight runs
  (loops → subagents + scheduled work), and the recipes/pitfalls page.

### Agents (`agents/`)
- `poteto-agent.md`: wrapper brief rewritten for `subagent.spawn`; the
  `~/.cursor/plugins/` search path replaced with this repo's location.
- `comment-sicko.md`: byte-identical to upstream. Its first line is a
  first-output contract ("My first output when spawned is exactly this."),
  so no Muse header note could be added without breaking it. The Muse use
  (paste the brief into `subagent.spawn`) is documented here, not in the file.

### Automations (`automations/benny/`)
- Kept as manual skills. The Cursor Automations runtime, `/automate`, Slack
  triggers, and `.cursor/automations/` install paths are marked N/A in place.
  The triage / reproduce-and-fix workflow itself (classify, reproduce twice,
  verify, draft PR) is unchanged.

### Scripts (`poteto-mode/scripts/`)
- `check-plan.mjs` + tests, `watch-pr`, `worktree-audit.sh`, `bootstrap.ts`:
  adapted where they shell out to Cursor/Claude CLIs; kept where they are plain
  node/git. See the scripts section below for what runs here.

## Scripts: what runs here

Environment: Node v24.22.1, bun 1.4.4, `gh`, `jq`, `rg` all present on this
machine. No Java/TLC (see statechart below).

- `check-plan.mjs` + `check-plan.test.ts`: ported. The `"/loop 1h"` boot-recipe
  marker is now `"subagent.spawn"`; the live-lanes marker is `"Ten lanes on
  \`<swarm workers>\` at the PR head"` (no per-model lanes in Muse). Test
  fixtures updated to match. `bun test`: 61 pass, 0 fail across `watch-pr` +
  `check-plan` (see below).
- `watch-pr/` (TypeScript, commander): ported. All GitHub reads go through the
  `gh` CLI, unchanged — `gh` carries its own auth, so no token handling was
  added. Bugbot-specific logic removed: `isBugbot` now matches only
  Bugbot-authored comments (the `author === "cursor"` + `CURSOR_AUTOMATION_ID`
  branch is gone), `passKey` is gone, and `bugbotReviewPasses` is always 0 —
  Bugbot passes count Cursor automation runs, which do not exist in Muse; the
  field stays in the output schema so JSON consumers are unaffected. The two
  Bugbot tests were rewritten to assert the new behavior. `reviewAutomationRunning`
  (policy.ts) is unchanged: a pending check named for a review automation is
  still a real pending check on GitHub. `tsc --strict` clean.
- `worktree-audit.sh`: ported. The `origin` CLI branch was dropped (the script
  uses `gh` only now). The last-chat signal changed from Cursor's
  `~/.cursor/projects/<slug>/agent-transcripts/` to Muse's `~/memory` notes
  (newest note mentioning the worktree path). Otherwise plain git/shell,
  runs under `muse.exec`.
- `bootstrap.ts`: unchanged (bun install helper for watch-pr's commander dep).
- Package names `@cursor-skill/poteto-mode-tools` and `@cursor-skill/statechart`
  renamed to `@pstack-muse/*`.
- `statechart/scripts/` (TLA+ toolchain): ships as-is except the package
  rename — its one "cursor" hit is `loopCheck`, an unrelated loop-check pass.
  Pure-TS tests (`node --test` on chart, render, messages, pipeline, cli):
  **267 pass, 0 fail**. The TLC-dependent tests (corpus, differential,
  plain-tlc, tlc-out, config-fuzz) need Java/TLC, which this VM lacks, so they
  were not run here.
- `orch` scripts were deleted upstream in 0.15.36, so there is nothing to port.

## Skill metadata (`skill.yaml`)

Every skill carries a `skill.yaml` next to its `SKILL.md` (name, title,
description, category). 55 were generated mechanically from the ported
frontmatter (slash invocations dropped, "multi-model" → "independent"); 3 were
hand-written (`poteto-mode`, `setup-pstack`, `poteto-help`). Benny skills are
manual skills and carry no `skill.yaml`.

## Deliberately unchanged

- `LICENSE` (MIT, Copyright (c) 2026 Lauren Tan) — verbatim.
- All 24 principle skills' rules — verbatim apart from metadata.
- The methodology in every playbook step — verbatim apart from tool names.
- `assets/logo.png` and guide images — unchanged.
- `comment-sicko.md` — byte-identical (see Agents above).
- `statechart/scripts/` — as-is except the package rename.
- The `origin` forge CLI branches in the playbooks — kept conditional with a
  `gh` fallback (see Playbooks above); `origin` is not present in Muse's
  environment, so they cost nothing and preserve the dual-forge structure.
- `benny/skills/reproduce-and-fix-issues/references/verify-existing-fix.md` —
  byte-identical; its content (verify an existing PR/commit, don't author a
  competing patch) names no tooling.

## Repo layout: standalone repo

- This repo is a standalone port, not a fork: it holds only the Muse port of
  pstack. The upstream original lives at cursor/plugins@6b03002 (pstack
  0.15.36); the AgenC packaging lives in its own repo,
  `Macro-Harder/pstack-agenc`.
- The port was first published inside the fork `Macro-Harder/pstack-agenc`
  (as a sibling directory `pstack-muse/`, so the upstream `pstack/` stayed
  pristine) and later moved here as its own repo. GitHub allows one fork per
  account per repo, which is why the fork kept the `pstack-agenc` name.
- Sync note: the first sync push into the fork (original helper, all ~1,736
  blobs) failed after ~38 min with `tree failed: 502 Server Error` from
  GitHub; the retry with the changed-blobs-only helper landed as `0eb238a`
  (443 blobs).
