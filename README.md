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
uvx cookiecutter /path/to/openghg-cookiecutter-template
```

### With `pipx`
```bash
pipx install cookiecutter
cookiecutter /path/to/openghg-cookiecutter-template
```

### With `pip`
```bash
python -m pip install cookiecutter
cookiecutter /path/to/openghg-cookiecutter-template
```

You can also point cookiecutter at a Git repository once you put this template on GitHub:

```bash
uvx cookiecutter gh:YOUR-ORG/openghg-project-cookiecutter
```
