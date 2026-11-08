# Reproduce run brief

> Source material for a manual repro run or a scheduled-task brief. Paraphrase this intent into the invocation after confirming that the copied pack is committed in the repository where the skill will run. There is no automation runtime in Muse: the operator invokes the skill by name with the report coordinates, or a scheduled task the user set up polls the channel and does.

Read and follow `benny/skills/reproduce-and-fix-issues/SKILL.md` for this run.

Configuration source. Include this repository-relative path only when it is committed in the same target repository. Otherwise paraphrase the configured values. Never use a plugin source or cache path:

```text
{{BENNY_CONFIG_PATH}}
```

Report:

```json
{
	"source_channel_id": "{{SLACK_CHANNEL_ID}}",
	"message_ts": "{{SLACK_MESSAGE_TS}}",
	"thread_ts": "{{SLACK_THREAD_TS_OR_EMPTY}}"
}
```

The brief should describe this as a new top-level report in the configured source Slack channel. It should include the configured repository, default branch, issue tracker, control adapter, feature map, and draft pull request capability.

Treat the source channel and root thread timestamp as immutable. If either is missing or does not match configuration, stop without posting.

Wait for a configured triage marker from the configured triage identity in this exact thread. Proceed only for `[benny:bug]` or `[benny:performance]`. In Muse there is no scheduled wake, so "wait" means: check the thread when the run starts and, if the verdict has not landed yet, report that the marker is missing and stop; the operator or a later scheduled run invokes the skill again.

Require the configured control-adapter skill before attempting a repro. Reproduce the exact discriminating symptom twice through the real UI. Verify existing pull requests or commits without authoring over them. Attempt a bounded fix only after a confirmed repro and the operational file's fix gate.

The coordinator is the only Slack poster. Every child prompt must forbid Slack write actions (the messaging connector's post/send tools) in every form. Children return findings only.

Never post a root message in the source channel.
