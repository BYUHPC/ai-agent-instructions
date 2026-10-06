# BYU ORC Storage, I/O, and Transfer Guidance

Read this file before selecting storage, creating many files, recursively
scanning a large path, or moving substantial data. The always-loaded rules in
[`BYU_ORC_AGENTS.md`](../BYU_ORC_AGENTS.md) still apply.

## Choose storage deliberately

BYU ORC provides network filesystems with different purposes and policies.
Consult the current storage documentation before choosing a location:

```text
https://rc.byu.edu/wiki/?id=Storage
```

Shared filesystems are sensitive to large quantities of small files, excessive
metadata operations, repeated scans, frequent creation/deletion, small reads
or writes, and highly concurrent access to one directory.

Reduce the number of files when practical; avoid thousands or millions in one
directory; avoid repeatedly scanning large trees; cache reusable metadata;
limit concurrent metadata-heavy processes; and use buffered or block-oriented
I/O. Prefer operations of at least 64 KiB and, where practical, several
megabytes.

Before a large recursive operation, estimate its scope and consider a narrower
alternative. Repeated commands such as these can be harmful on large paths:

```bash
find /large/path -type f
du -a /large/path
ls -lR /large/path
```

## Small-file workloads

When a workflow creates, stores, or transfers a large number of files,
especially small files, agents should normally recommend bundling them into
larger container files—often compressed archives such as tar—and unpacking them
only when needed. This is the default recommendation, not a strict requirement:
keep files separate when the access pattern or tooling requires it. Other
formats suited to particular workloads include HDF5, Zarr with an appropriate
layout, SQLite, Parquet, LMDB, or an application-specific container. Archives
may be unsuitable for frequent independent or concurrent access.

## Temporary data

Where supported, put temporary or high-I/O intermediate data on storage
intended for that purpose rather than repeatedly using shared home or project
storage. Verify capacity, purge behavior, persistence, and backup policy first.
`/tmp` is node-local: files created there are not automatically visible from
other nodes and are not durable storage. It is often best for job-specific
temporary data, but its space is shared with other jobs, so use it cautiously
and copy anything that must survive the job or be read elsewhere.

## Tape Storage

BYU ORC Tape Storage is a paid archival service for large datasets that need
long-term retention but are not accessed regularly. It is inexpensive and
heavily subsidized for BYU research needs. Transfers use Globus; bundle many
small files into larger archives whenever possible; keep them separate only
when the workflow requires it. Globus may handle POSIX permissions and
ownership differently from other transfer utilities, so do not assume they
will be preserved identically.

## Transfers

BYU ORC filesystems are not directly mounted or exported outside the HPC
environment. Use an approved transfer path based on the task:

- **Globus** — Usually the best choice for large or long-running transfers,
  cross-institution transfers, and transfers involving many files. Use the
  Globus web interface or command line. **Globus Connect Personal** is licensed
  for BYU use and can connect personal or departmental computers to Globus for
  transfers to and from ORC.
- The Globus CLI may be useful for shell-based or scripted transfers when its
  asynchronous tasks, batching, checksums, and monitoring are valuable. Its
  authentication and consent setup may be more involved, and preservation of
  Unix permissions or other metadata may differ from SSH-based tools, so
  consider it selectively rather than as a general replacement. Use the
  Globus API or SDK primarily for application integration and specialized
  automation.
- **Open OnDemand** — Best for quick, occasional, web-based transfers, such as
  downloading or viewing images generated on ORC systems.
- **SSH-based tools** — `scp`, `sftp`, `rsync`, and similar tools are useful for
  direct, scripted, or repeatable transfers. `rclone` is especially useful for
  cloud-storage workflows.

SSHFS is permitted but is less reliable than direct transfer tools and is not
generally recommended. Do not configure it to auto-reconnect: after a
disconnect it cannot answer the second-factor prompt, may retry repeatedly, and
can cause the user's IP address to be blocked.

For transfers initiated from outside ORC, the supported inbound access paths
are SSH, Globus, and Open OnDemand. Other transfer methods may generally be
used when initiated from ORC systems, provided they comply with BYU ORC usage
policies and the policies of the remote site. Large or long-running transfers
should use verification and resume support where available.
