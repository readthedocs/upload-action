# Read the Docs Upload action

Upload pre-built documentation artifacts to [Read the Docs](https://readthedocs.com/) from GitHub Actions.

Build your documentation with whatever tool you use, then hand the output to this action. It resolves the Git metadata (branch, tag, or pull request commit) from the workflow environment and uploads the artifacts. Under the hood it runs the [readthedocs-cli](https://github.com/readthedocs/readthedocs-cli).

```yaml
- uses: readthedocs/upload-action@v1
  with:
    token: ${{ secrets.READTHEDOCS_TOKEN }}
    project-slug: my-project
    html: _build/html
```

## Inputs

| Input | Required | Description |
|---|---|---|
| `token` | yes | Read the Docs API token. Store it as a repository secret. |
| `project-slug` | yes | Project's slug on Read the Docs. |
| `html` | yes | Directory containing the built HTML. |
| `api-url` | no | Base URL of the Read the Docs instance. Defaults to Read the Docs Community. |
| `pdf` | no | Path to the PDF file to upload. |
| `epub` | no | Path to the ePub file to upload. |
| `htmlzip` | no | Path to the HTML zip file to upload. |
| `privacy-level` | no | Privacy level for the version (`public` or `private`). Read the Docs for Business only. |
| `version-name` | no | Override the inferred branch/tag name or pull request number. |
| `version-type` | no | Override the inferred version type: `branch`, `tag`, or `external`. |
| `commit` | no | Override the inferred commit hash. |
| `verbose` | no | Show debug output. Defaults to `false`. |

The version name, type, and commit are inferred from the workflow event, including the correct head commit for pull requests. Set the overrides only when you need to.

## Examples

Build with Sphinx and upload the HTML:

```yaml
name: Docs

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:

jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1  # v7.0.1
      - uses: actions/setup-python@5fda3b95a4ea91299a34e894583c3862153e4b97  # v7.0.0
        with:
          python-version: '3.13'
      - run: pip install -r docs/requirements.txt
      - run: sphinx-build -b html docs/ _build/html

      - uses: readthedocs/upload-action@v1
        with:
          token: ${{ secrets.READTHEDOCS_TOKEN }}
          project-slug: my-project
          html: _build/html
```

Also upload downloadable formats by pointing at each file:

```yaml
      - uses: readthedocs/upload-action@v1
        with:
          token: ${{ secrets.READTHEDOCS_TOKEN }}
          project-slug: my-project
          html: _build/html
          pdf: _build/latex/my-project.pdf
          epub: _build/epub/my-project.epub
```

A project that does not build with Python. The upload step still needs no interpreter setup of its own:

```yaml
      - uses: actions/setup-node@820762786026740c76f36085b0efc47a31fe5020  # v7.0.0
        with:
          node-version: '22'
      - run: npm ci && npm run build

      - uses: readthedocs/upload-action@v1
        with:
          token: ${{ secrets.READTHEDOCS_TOKEN }}
          project-slug: my-project
          html: dist
```

## Pull requests from forks

GitHub does not expose secrets to workflows triggered by pull requests from forks, so the `token` will be empty and the upload will fail with an explanation. This is expected for now and affects preview builds on public projects.

## Versioning

Releases follow semantic versioning and are tagged `vX.Y.Z`. Pin to the full commit hash of a release and note the version in a comment, as [GitHub recommends](https://docs.github.com/en/actions/reference/security/secure-use#using-third-party-actions). Dependabot keeps the pin up to date like any other action:

```yaml
- uses: readthedocs/upload-action@<commit hash>  # v1.0.0
```

The `v1` tag moves to the latest `v1.x.y` release, so pinning to it gets fixes automatically at the cost of running code you did not review:

```yaml
- uses: readthedocs/upload-action@v1
```

The action itself follows the same standard: the actions it uses are pinned to a commit hash, and it runs an exact [readthedocs-upload](https://pypi.org/project/readthedocs-upload/) release from PyPI. The pinned version is in `action.yml`, and the [changelog](./CHANGELOG.md) lists which one each release ships.

## License

[MIT](./LICENSE)
