# pypi-poetry-publish

Opinionated GitHub action to fully automate publishing packages to PyPI - using Poetry and GitHub releases. This action assumes you use [poetry](https://python-poetry.org/) as
your package manager and have a `pyproject.toml` file in the root directory of your repository.

## Process

1. You create a new Release and Tag e.g. `1.0.0` (Trigger)
2. The action will:
	1. build your package
	2. adjust the version in the `pyproject.toml` and `__init__.py` in the package directory according to the tag
	3. publish the package to the PyPI registry

## Inputs

| Name                   | Description                                                             | Mandatory | Default  |
|------------------------|-------------------------------------------------------------------------|-----------|----------|
| `PACKAGE_DIRECTORY`    | The directory the package is located in e.g. `./`, `./src/`             | ✓         |          |
| `ACTIONS_ACCESS_TOKEN` | GitHub token with write access to the repository, to adjust the version | ✓         |          |
| `PYTHON_VERSION`       | The Python version to perform the build with e.g. `3.10`                | ✓         |          |   
| `PYPI_TOKEN`           | A token from the pypi registry to publish the package                   | ✓         |          |
| `BRANCH`               | The branch to publish from                                              |           | `master` |

## Example usage

1. Create a workflow (e.g. `.github/workflows/publish.yml` ) with the following content:

```yaml
name: Build and publish service client

on:
  release:
    types: [ published ]

jobs:
  publish-service-client-package:
    runs-on: ubuntu-latest
    steps:
      - name: Publish PyPi package
        uses: code-specialist/pypi-poetry-publish@v1
        with:
          PACKAGE_DIRECTORY: "./example-package/"
          PYTHON_VERSION: "3.10"
          ACTIONS_ACCESS_TOKEN: ${{ secrets.ACTIONS_ACCESS_TOKEN }}
          PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}
          BRANCH: "master"
```

2. Create a new release and tag e.g. `1.0.0` and the action will be triggered and publishes your package