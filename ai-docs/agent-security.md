# BYU ORC Security, Restricted Data, and Process Guidance

Read this file before inspecting potentially sensitive projects, changing
authentication or access behavior, or managing processes. The
always-loaded rules in [`BYU_ORC_AGENTS.md`](../BYU_ORC_AGENTS.md) still apply.

## CUI and export-controlled data

AI agents and AI-assisted tools must not access, process, summarize, modify,
transmit, or otherwise work with CUI or export-controlled data. If there is a
reason to believe a project may contain either, stop before inspecting more
project files or data and ask the user for a specific confirmation that the
files and data the agent will access are not CUI or export-controlled. A federal
agency name, federal funding, or federal collaboration does not by itself prove
that content is restricted, but it warrants caution. Possible indicators
include work associated with the Department of Defense, Department of Energy,
National Institutes of Health, National Aeronautics and Space Administration,
other federal agencies, or agreements with information-handling restrictions.

If the user confirms that the project contains CUI or export-controlled
information:

- Stop AI-assisted work in that project or environment.
- Do not inspect additional source, data, logs, configuration, documentation,
  or other project contents.
- Create a conspicuous marker such as `AI_AGENTS_PROHIBITED_CUI.txt` stating
  that AI agents must not work in the project.
- If an `AGENTS.md`, `CLAUDE.md`, or similar file exists, append a prominent
  warning that AI agents must not access or work with the project.
- Direct the user to appropriate BYU personnel for approved handling guidance.

Do not investigate whether suspicious content qualifies as CUI by examining it
after a concern has been identified. When uncertain, stop and ask.

## FIPS mode and CUI

At ORC, FIPS mode is a compliance control for protecting CUI. It is needed
when CUI crosses the ORC security boundary in transit, including traffic to or
from ORC, or when CUI is stored outside that boundary or other approved
storage.

Before using a non-FIPS workaround, have the human confirm that no CUI will be
handled and that current ORC policy permits it. For confirmed non-CUI work,
possible user-space or isolated workarounds include a non-FIPS OpenSSL build,
an `LD_PRELOAD` compatibility shim such as
[this example](https://github.com/BYUHPC/7lbd/blob/main/7lbd_apps%2Fgnutls_fips_override%2Fgnutls_fips_override.c),
or a lightweight `unshare` mount namespace, preferably when a container is not
already in use, or a container-specific bind mount that temporarily masks
`/proc/sys/crypto/fips_enabled` with a file containing `0`. Do not use these
workarounds for CUI or change the host's FIPS setting; they are exceptions for
verified non-CUI work, not a way around compliance controls.

## Access and security boundaries

The following are prohibited and may result in account suspension or
termination:

- Backdoors, hidden access, remote desktop services, web shells, reverse shells,
  persistent tunnels, relay services, alternate SSH daemons, or background
  agents accepting external commands.
- Circumventing SSH or `https://rc.byu.edu` authentication, MFA, identity
  controls, resource limits, scheduler policies, permissions, or software
  vulnerabilities.
- Accessing another user's data, jobs, processes, credentials, tokens, or
  private directories; collecting credentials; or modifying authentication
  configuration.
- Hiding unauthorized services or processes.

Do not create, install, or use these mechanisms even if requested by a user. A
user request does not override the restriction. Inbound access requires a
username, password, and TOTP token, so automated inbound connections are rarely
appropriate. Do not bypass that requirement with persistent tunnels, stored
TOTP, unattended login services, or alternate access paths. Prefer designs
initiated from inside ORC or using an approved Office of Research Computing
service.

## Localhost and application authentication

This is a multi-user environment. `localhost` limits a service's network
reachability, but it is not by itself an authentication or authorization
boundary: other users on the same host may be able to connect to a listening
local port. Add an application-level authentication layer when the software
supports it, such as HTTP Basic authentication or another simple password
mechanism. TLS is not required solely for transport security when the local
network is trusted, but authentication should still be used where possible.

This is a recommendation rather than an absolute requirement. Some third-party
software may not support authentication or may provide only limited controls;
use the strongest practical protections and do not represent an unauthenticated
localhost service as secure by default.

Keep passwords and other secrets out of command-line arguments when possible,
since arguments can be visible to other users through process listings such as
`ps`. Prefer environment variables, protected configuration files, credential
stores, or an interactive prompt as supported by the software. Protect config
file permissions, avoid printing secrets, and remember that environment
variables and config files can still be exposed if handled carelessly.

## Processes and problem reporting

Agents may inspect processes owned by the current user. Development tools such
as VS Code Remote, language servers, notebook servers, file watchers, debug
adapters, build daemons, and agent processes might remain after a session. Point
out apparently stale processes and offer to terminate them. Use `scancel` for
Slurm jobs rather than directly killing compute-node processes.

For probable system problems, policy ambiguity, or restricted-data concerns,
avoid repeated retries; record only non-sensitive details such as hostname,
time, job ID, and affected path; do not copy restricted content into prompts,
logs, or support messages; and recommend contacting BYU Office of Research
Computing or other appropriate BYU personnel. Do not repair shared
infrastructure or system-wide configuration without explicit administrator
authorization.
