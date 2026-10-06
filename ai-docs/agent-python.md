# BYU ORC Python Guidance

This document collects Python-specific guidance for using Python, managing
environments and packages, using Jupyter, and working with Python frameworks
such as PyTorch and TensorFlow. The always-loaded rules in
[`BYU_ORC_AGENTS.md`](../BYU_ORC_AGENTS.md) still apply.

General guidance for modules and installation, Slurm, performance, storage, and
security is maintained in the documents linked from `BYU_ORC_AGENTS.md` and is
not repeated here.

## Choosing the Python setup

A useful first step is to inspect the centrally provided software:

```bash
module avail python
module spider python
module spider miniforge3
```

A centrally provided Python module is often the simplest option when its version
and packages meet the project's needs:

```bash
module load python
python --version
which python
```

For a project-specific Python version or package set, a dedicated Mamba/Conda
environment through the site's Miniforge3 module is usually a good fit:

```bash
module load miniforge3
mamba create -n my_project python
mamba activate my_project
```

The operating system's Python is generally best reserved for system utilities,
rather than project work. A python module should generally be loaded and used
instead. It is also helpful to avoid mixing a Python module, a Conda environment, and
user-local package paths unless the combination is intentional and has been
verified. For consistency, a job, notebook, and interactive test can use the
same intended interpreter; `which python` and `python --version` are useful checks.

BYU's preferred environment manager is Conda, installed through Miniforge3,
with `conda-forge` as the normal package channel. See the current ORC
documentation for [Python](https://rc.byu.edu/wiki/?page=Python) and
[Conda Environments](https://rc.byu.edu/wiki/?id=Conda+Environments) for
site-specific details.

Users should generally prefer Miniforge3 over Anaconda Distribution, Miniconda3,
or any other Conda/Mamba installation configured to use Anaconda-maintained
`defaults` repositories, unless they have affirmatively verified that the
applicable Anaconda Terms of Service, licensing, institutional-use, and HPC-use
requirements have been reviewed and are satisfied. This caution concerns access
to Anaconda-maintained repositories, not Conda or Mamba as package-management
tools by themselves.

## Installing and recording packages

A separate environment for each workflow is usually easier to maintain. Packages
can be installed into the active environment, for example:

```bash
mamba install numpy scipy pandas
mamba list
```

Mamba/Conda packages are generally a good first choice because the solver
manages compiled dependencies together. If a needed package is unavailable
through Mamba/Conda, `pip` can be used inside the active project environment:

```bash
python -m pip install package_name
```

It is useful to record the environment with the project so it can be recreated:

```bash
mamba env export > environment.yml
mamba create -f environment.yml
```

Before committing or sharing an exported file, review it for machine-specific
paths, unnecessary packages, and credentials. Environment files are best kept
free of secrets and tokens unless necessary. Group environments also deserve some
care because group members may be able to alter them; a private environment may
be more appropriate when shared mutation is not intended.

Consider a user-controlled location appropriate to the environment's size and
lifespan; [`agent-storage.md`](agent-storage.md) covers the related storage
choices and tradeoffs.

## Python in Slurm jobs

[`agent-slurm.md`](agent-slurm.md) covers when to use Slurm, resource requests,
job sizing, and scheduler behavior. The Python-specific consideration is to
make the interpreter environment explicit in the job script, rather than relying
on the submission shell's inherited state:

```bash
module purge
module load miniforge3
mamba activate my_project

srun python my_script.py --input input.dat --output output.dat
```

The module and activation commands should match the environment used during
interactive testing. Package installation is generally better handled while
preparing the environment rather than repeated inside every job.

When reproducibility or troubleshooting matters, recording the interpreter and
important package versions in the job output can be useful:

```bash
which python
python --version
python -c 'import numpy; print(numpy.__version__)'
```

## JupyterLab and Open OnDemand

For interactive notebook work, use the current Open OnDemand and Slurm
guidance. For packages beyond the default JupyterLab installation, an
environment containing Python and JupyterLab can be created as follows:

```bash
module load miniforge3
mamba create -n my_jupyter python jupyterlab
mamba activate my_jupyter
mamba install numpy pandas
```

In the Open OnDemand JupyterLab form, the Python/JupyterLab environment setup
field can be enabled and configured to load the same environment before
launching:

```bash
module load miniforge3
mamba activate my_jupyter
```

The default Open OnDemand JupyterLab configuration may not include packages
such as NumPy. Follow the current [JupyterLab on Open OnDemand
documentation](https://rc.byu.edu/wiki/?id=JupyterLab+On+Open+OnDemand), since
the form and available modules may change.

## Scientific Python performance

The general principles in [`agent-performance.md`](agent-performance.md) apply
to Python. Python is often effective as an interface or orchestration layer,
while computationally intensive sections often benefit from libraries that
execute their hot paths in optimized native code or on a GPU. Large
element-by-element Python loops may be worth replacing with vectorized or
compiled implementations when measurement shows that they are a bottleneck.

For threaded numerical libraries or multiprocessing, use the allocation and
threading guidance in [`agent-performance.md`](agent-performance.md) rather
than choosing CPU counts or thread settings in isolation.

## GPU frameworks

When working with PyTorch, TensorFlow, or another GPU framework, a useful
sequence is:

1. Identify the framework, Python, and CUDA compatibility needed by the code.
2. Check the node's supported driver/CUDA version with `nvidia-smi` in the
   relevant allocation.
3. Select a compatible GPU build in the dedicated environment; the newest build
   is not necessarily the best match.
4. Verify inside the job that the framework sees the devices assigned to it.

For example, the current ORC PyTorch guidance uses Mamba/Conda environments,
selects a compatible CUDA build, and includes a GPU request in the job script.
The current [PyTorch documentation](https://rc.byu.edu/wiki/?page=PyTorch) and
[available software guide](https://rc.byu.edu/documentation/apps/guide/) are
useful references for available builds and examples. Since versions change,
these current pages are preferable to copying an old version-specific example.
Slurm GPU requests and allocation details are covered in
[`agent-slurm.md`](agent-slurm.md).
