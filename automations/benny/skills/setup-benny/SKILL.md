---
name: setup-benny
description: Configure Benny and prepare its triage and repro manual skills. Use when installing Benny or changing its Slack, tracker, repository, routing, control, or budget settings.
---

# Set up Benny

Benny ships as a dormant manual-skills pack inside pstack. The plugin manifest exposes only pstack's normal skill root; this file and the two operational files are not slash skills.

> Muse note: Cursor Automations has no Muse equivalent, so there is no `/automate`, no Automations editor, and no runtime that watches Slack or wakes on a report. The two benny skills run manually, invoked by name with the report coordinates, or from a scheduled task the user sets up. This skill configures the pack and the shared dependencies; it never creates a runtime.

The human enters setup by pointing Muse at the pack's `FOR_AGENTS.md`. The bootstrap flow copies the whole pack into the target repository, then reads this file directly at `benny/skills/setup-benny/SKILL.md`.

Benny needs external configuration and two configured skills. It needs no live automation.

Do not set up a scheduled task for either skill until the user explicitly asks. Never put a secret value in plugin files, prompts, or committed configuration.

## 1. Copy the pack and confirm shared pstack skills

Do this before asking for Benny configuration and before preparing any scheduled task.

Ask which repository will run the skills. The source pack is the directory containing `FOR_AGENTS.md`. The destination is `<target-repository>/benny/`.

Merge the entire source pack into the destination:

1. Create the destination when it is absent.
2. Copy every source file to the same relative path.
3. Preserve destination-only files. Never delete unrelated files during install or refresh.
4. Keep user-owned configuration, feature maps, and routing maps outside the destination. Never overwrite them.
5. When an existing source-managed file differs, inspect the diff and merge without discarding local edits. If ownership is ambiguous, stop and ask before replacing it.
6. Verify that the destination contains `FOR_AGENTS.md`, this setup file, both operational files, their references, and the templates.

If this file is already being read from the target destination, treat the copy as complete and run the same verification before continuing.

Start a fresh agent rooted in the target repository. Verify that these shared pstack skills resolve from the installed pstack-muse location:

- `how`
- `why`
- `tdd`
- `unslop`
- `principle-separate-before-serializing-shared-state`
- `principle-minimize-reader-load`
- `principle-guard-the-context-window`
- `principle-sequence-verifiable-units`
- `principle-fix-root-causes`
- `principle-prove-it-works`

Do not count a skill loaded from the current session only. The check must show that a fresh agent in the target repository receives pstack from the installed location.

If any shared dependency does not resolve, stop and explain the failure.

The Benny files are read directly from `benny/`. Do not add that directory to a plugin manifest or expect its `SKILL.md` files to appear in a slash-skill list.

Tell the user that `benny/` and any referenced secret-free configuration must be committed before either skill runs on live reports. Do not commit them unless the user asks.

Once this check passes, a run may read the committed operational files by their stable repository-relative paths. They must not embed a copied excerpt of the file contents.

## 2. Adapt the configuration

Open these copied examples:

- `../../templates/configuration.example.yaml`
- `../reproduce-and-fix-issues/references/feature-map.example.md`

Create user-owned copies outside `benny/`. These are configuration files, not pack files. Example locations:

- Project config, such as `.benny/configuration.yaml`
- Project feature map, such as `.benny/feature-map.md`
- Project routing map, such as `.benny/routing.md`
- User config, such as `~/.config/benny/configuration.yaml`
- User feature map, such as `~/.config/benny/feature-map.md`

Fill one feature-map section for every user-facing feature the skill may reproduce. Keep it at the user point of view. Do not freeze implementation details or current code paths in the map.

Do not edit the copied examples. Pack refreshes may update source-managed files after conflict review, but they must never touch the user-owned copies.

Prefer committed, secret-free files in the target repository when a fresh run must read them. Otherwise paraphrase the required values into the invocation. Reference a repository file only after confirming that the file is committed in the repository where the skill runs.

Use stable repository-relative paths for committed pack and configuration files. Never reference the plugin source directory or a plugin cache path from a live run.

## 3. Fill the required choices

Ask for or confirm:

- Source Slack channel ID
- Optional operations or status channel ID
- Repository URL and default branch
- Triage identity or Slack user ID
- Issue tracker type, team, project, labels, and intake status
- Tracker adapter skill or connector actions
- Optional routing map path
- Required control skill name
- Required user-facing feature-map path
- Status emoji strings
- Pull request URL format
- Polling and effort budgets

There is no per-role model choice in Muse: every role runs the one model, so no model slugs are configured. (N/A: upstream asks for a model slug per role from the Cursor model picker.)

The source channel, triage identity, repository, tracker adapter, control skill, and feature map must be explicit. Fail setup if any required value stays ambiguous.

Use pstack's `unslop` skill on the final skill names, descriptions, and invocation briefs before saving them.

## 4. Check integration capabilities

The triage skill needs:

- Read access to the configured source Slack channel and its threads
- Thread-reply access in that channel
- Attachment metadata and file download access when reports include media
- Search, read, create, and update access through the configured issue-tracker adapter

The repro skill needs:

- Read access to the source thread
- Thread-reply access in the source channel
- Optional post and edit access in the configured operations channel
- Repository read and history access
- A pull request action that can open a draft pull request (`gh pr create --draft` via `muse.exec`)
- The configured control-adapter skill

Prefer the configured Slack actions from the messaging connector (find it with `muse.connector_search`) for reads and posts. The optional `BENNY_SLACK_BOT_TOKEN` may fill a narrow gap such as editing one operations status message or downloading an attachment. Store the value in a secret manager or environment, not in YAML.

Do not use undocumented integration endpoints.

## 5. Prepare the routing map

If the user wants reroutes or owner pings:

1. Copy `../triage-issue-reports/references/routing.example.md` outside `benny/`.
2. Replace every placeholder with public or organization-local values.
3. Keep owner pings off by default.
4. Allow a ping only for a configured feature owner or a confirmed likely regression author.

If no routing map is configured, triage may classify a report but must not guess a destination or owner.

## 6. Verify the control adapter

Read `../reproduce-and-fix-issues/references/control-adapter.md` and the user's completed feature map.

Confirm that the named skill can:

- Bring up the target app
- Navigate every mapped feature through the real UI
- Exercise mapped states through declared adapter actions
- Inspect state without forcing the result
- Capture screenshots
- Start and stop a recording
- Clean up its processes and temporary data

If any capability is missing, leave the repro skill disabled. It must fail closed rather than claim a reproduction it did not perform.

## 7. Prepare the scheduled tasks

Ask whether this is first-time setup or configuration of existing scheduled tasks.

Read `../../FOR_AGENTS.md` from the copied pack as the primary user-intent source for either path. Use it to understand the two skills, their inputs, tools, instructions, outcomes, and shared rules.

### First-time setup

Prepare one scheduled-task brief at a time, and only when the user explicitly asks for scheduled runs. (Manual invocation by name needs no brief: the operator just invokes the skill with the report coordinates.)

For each brief:

1. Read the matching copied prompt template as secondary internal source material.
2. Turn `FOR_AGENTS.md`, the finished Benny configuration, and the template intent into a complete natural-language brief.
3. Tell the brief to read and follow its exact committed operational file under `benny/`.
4. Use the stable repository-relative path, not a plugin source or cache path. Do not copy the operational file contents into the brief.
5. Let the scheduled task poll the source Slack channel for new top-level reports and invoke the skill with each report's coordinates.
6. Confirm that the copied pack and any referenced configuration files are committed in the same repository where the skill will run.

Give the triage brief this complete intent, filled from configuration:

- Name `benny-triage`.
- Read and follow `benny/skills/triage-issue-reports/SKILL.md` for every run.
- Poll the configured source Slack channel for new top-level reports.
- Read the triggering thread and reply only inside it.
- Use the configured issue-tracker integration.
- Classify, inspect evidence, trace cause, dedupe, and create only clear new bugs.
- End one thread-only verdict with the configured `[benny:bug]`, `[benny:performance]`, or `[benny:other]` marker and optional tracker URL.
- Never post a source-channel root message.

After the triage brief is complete, give the repro brief this complete intent:

- Name `benny-reproduce`.
- Read and follow `benny/skills/reproduce-and-fix-issues/SKILL.md` for every run.
- Poll the same source Slack channel for new top-level reports.
- Use the configured repository and default branch.
- Read the source thread and reply only inside it.
- Include pull request creation and the configured tracker, control-adapter, and feature-map requirements. Paraphrase mapped user paths and states unless an eligible committed file in the same repository is confirmed.
- Wait for a trusted triage marker before acting.
- Reproduce the exact symptom twice through the mapped real UI and capture evidence.
- Verify an existing fix without authoring over it.
- Attempt an optional bounded fix only after confirmed repro, then open a draft pull request when proof and checks pass.
- Never post a source-channel root message.

Do not duplicate the scheduled task's own polling, authentication, or scheduling work in the brief.

### Existing scheduled tasks

There is no automation editor to inspect or update existing tasks through. Finish configuration, routing, control-adapter, and feature-map validation. Then give the user this concise checklist to edit each scheduled task directly.

For the existing triage task, update:

- Name and description
- Direct instruction to read `benny/skills/triage-issue-reports/SKILL.md`
- Source Slack channel to poll for new top-level reports
- Slack thread read and reply capabilities
- Issue-tracker integration
- Paraphrased triage instructions, thread-only rule, and Benny verdict markers

For the existing repro task, update:

- Name and description
- Direct instruction to read `benny/skills/reproduce-and-fix-issues/SKILL.md`
- Matching source Slack channel to poll
- Repository and default branch
- Slack thread read and reply capabilities
- Pull request action
- Tracker, control-adapter, and feature-map requirements
- Paraphrased marker wait, evidence, verification, and bounded-fix instructions

Ask the user to update each existing scheduled task directly. Do not create replacements or duplicates.

### Creation boundary

Never call a direct automation backend service or backend automation tool. Never use a browser URL that carries draft fields. Never build or open a Cursor protocol deep link. There is no Automations editor in Muse; the finish path is the scheduled task the user sets up, or manual invocation by name.

Do not enable scheduled runs for either skill until the thread-safety test passes.

## 8. Test thread safety

Use a test channel or a harmless test report, run by invoking each skill by name.

Before testing, confirm that the target repository's `benny/` and every referenced secret-free configuration file are committed on the branch the skill runs from. Confirm that both invocation briefs point at their exact committed operational files. If any check fails, stop. Tell the user that the skill cannot run on live reports yet.

Verify:

1. Triage stores the root `thread_ts` and posts exactly one verdict as a reply.
2. The verdict contains one configured marker.
3. Repro accepts the marker only from the configured triage identity.
4. Repro keeps the same immutable source coordinates.
5. No source-channel root message appears.
6. A delegated worker cannot use any Slack write action.
7. Missing coordinates, a deleted parent, or a failed preflight produces no post and no tracker issue.

Enable normal traffic only after all seven checks pass.
