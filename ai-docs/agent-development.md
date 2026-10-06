# BYU ORC Development Guidance

Read this file before writing or modifying code, scripts, workflows, or
development documentation. The always-loaded rules in
[`BYU_ORC_AGENTS.md`](../BYU_ORC_AGENTS.md) still apply.

Use login nodes for lightweight editing, source inspection, and modest
development tasks. Use Slurm for sustained computation, significant CPU or
memory use, GPUs, intensive I/O, or many processes. See
[`agent-slurm.md`](agent-slurm.md) for resource requests and scheduler
behavior, [`agent-software.md`](agent-software.md) for modules and installation,
and [`agent-storage.md`](agent-storage.md) for storage and I/O choices.

## Context and token efficiency

These are very soft recommendations for AI-assisted work. When practical,
consider minimizing tool calls and context growth:

- If an exact file path is known, read it directly rather than checking for it
  first with `ls`, `find`, `rg --files`, `test -f`, or similar commands; use
  failure-driven discovery if the direct read fails.
- Avoid repeatedly inspecting repository structure once it is known.
- Prefer targeted searches over repository-wide searches.
- Limit command output to what is needed for the next decision; concise forms
  such as `git status --short`, targeted `git diff`, and filtered test output
  are often sufficient.
- Avoid rereading unchanged files when the existing context is sufficient.
- For large files, read only relevant ranges when their location is known.
- Avoid sending large logs, generated files, lockfiles, build artifacts,
  session histories, or unrelated repository content to an LLM.
- On successful commands, retain only the information needed for subsequent
  work.

When practical, consider converting documents to Markdown locally with
[MarkItDown](https://github.com/microsoft/markitdown) or a similar tool before
sharing only the needed portion. For sensitive input, a network-isolated
wrapper such as `unshare -Unr` can help ensure the local conversion process
cannot send data outside the environment; verify that the tool works locally
before use.

## Testing and validation

Use validation appropriate to the change. Unit tests and regression tests are
recommended for development when practical, especially for reusable code or
behavior that is likely to change; this is guidance rather than a requirement
for every change. Linting, type checks, smoke tests, example runs, or manual
review may also be appropriate.

For computational workflows, when practical run a small representative test,
measure runtime and memory, adjust the resource request, and scale up
carefully. Keep resource-intensive test suites and benchmarks in Slurm rather
than on login nodes. Do not repeat expensive runs without using the results to
diagnose a failure or improve the request.

## Version control and notes

Where practical, use version control such as Git to track changes to code and
documents. Keep commits focused and avoid committing credentials, tokens,
generated data, or machine-specific paths. Consider maintaining a lightweight
lessons-learned file. Record each lesson as a short, standalone, actionable
entry with enough context to be useful without restating the entire workflow.
