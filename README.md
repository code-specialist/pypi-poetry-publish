# pypi-poetry-publish

Opinionated GitHub action to fully automate publishing packages to PyPI - using Poetry and GitHub releases.

> :information_source: We published this action because we use it in our projects and thought it would be useful to others as well. This action is **open to any kind of collaboration and 
> contribution** - We're happy to receive feedback, issues, pull requests or just kudos. :heart:

## Features

- Automatically publish your package to PyPI when you create a new release on GitHub
- Publish to the public PyPI, the TestPyPI, or a custom PyPI server
- Runs on private GitHub Actions runners
- Support for private PyPI repository dependencies
- Configurable **Python version**, **Poetry**, and **Poetry Core version**
- Configurable branch e.g. `main`, `master`, `beta`, etc.

> :information_source: If you do not use a custom runner, you may use the builtin functionality `GITHUB_TOKEN` with write permissions as the `ACCESS_TOKEN` as seen in the first example.
> See [https://docs.github.com/en/actions/security-guides/automatic-token-authentication#using-the-github_token-in-a-workflow](https://docs.github.
> com/en/actions/security-guides/automatic-token-authentication#using-the-github_token-in-a-workflow)

> :warning: We recommend you to use this workflow with the test PyPI registry e.g. `PUBLISH_REGISTRY: "https://test.pypi.org/simple/"` until you can confirm your workflow works as
> expected.

## Prerequisites

This action assumes you use [poetry](https://python-poetry.org/) as your package manager and have the `pyproject.toml` and `poetry.lock` files in the root directory of your
repository.

```
my_project/
├─ example_package/
│  ├─ __init__.py
├─ pyproject.toml
├─ poetry.lock
```

## Process

1. You create a new Release and Tag e.g. `1.0.0` (Trigger)
2. The action will:
	1. build your package
	2. adjust the version in the `pyproject.toml` and `__init__.py` in the package directory according to the tag
	3. publish the package to a PyPI registry

## Example process

### Before the release

**./pyproject.toml**

```toml 
[tool.poetry]
name = "code-specialist-example-package"
version = "0.1.0"
description = "Example package"
authors = ["Code Specialist"]
packages = [{include = "example_package"}]


[tool.poetry.dependencies]
python = "^3.10"
```

**./example_package/__init__.py**

```python
__version__ = "0.1.0"
```

### After GitHub Release 1.0.0

The action will alter the content of your repository to avoid conflicts and version mismatches:

**pyproject.toml**

```toml 
[tool.poetry]
name = "code-specialist-example-package"
version = "1.0.0" # adjusted to 1.0.0
description = "Example package"
authors = ["Code Specialist"]
packages = [{include = "example_package"}]


[tool.poetry.dependencies]
python = "^3.10"
```

**./example_package/__init__.py**

```python
__version__ = "1.0.0" # adjusted to 1.0.0
```

## Inputs

| Name                  | Description                                                                                               | Mandatory | Default                    |
|-----------------------|-----------------------------------------------------------------------------------------------------------|-----------|----------------------------|
| `PACKAGE_DIRECTORY`   | The directory the package is located in e.g. `./`, `./src/`                                               | ✓         |                            |
| `ACCESS_TOKEN`        | GitHub token with write access to the repository, to adjust the version                                   | ✓         |                            |
| `PYPI_PASSWORD`       | Either a password for the registry user or a token in combination with `__token__` as the `PYPI_USERNAME` | ✓         |                            |
| `PYPI_USERNAME`       | The username for the pypi registry                                                                        |           | `__token__`                |
| `POETRY_VERSION`      | The Poetry version to perform the build with                                                              |           | `1.1.8`                    |   
| `POETRY_CORE_VERSION` | The Poetry Code version to perform the build with                                                         |           | `1.0.4`                    |   
| `PYTHON_VERSION`      | The Python version to perform the build with                                                              |           | `3.10`                     |   
| `BRANCH`              | The branch to publish from                                                                                |           | `master`                   |
| `PUBLISH_REGISTRY`    | The registry to publish to e.g.`https://test.pypi.org/simple/`                                            |           | `https://pypi.org/simple/` |

## Example usage

Each example requires you to:

1. Create a workflow file e.g. `.github/workflows/publish.yaml`
2. Create a new release and tag e.g. `1.0.0` and the action will be triggered and publishes your package


> :information_source: If there is a use case you would like to see, please open an issue or a pull request.

### `publish.yaml` to publish to public PyPI

- Requires GitHub secrets:
	- `PYPI_PASSWORD` with a valid token

In order to use the `GITHUB_TOKEN` you also have to provide permissions for the token. In this case you require the `contents:write` permission.

```yaml
name: Build and publish python package

on:
  release:
    types: [ published ]

jobs:
  publish-service-client-package:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Publish PyPi package
        uses: code-specialist/pypi-poetry-publish@v1
        with:
          PACKAGE_DIRECTORY: "./example-package/"
          PYTHON_VERSION: "3.10"
          ACCESS_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PYPI_PASSWORD: ${{ secrets.PYPI_PASSWORD }}
```

### `publish.yaml` to publish to the test PyPI

- Requires GitHub secrets:
	- `PYPI_PASSWORD` with a valid token
	- `ACCESS_TOKEN` access token with write access to the GitHub repository

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
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          PYPI_PASSWORD: ${{ secrets.PYPI_PASSWORD }}
          PUBLISH_REGISTRY: "https://test.pypi.org/legacy/"
```

### `publish.yaml` to publish to a private PyPI

- Requires GitHub secrets:
	- `PYPI_USER` username for the PyPI registry
	- `PYPI_PASSWORD` with the password for the `PYPI_USER` user
	- `ACCESS_TOKEN` access token with write access to the GitHub repository

```yaml
name: Build and publish python package

on:
  release:
    types: [ published ]

jobs:
  publish-service-client-package:
    runs-on: ubuntu-latest
    steps:
      - name: Publish PyPI package
        uses: code-specialist/pypi-poetry-publish@v1
        with:
          PACKAGE_DIRECTORY: "./example-package/"
          PYTHON_VERSION: "3.10"
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          PYPI_PASSWORD: ${{ secrets.PYPI_PASSWORD }}
          PYPI_USER: ${{ secrets.PYPI_USER }}
          PUBLISH_REGISTRY: "https://pypi.code-specialist.com/simple/"
```

### `publish.yaml` to publish to a private PyPI with private dependencies

TODO