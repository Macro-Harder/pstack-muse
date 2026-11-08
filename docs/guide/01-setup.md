# Set up pstack

In this page you install the skills, set your effort and fan-out preferences, and run your first task. Setup is one command plus a short conversation.

## Install the skills

There is no plugin to install in Muse. The skills live in `pstack-muse/skills/` in this repo; once installed they live under `~/workspace/custom-skills/`. Read the [Muse adapter](../harness/muse.md) once per session: it maps every Cursor tool and path name a skill uses to its Muse equivalent, so a skill written for Cursor still reads correctly here.

## Set your effort and fan-out preferences

Run:

```text
setup-pstack
```

[`setup-pstack`](../../skills/setup-pstack/SKILL.md) asks for an effort level (`light`, `standard`, or `deep`) and for review-panel sizes (how many independent subagents `arena`, `interrogate`, and `reflect` run; defaults 2, 2, and 3), plus how strict verification should be. Answer the questions. It writes the choices to `~/workspace/user/pstack-muse-config.md`, which later sessions read.

There is no per-role model choice in Muse: every role runs the one model. Upstream pstack routes code delegates to one model and judgment to another and records those choices in a model rule file; in this port that routing is gone and the effort and panel sizes take its place. A review panel is a set of same-model independent runs, and agreement between two independent runs is still high signal.

You only override what you care about. A choice you don't set keeps the default. To restore a default, rerun `setup-pstack` and pick it again.

## Accept the verification offers, or don't

At the end of setup, `setup-pstack` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes a project-local skill, `verify-<app>/`, that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers it in depth.

If you're new to pstack, say yes. An agent that can check its own work keeps going until the check passes. An agent that can't hands every result back to you to check by hand. Of everything in this guide, the verification skill pays off the most.

Then setup offers once to generate a formal verification skill with [`create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md), unless the project already has one. Tests check the cases someone thought to write. A formal verification skill checks one key property of your code, such as "a job never runs twice at once", in every case within a stated scope. Say yes when your code has concurrency, retries, a state machine, or a parser, where a bug is costly and tests rarely reach the case that breaks. Otherwise say no, and run it later if that changes. [Verify and ship](./06-verify-and-ship.md#prove-a-property-with-a-formal-verification-skill) covers it.

After setup, start a new chat. The configuration applies to new sessions.

## Keep the cost in check

pstack spends extra tokens on subagents and review panels. That's the price of the rigor. To spend fewer:

- Rerun `setup-pstack` and pick a smaller effort level or shorter panel lists. Each panel entry runs one subagent.
- Save `poteto-mode` for work that needs rigor. A small, obvious edit doesn't.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt. If `poteto-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. Muse has no Custom Mode that keeps a skill on across turns (N/A: upstream keeps `poteto-mode` in context with Option+Enter / Alt+Enter). Invoke `poteto-mode` by name at the start of each task instead.

Next: [Route work through `poteto-mode`](./02-poteto-mode.md).
