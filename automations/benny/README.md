# benny

benny gives you two manual skills for slack issue reports. one triages each report. the other reproduces confirmed bugs and may prepare a small draft fix.

Cursor Automations has no Muse equivalent, so there is no automation runtime here: nothing watches the channel on its own and nothing wakes on a new report. You invoke the two skills by name, with the report coordinates, or set up a scheduled task that does. The triage and reproduce workflow itself is unchanged.

the files in this directory are dormant setup and skill sources. they do not appear as slash skills.

## set it up

1. point Muse at [`FOR_AGENTS.md`](./FOR_AGENTS.md) and name the target repository.
2. let setup merge this whole directory into the target at `benny/`. it must preserve destination-only files and review conflicts instead of overwriting local edits.
3. let setup confirm the shared pstack skills (`how`, `why`, `tdd`, `unslop`, and the principle skills benny uses) resolve from the installed pstack-muse location.
4. keep user-owned configuration outside the copied pack, for example in `.benny/`. adapt [`configuration.example.yaml`](./templates/configuration.example.yaml) and [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md).
5. commit `benny/` and any secret-free configuration before running either skill on live reports.
6. run a harmless test report through both skills by name and verify every source-channel post stays in the original thread.
