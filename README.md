# pstack for Muse (unofficial)

This repo is an **unofficial Muse port of [pstack](https://github.com/cursor/plugins/tree/main/pstack)
by Lauren Tan ([@poteto](https://github.com/poteto))**. pstack is developed in
[cursor/plugins](https://github.com/cursor/plugins). This port is **not affiliated with
or endorsed by** Lauren Tan or Cursor. All credit for pstack goes to its author.

- Upstream: https://github.com/cursor/plugins/tree/main/pstack
- Based on upstream commit [`6b0300214a09fd6029cfb7077043125758cd1fd2`](https://github.com/cursor/plugins/tree/6b0300214a09fd6029cfb7077043125758cd1fd2/pstack) (pstack version 0.15.36)
- The upstream pstack README is at [cursor/plugins/tree/main/pstack/README.md](https://github.com/cursor/plugins/tree/main/pstack/README.md).
- Questions or problems with this Muse port: please open an issue
  [in this repo](https://github.com/Macro-Harder/pstack-muse/issues), not upstream.

pstack is Lauren's answer to AI writing too much slop code: go deep first, write less but higher
quality code. This port keeps the methodology intact and changes only the tooling: every skill,
playbook, principle, guide page, agent, and automation was adapted from Cursor / Claude Code
tooling to Muse tooling. See [PORTING-NOTES.md](PORTING-NOTES.md) for the full mapping and for
anything deliberately left Cursor-specific.

## Install

Copy the skills into Muse's custom-skills directory:

```sh
git clone --depth 1 https://github.com/Macro-Harder/pstack-muse.git /path/to/pstack-muse
cp -r /path/to/pstack-muse/skills/poteto-mode ~/workspace/custom-skills/
```

Copy any other skill the same way, or copy the whole `skills/` directory. Each skill is a
directory with a `SKILL.md` and a `skill.yaml`; Muse reads them as custom skills.

## Get started

Two steps:

1. Run `setup-pstack`. In Muse there is no per-role model choice (Muse runs one model), so it
   configures effort and fan-out preferences instead of model slugs.
2. Use `poteto-mode` at the start of any non-trivial task. It matches the task to one of the
   twenty-three playbooks, opens a todo list with the playbook's steps, and routes to the other
   skills as the steps need them.

New here? The [pstack guide](docs/guide/README.md) walks through a first real task, adapted for
Muse. Stuck, or unsure which skill fits? Ask `poteto-help`.

## What changed for Muse

- Subagents: Cursor's `Task` tool with `subagent_type` and per-role `model` becomes Muse's
  `subagent.spawn`. All delegates run the same model (Muse). The `poteto-agent` wrapper is a
  brief that says "read poteto-mode in full first", not a subagent type.
- Todo lists: `todo.write`.
- Questions: asked in chat text; a bounded choice uses `muse.create_options`.
- Shell, files, diffs: `muse.exec`, `muse.read`, `muse.write`, `muse.edit`.
- Memory: `muse.memory_search` / `muse.memory_get`.
- Connectors and MCP: `muse.connector_search`.
- Model routing is gone. Multi-model review panels become same-model independent runs; agreement
  between two independent runs is still high signal.
- `/loop`, Custom Modes, Bugbot, Cursor Automations, and the Claude Code adapter have no Muse
  equivalent and are marked N/A in place, not silently dropped.

## License and attribution

pstack is Copyright (c) 2026 Lauren Tan ([@poteto](https://github.com/poteto)),
licensed under the MIT License. This repository is an unofficial derivative work
— a Muse port of pstack, originally published as part of
[cursor/plugins](https://github.com/cursor/plugins). The original license text
is reproduced in full in [LICENSE](LICENSE), kept verbatim as the MIT License
requires. This port is not affiliated with or endorsed by Lauren Tan or Cursor.
