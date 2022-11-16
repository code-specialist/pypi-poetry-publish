# pypi-poetry-publish

Opinionated GitHub action to fully automate publishing packages to PyPI - using Poetry and GitHub releases. This action assumes you use [poetry](https://python-poetry.org/) as
your package manager and have the `pyproject.toml` and `poetry.lock` files in the root directory of your repository.

## Process

1. You create a new Release and Tag e.g. `1.0.0` (Trigger)
2. The action will:
	1. build your package
	2. adjust the version in the `pyproject.toml` and `__init__.py` in the package directory according to the tag
	3. publish the package to a PyPI registry

## Inputs

| Name                   | Description                                                                                               | Mandatory | Default                    |
|------------------------|-----------------------------------------------------------------------------------------------------------|-----------|----------------------------|
| `PACKAGE_DIRECTORY`    | The directory the package is located in e.g. `./`, `./src/`                                               | ✓         |                            |
| `ACTIONS_ACCESS_TOKEN` | GitHub token with write access to the repository, to adjust the version                                   | ✓         |                            |
| `PYTHON_VERSION`       | The Python version to perform the build with e.g. `3.10`                                                  | ✓         |                            |   
| `PYPI_PASSWORD`        | Either a password for the registry user or a token in combination with `__token__` as the `PYPI_USERNAME` | ✓         |                            |
| `PYPI_USERNAME`        | The username for the pypi registry                                                                        |           | `__token__`                |
| `BRANCH`               | The branch to publish from                                                                                |           | `master`                   |
| `REGISTRY`             | The registry to publish to e.g.`https://test.pypi.org/simple/`                                            |           | `https://pypi.org/simple/` |

## Example usage

Each example requires you to:

1. Create a workflow file e.g. `.github/workflows/publish.yml`
2. Create a new release and tag e.g. `1.0.0` and the action will be triggered and publishes your package

### publish.yaml to publish to public pypi

- Requires GitHub secrets:
	- `PYPI_PASSWORD` with a valid token
	- `ACTIONS_ACCESS_TOKEN` access token with write access to the GitHub repository

```yaml
name: Build and publish python package

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
          PYPI_PASSWORD: ${{ secrets.PYPI_PASSWORD }}
```

### publish.yaml to publish to the test pypi

- Requires GitHub secrets:
	- `PYPI_PASSWORD` with a valid token
	- `ACTIONS_ACCESS_TOKEN` access token with write access to the GitHub repository

```yaml
name: Build and publish python package

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
          PYPI_PASSWORD: ${{ secrets.PYPI_PASSWORD }}
          REGISTRY: "https://test.pypi.org/legacy/"
```

### publish.yaml to publish to a private pypi

- Requires GitHub secrets:
	- `PYPI_USER` username for the pypi registry
	- `PYPI_PASSWORD` with the password for the `PYPI_USER` user
	- `ACTIONS_ACCESS_TOKEN` access token with write access to the GitHub repository

```yaml
name: Build and publish python package

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
          PYPI_PASSWORD: ${{ secrets.PYPI_PASSWORD }}
          PYPI_USER: ${{ secrets.PYPI_USER }}
          REGISTRY: "https://pypi.code-specialist.com/simple/"
```

