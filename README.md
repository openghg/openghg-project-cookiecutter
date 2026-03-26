# OpenGHG project cookiecutter

A small cookiecutter template for notebook-first projects that depend on
`openghg` and `openghg_inversions`, while still keeping reusable code in `src/`.

It creates a repo with:

- `src/` for importable Python code
- `notebooks/` for experiments
- `data/` for local datasets and generated artefacts
- `pyproject.toml` configured for `uv`
- optional Jupyter, Dask dashboard, `nbdime`, and `jupytext` extras
- a helper module for stable paths from notebooks
- a kernel installation script for the project `.venv`

## Using the template

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

## Examples

### 1) Project with reusable inversions or forward-model code

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

    from my_project.inversion.model import run_inversion
    from my_project.io.loaders import load_inputs

    inputs = load_inputs(...)
    result = run_inversion(inputs)

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

**Option A: bootstrap (recommended default)**

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



**Option B: editable install**

Install the package into the current environment:

    uv pip install -e .

or:

    pip install -e .

Then import normally:

    from my_project.flux import compute_flux

Pros:
- Clean imports
- Closer to real package usage



**Option C: dedicated kernel**

Create a kernel from the project environment:

    uv run python -m ipykernel install \
      --user \
      --name my-project \
      --display-name "Python (my-project)"

Then select it in Jupyter.



### Auto-reload during development

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
