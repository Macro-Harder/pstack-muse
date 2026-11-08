---
name: swarm
description: "Fan out N parallel workers, drain them, and return one report. Use for swarm, 'swarm this', or parallel coverage, races, gauntlets, and exploration."
---

# Swarm

## Start

Open a todo list with `todo.write`, with one entry per phase before launching anything.

1. Frame
2. Fan out
3. Aggregate
4. Report

## Phase A: Frame

1. State the done predicate and the artifact or report the swarm must return.
2. Choose the shape. Partition into slices, race N workers on identical briefs, or mix both. For a race or mixed shape, declare `first pass`, `rank all`, or `best-of` before spawning.
3. Pick the worker effort. Muse runs one model, so there is no per-role model to pick and no model race: a race pits different briefs or approaches against each other, run as same-model independent runs. Upstream picks the worker model from the `swarm workers` line in `~/.cursor/rules/pstack-models.mdc` (default `grok-4.7-xhigh-fast`); that rule file does not exist in Muse. Pick each subagent's brief depth per the Subagents section of the **poteto-mode** skill.
4. Give each worker its own writable output when it writes. When workers verify or measure commits, each brief names the exact SHAs. A measurement brief also names the method (sample count, what one sample is, order). The worker records both in its result.

## Phase B: Fan out

Spawn all N workers in one message with `subagent.spawn`. Spawns are async by default in Muse; there is no `run_in_background` flag to set and no `environment: "cloud"` / `"local"` switch, since Muse subagents run on this machine and there is no separate cloud worker environment. When a worker needs access to something on the user's own computer rather than this VM, say so in the brief and note it as a gap instead of assuming it can reach it. The non-default pushed branch option (`cloud_base_branch` upstream) is N/A in Muse: name the branch in the brief and have the worker check it out with `muse.exec`.

Every brief stands alone. Include the goal, scope, exact slice or race arm, how to verify, and what to report. Reports use `PASS`, `ISSUES`, or `BLOCKED` with evidence. A worker that can prove a defect reports `ISSUES` and lists every issue it can prove, not only the first.

If a worker drops out, proceed with N-1 and note it.

## Phase C: Aggregate

Read the terminal results. Drop a result that does not record the SHAs and method its brief names, and respawn that worker once. After a second miss, record a gap. A gap does not count as a pass. For coverage, every required slice needs a result. For a race, apply the selection rule declared up front. Use first pass, rank all, or best-of. Do not paste raw worker dumps.

## Phase D: Report

Return one consolidated in-chat report with a result table, issue one-liners, gaps or dropouts, and the race rule when used.
