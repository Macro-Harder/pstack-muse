---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

Invoke when the user says "reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## 1. Locate the active transcript

Before fanning out, get this conversation in front of the reviewers. The active conversation is in context; there is no user-facing transcript directory to glob in Muse, so do not go looking for one. If the session is long, pass a tight digest of it instead: the opening user prompt, the decisions, the corrections, and the artifacts. Do not read other projects' records. If no digest can be built, pass the conversation itself.

## 2. Spawn three reviewers in parallel

One message, three `subagent.spawn` calls. There is no per-role model in Muse: every delegate runs the same model as the parent chat, so the three lenses are same-model independent runs, not different models. Agreement between independent runs is still high signal. Reviewers need connector access to look up the tickets, chat threads, and observability traces the transcript references; use `muse.connector_search` to find the connector for each source, and skip an unavailable one and say so.

| Lens | Prompt template |
|---|---|
| Judgment | `references/judgment-reviewer.md` |
| Tooling | `references/tooling-reviewer.md` |
| Divergent | `references/divergent-reviewer.md` |

Pass each template verbatim, substituting the transcript or digest where marked. Reviewers return findings in the subagent response body.

## 3. Synthesize

One `subagent.spawn` call. It spot-verifies citations through the connectors. Pass `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. It returns an Accepted / Rejected / Backlog list.

## 4. Structural enforcement check

Move any Accepted item that a lint rule, script, metadata flag, or runtime check would enforce more reliably to Backlog. See the **encode-lessons-in-structure** principle skill.

## 5. Apply

Present the synthesizer's full Accepted / Rejected / Backlog output and wait for explicit approval before applying any Accepted edit. The user picks the subset and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

File each Backlog item to your team's devex or backlog tracker without waiting. Only the Accepted list waits for approval.

Never edit a skill under the installed copy without copying it first. Before approval, show its destination, `~/workspace/custom-skills/<skill>/` or the project skills directory. After approval, copy the skill folder there and edit the copy. Never open a PR or push.

Follow each approved row's Routing exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): the parent does it directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): hand to the skill template at `/opt/muse/skills/custom-connector-setup/templates/skill/` and run its draft / test / iterate loop. (Upstream hands this to Cursor's built-in `create-skill` skill, which Muse does not have; the template is the Muse equivalent.)
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): hand to the skill template and run its description-optimization loop.
- `new skill via create-skill: <kebab-name>`: hand creation to the skill template. Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done.

## 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.

## What did not carry over

Upstream finds the active transcript in Cursor's `agent-transcripts/` directory and spawns the three reviewers on different models (judgment and divergent on `claude-opus-5-5-xhigh`, tooling on `grok-4.7-xhigh-fast`), picking each from the `pstack-models.mdc` rule. None of that exists in Muse: no transcript directory, no per-role model routing, no model rule file. The lenses and their prompt templates are unchanged; the reviewers are same-model independent runs. The reviewer templates in `references/` still name Cursor transcript and tool paths in one line each; those lines are upstream wording for the transcript the reviewer reads, and the transcript or digest passed in step 1 is the Muse substitute.
