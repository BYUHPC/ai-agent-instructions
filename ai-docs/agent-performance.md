# BYU ORC Performance and Resource-Efficiency Guidance

Read this file when writing code, especially when performance, parallelism,
resource requests, or I/O-heavy code apply. The always-loaded rules in
[`BYU_ORC_AGENTS.md`](../BYU_ORC_AGENTS.md) still apply.

HPC resources are shared and competitively scheduled. Inefficient code can
increase runtime, consume unnecessary CPU, GPU, memory, or storage resources,
reduce fair-share priority, and burden other users.

Prioritize correctness, safety, and reproducibility. When performance matters,
measure a representative workload, identify the actual bottleneck, and make a
targeted change. Prefer algorithm, data-structure, I/O, and library improvements
before adding parallelism or low-level tuning. Do not introduce parallelism
merely because resources are available.

## Interpreted languages

For computationally intensive Python or other interpreted-language work,
prefer established libraries that execute performance-critical operations in
optimized native code. Examples include NumPy, SciPy, PyTorch, JAX, Numba,
Cython, Polars, and suitable domain-specific libraries.

Avoid large element-by-element interpreted loops when a practical vectorized
or compiled implementation exists.

## Parallel code

Before requesting multiple CPUs or GPUs, verify that the application can use
them effectively. Match thread counts to the allocation, avoid nested
oversubscription, prevent libraries from creating more threads than requested,
and distinguish threading, multiprocessing, MPI, and GPU parallelism. Benchmark
scaling before requesting large allocations.

For communication between processes on multiple nodes, prefer MPI and use a
supported MPI module.

Common thread-control variables include:

```text
OMP_NUM_THREADS
OPENBLAS_NUM_THREADS
MKL_NUM_THREADS
NUMEXPR_NUM_THREADS
```

Values should normally derive from the Slurm allocation rather than be
hard-coded.

## Performance claims

Do not claim that code is optimized without measurement. When performance
matters, establish a representative baseline, profile the workload, identify
the actual bottleneck, make a targeted change, and measure the result.
