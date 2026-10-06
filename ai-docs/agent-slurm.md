# BYU ORC Slurm and Node Guidance

Read this file before choosing nodes, submitting jobs, inspecting scheduler
state, or troubleshooting Slurm. The always-loaded rules in
[`BYU_ORC_AGENTS.md`](../BYU_ORC_AGENTS.md) still apply.

## Login and compute nodes

All cluster nodes run Linux. Login nodes are suitable for lightweight
interactive work such as editing and organizing files, inspecting source,
preparing job scripts, modest compilation, and submitting or monitoring jobs.
They are not suitable for sustained computation, large-scale processing,
intensive I/O, or hosting server processes. Internet downloads on login nodes
are permissible but should not use the proxy server.

Use Slurm when work will run for a substantial time, consume significant CPU or
memory, use GPUs, perform intensive I/O, or launch many processes. Never evade,
reset, or work around login-node CPU-time or other resource limits.

## Network access and proxy use

Login nodes can access the Internet without the compute-node proxy, so use them
for most downloads when practical. Compute nodes have an available proxy, but
it is slow and examines traffic, including decrypting TLS content for antivirus
scanning. Avoid large downloads through the proxy when possible: downloading on
a login node or before the job avoids tying up an expensive allocation.

`module load http_proxy` sets the relevant proxy environment variables. Apart
from minimal testing, load it only inside a job that needs it; do not use it on
login nodes, where it is unnecessary and slow. Follow any current ORC network
and software-download policies, and treat proxied traffic as inspected rather
than assuming end-to-end confidentiality through the proxy.

## Local configuration and commands

Slurm configuration is synchronized on each node under:

```text
/run/slurm/conf/
```

Inspect local configuration files when determining configured partitions, limits,
or scheduler behavior. Commands such as `squeue`, `sacct`, `scontrol`, `sinfo`,
`sprio`, and `sshare` make RPC requests to the Slurm controller. Use them only
when current job state, accounting data, or other dynamic information is
needed. Slurm commands are installed in `/apps/slurm/latest/bin/`.

Most partition access control here is handled through QOS. Run
`sacctmgr -nP list assoc where user="$USER" format=QOS` to display your full QOS
list, then compare it with the partitions’ `AllowQOS` entries in
`/run/slurm/conf/partitions.conf` to help identify partitions that accept one
of your QOSs. This check is only necessary if you need to request a specific
partition; otherwise, omit the partition and request only the resources you
need (e.g., CPUs or tasks, memory, time, and any required node features). Other
restrictions may still apply.

If Slurm appears broken, first check the command, path, configuration location,
shell startup, Lmod delays, timeout, and environment. A sandboxing issue or a
slow login-shell setup can look like a scheduler failure. Avoid repeated
retries that add controller load.

## If Slurm appears down or unavailable

Do not conclude that Slurm is down from a timeout or from `scontrol ping`
reporting both controllers as `DOWN`. Agent tools may start login shells whose
Lmod initialization takes several seconds; a short outer timeout can expire
before the Slurm command runs. Sandboxing can also block the client from
reaching the controller.

For a diagnostic probe, prefer a non-login shell and the absolute Slurm
binaries, for example:

```text
/apps/slurm/latest/bin/scontrol ping
/apps/slurm/latest/bin/squeue -h -u "$USER"
```

If a login shell is unavoidable, allow at least 15 seconds for startup. Check
`type squeue sinfo scontrol`: site shell functions may add a 15-second cache
layer, so absolute paths are preferable when current state matters. Do not run
many login-shell probes concurrently. If the direct probe works only outside
the sandbox, treat that as an execution-context problem and obtain the
appropriate approved execution context; do not report a Slurm outage or retry
rapidly.

## Resource requests and partitions

Every job should specify CPU core count, node count, memory, and a time limit;
always request memory, even for small expected use. Do not specify a partition
unless a real requirement makes it necessary. Leaving it unspecified gives
Slurm more eligible placement options, subject to hardware, time-limit, and
user eligibility.

While testing, it can be less wasteful to request somewhat more wall time than
anticipated so a job is not killed just before it finishes. Stay within the
partition and QOS limits, avoid excessive padding, and tighten the time limit
after measurements make the expected runtime clearer.

Specify features or constraints only when required. For example, `ib` may be
appropriate for a multi-node job that benefits from InfiniBand. Optional
constraints reduce eligible nodes and can increase queue time. Use the
`standby` QOS only when the user accepts preemption; requeueing may be suitable
where supported.

Prefer job arrays for large batches. Do not request all available memory by
default, GPUs for code that does not use them, many cores for single-threaded
code, or excessive wall time without justification.

FYI: GPU-node usage is “charged” at different rates by GPU generation; when
choosing among otherwise suitable GPU partitions, the partition’s
`TRESBillingWeights`—especially `GRES/gpu`—in
`/run/slurm/conf/partitions.conf` can help optimize usage. This is optional.

## Sizing, waiting, and short jobs

When practical, run a small representative test, measure runtime and memory,
adjust the request, and scale up carefully. Do not submit duplicates merely
because a job is pending, or repeatedly resubmit a failed job without finding
the cause.

For periodic monitoring, wait at least 60 seconds between queries, use longer
intervals for long-running or pending jobs, apply backoff, and avoid multiple
independent monitoring loops. Prefer job dependencies, `scontrol wait_job`,
`sbatch --wait`, output/error files, or a workflow system that controls
submission rate.

Aggregate short tasks by processing multiple inputs per job, batching task
lists, sizing array elements appropriately, running sequential tasks in one
allocation, or limiting submission rates. Target 10–30 minutes of useful work
per job when practical. Do not use `sleep` to inflate elapsed time.

## Long jobs and checkpointing

For jobs expected to run a day or more, consider checkpointing so work can
resume after wall-time limits, failures, maintenance, cancellation, or
preemption. Application-aware checkpointing is generally more reliable and
portable than application-agnostic methods. Early revisions do not always need
it; if a user reports multi-day runtimes or a job may exceed available limits,
recommend checkpoint-and-restart support. Until then, a concise `TODO` near the
main processing loop can be appropriate.
