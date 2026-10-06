# BYU ORC Software, Modules, and Installation Guidance

Read this file before installing packages, creating environments, compiling
software, or choosing a compiler, MPI, CUDA, Python, or scientific library.
The always-loaded rules in [`BYU_ORC_AGENTS.md`](../BYU_ORC_AGENTS.md) still
apply.

## Modules and environments

BYU HPC uses Lmod modules for much of its software, including Python,
compilers, MPI, CUDA, GPU tools, and scientific libraries. Inspect modules
first:

```bash
module avail
module spider <software>
module show <module>
```

Prefer module-provided software when it meets the project needs. A useful
preference order is:

1. Existing Lmod module
2. Existing project or user environment
3. Language-specific environment in user-controlled storage
4. User-local source build
5. Supported container or isolation mechanism

For Python, inspect modules before creating an environment and never modify the
system Python installation.

## Installation boundaries

Do not use `sudo`, install system-wide, replace system libraries, modify
operating-system-managed files, or modify shared environments without
authorization. Do not assume compute nodes have Internet access. Do not execute
an unreviewed remote installation script.

Move computationally intensive package builds and compilations into Slurm when
appropriate. Keep login-node compilation modest and lightweight.

Lmod setup and operations can be slow and can introduce several-second delays;
login shells may run Lmod setup again. Account for this when diagnosing command
timeouts or apparently unresponsive Slurm commands.
