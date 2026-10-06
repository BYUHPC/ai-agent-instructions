# BYU ORC Cron and Periodic-Task Guidance

Read this file before using, modifying, or recommending `cron`. The
always-loaded rules in [`BYU_ORC_AGENTS.md`](../BYU_ORC_AGENTS.md) still apply.

Cron is limited to lightweight periodic tasks. It must not perform sustained
computation, intensive I/O, frequent Slurm queries, large recursive scans, or
work that belongs in Slurm.

For lightweight periodic work that should run through Slurm, suggest using
`scrontab`, Slurm's scheduled-job facility, instead of regular `cron`. As a
general scheduling guideline, suggest intervals no more frequent than once
every six hours, and typically no more frequent than daily. This is guidance,
not a strict requirement; choose an interval appropriate to the task while
avoiding unnecessary scheduler and filesystem load.

In every chat session where an agent uses, modifies, or recommends `cron` or
`scrontab`, it must first list the user's existing `cron` and `scrontab`
entries. Ask whether each entry is still required, preferably one at a time.
This review repeats in every such session. Do not remove, disable, or modify an
existing entry without approval.
