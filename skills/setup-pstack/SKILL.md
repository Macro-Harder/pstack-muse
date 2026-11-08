---
name: setup-pstack
description: Configure pstack for Muse: effort level, review-panel sizes, and verification strictness. Muse runs one model, so there is no per-role model choice. Use for setup-pstack, "configure pstack", or changing pstack's effort and fan-out.
---

# Setup pstack

Muse runs one model, so upstream pstack's per-role model routing does not apply here.
There is nothing to detect and no model rule file to write. What is configurable in
this port is effort, fan-out, and verification strictness. Record the choices in
`~/workspace/user/pstack-muse-config.md` so later sessions can read them.

## Steps

### 1. Ask for an effort level

Offer these three options with these exact labels, using `muse.create_options`
for the bounded choice. With no config file, say that `standard` matches the
skill defaults.

- `light — short briefs, no review panels unless the work is contested`
- `standard — full playbooks, a 2-worker panel where a skill calls for one`
- `deep — thorough briefs, panels wherever a skill allows them`

### 2. Ask for panel sizes

Review panels (arena runners, interrogate reviewers, reflect reviewers) run as
same-model independent subagents in Muse, one per entry. Ask how many workers
each panel should use. Defaults: arena 2, interrogate 2, reflect 3. A shorter
panel runs fewer subagents and spends fewer tokens. Record the numbers.

### 3. Ask for verification strictness

- `standard` — prove it works against the real artifact before declaring done.
- `strict` — standard, plus a decision trail (`show-me-your-work`) on every
  multi-step task and an independent verification worker on every fix.

### 4. Write the config

Write `~/workspace/user/pstack-muse-config.md` with one line per choice, in this shape:

```
# pstack for Muse — configuration
effort: standard
arena runners: 2
interrogate reviewers: 2
reflect reviewers: 3
verification: standard
```

Overwrite the whole file so re-runs stay idempotent. A missing file means defaults.
A line the user deletes falls back to its default.

### 5. Confirm

Tell the user the config was written and that it applies to later sessions that
read it. Re-running this skill updates it.

### 6. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*`
skill, or an existing harness). If not, offer once: "want a project-local verification
skill, so agents can drive the app the way a user does and prove changes work? I can
generate one with the create-verification-skill skill." On yes, invoke
`create-verification-skill`. On no, move on without pushing.

### 7. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill
that runs a model checker, a proof, or another verifier against the code). If not,
offer once: "want a formal verification skill, so agents can prove a key property of
your code holds in every case in scope, not only the cases tests reach? I can generate
one with the create-formal-verification-skill skill." On yes, invoke
`create-formal-verification-skill`. On no, move on without pushing.

## What did not carry over

Upstream `/setup-pstack` detects the model slugs available to a `Task` subagent,
asks for a reasoning budget, and writes `~/.cursor/rules/pstack-models.mdc` mapping
each role to a model (code delegates to Grok, judgment to Opus 5.5). None of that
exists in Muse: there is one model, no per-role routing, no Task `model` parameter,
and no Cursor rules directory. The budget question became the effort question above,
and the role table became the panel sizes. The offer steps are unchanged.
