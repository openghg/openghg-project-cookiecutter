# OpenGHG project cookiecutter

A small cookiecutter template for projects that depend on
`openghg` and `openghg_inversions`.

The directory layout includes places for notebooks and data; this is meant to make it easier to manage experiments with notebooks.

It creates a repo with:

- `src/` for importable Python code
- `notebooks/` for experiments
- `data/` for local datasets and generated artefacts
- `pyproject.toml` configured for `uv`
- optional Jupyter, Dask dashboard, `nbdime`, and `jupytext` extras
- a helper module for stable paths from notebooks
- a kernel installation script for the project `.venv`

## Using the template

When you run the template, it will prompt you for some metadata:

```
ab12345$ uvx cookiecutter openghg-cookiecutter-template/
Installed 21 packages in 111ms
  [1/5] repo_name (my-openghg-project): my-test-project
  [2/5] package_name (my_openghg_project): my_test_project
  [3/5] project_description (Notebook-first OpenGHG/OpenGHG Inversions project): A test of the template.
  [4/5] python_requires (>=3.11): 
  [5/5] author_name (Your Name): Brendan Murphy
```

To run the template you have a few options.

### Preferred: `uvx`

```bash
uvx cookiecutter gh:openghg/openghg-project-cookiecutter
```

Or you can clone this repo and use:
```bash
uvx cookiecutter /path/to/openghg-cookiecutter-template
```

### With `pipx`
```bash
pipx install cookiecutter
cookiecutter gh:openghg/openghg-project-cookiecutter
```

### With `pip`
```bash
python -m pip install cookiecutter
cookiecutter /path/to/openghg-cookiecutter-template
```

## Quickstart

### Environment and kernel setup

From inside your project:

```bash
uv sync

uv run python -m ipykernel install \
  --user \
  --name my-project \
  --display-name "Python (my-project)"
```

`uv sync` installs dependencies and makes your package importable in the
project environment.

Then use your code (in the `src` directory) in a notebook with:

```python
%load_ext autoreload
%autoreload 2

from my_project.flux import compute_flux
```

### Git + GitHub setup

#### Initialise git and push to GitHub (with gh CLI)

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main

gh repo create openghg/<repo_name> \
  --source=. \
  --remote=origin \
  --push
```

Use a different org (or omit the org prefix) if you want a non-OpenGHG target.

#### Fallback if GitHub CLI is not installed

If you do not have `gh` installed:

1. Go to https://github.com/new
2. Create a repository with the same name
3. Then run:

```bash
git remote add origin git@github.com:<USER_OR_ORG>/<repo_name>.git
git branch -M main
git push -u origin main
```

## Development workflow

- Put reusable code in `src/<package_name>/`
- Import it in notebooks
- Avoid copying code between notebooks

```python
from my_project.flux import compute_flux
```

## What to commit (and what not to)

Commit:
- `src/`
- `notebooks/`
- `pyproject.toml`
- `README.md`

Do NOT commit:
- `data/`
- `*.nc`
- `*.zarr`
- `.ipynb_checkpoints/`

`.gitignore` already handles this.

## Examples

### 1) Project that depends on `openghg` (and `openghg_inversions`)

Use this pattern when you expect code to outgrow a single notebook and want reusable, testable modules.

**How to work**
1. Put reusable code in `src/<package_name>/`
2. Keep notebooks focused on orchestration and plotting
3. Import from the package rather than copying code

**Example structure**

    src/
      my_project/
        inversion/
          model.py
          likelihood.py
        io/
          loaders.py
        diagnostics/
          plots.py

    notebooks/
      00_explore_inputs.ipynb
      10_run_inversion.ipynb

**Example usage**

``` python
    from my_project.inversion.model import run_inversion
    from my_project.io.loaders import load_inputs

    inputs = load_inputs(...)
    result = run_inversion(inputs)
```

This keeps logic reusable and makes it easier to move stable code into openghg or openghg_inversions.


### 2) Experiment-driven notebooks with gradual extraction to `src/`

This is the typical workflow:

1. Prototype in a notebook
2. Move stable functions into `src/<package_name>/`
3. Import them back into the notebook

**Typical flow**

    # Prototype in notebook
    def compute_flux(...):
        ...

    # Move to src/my_project/flux.py
    def compute_flux(...):
        ...

    # Import in notebook
    from my_project.flux import compute_flux



### Using `src/` code in notebooks

There are three common approaches.

**Option A: editable install (preferred)**

With `uv`, use:

    uv sync

or:

    pip install -e .

Use `pip install -e .` as a fallback for non-`uv` environments.

Then import normally:

    from my_project.flux import compute_flux

Pros:
- Clean imports
- Closer to real package usage



**Option B: bootstrap (fallback for HPC / existing kernels)**

The preferred approach is to use `uv sync` (editable install) with autoreload.

Add this near the top of the notebook:

    import sys
    from pathlib import Path

    ROOT = Path().resolve().parents[1]
    SRC = ROOT / "src"

    if str(SRC) not in sys.path:
        sys.path.insert(0, str(SRC))

Then:

    from my_project.flux import compute_flux

Pros:
- No environment changes
- Works well on HPC
- No kernel restart required



**Option C: dedicated kernel**

Create a kernel from the project environment:

    uv run python -m ipykernel install \
      --user \
      --name my-project \
      --display-name "Python (my-project)"

Then select it in Jupyter.

Pros:
- Can be used from any Jupyter Lab instance


### Auto-reload during development (important for options B and C)

This is recommended when using editable installs or a dedicated kernel.

To automatically reload code changes:

    %load_ext autoreload
    %autoreload 2

This reloads imported modules before each execution.

**Pitfalls**
- Existing objects may still use old definitions → re-run cells
- Class changes can behave inconsistently → recreate objects
- Large changes → restart kernel



### Alternative: %run (early prototyping only)

    %run ../src/my_project/flux.py

Useful for quick iteration, but:
- pollutes the namespace
- not representative of real usage
- harder to refactor later



### Suggested workflow

1. Start in a notebook
2. Move stable code into `src/`
3. Import it back into notebooks
4. Use `%autoreload 2` while iterating
5. Restart kernel when behaviour becomes unclear

This keeps notebooks flexible while building a reusable codebase.

## Appendix: Installing GitHub CLI (gh) on Rocky Linux

```bash
mkdir -p ~/bin
cd ~/bin

curl -L https://github.com/cli/cli/releases/download/vX.Y.Z/gh_X.Y.Z_linux_amd64.tar.gz | tar xz
cp gh_X.Y.Z_linux_amd64/bin/gh ~/bin/

export PATH="$HOME/bin:$PATH"

gh auth login
```

Replace `X.Y.Z` with the version you want to install. Add the `PATH` export to
your shell config (for example `~/.bashrc`) to make it persistent.

For shared setup instructions (uv, gh, HPC workflows), consider maintaining a
separate repository such as:

    openghg-tools

TODO (@brendan-m-murphy): decide whether to create and maintain `openghg-tools`
as the shared setup repository.
