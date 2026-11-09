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
| `project-slug` | no | Project's slug on Read the Docs. Required unless using a project token. |
| `html` | yes | Directory containing the built HTML. |
| `api-url` | no | Base URL of the Read the Docs instance. Inferred from API tokens (`rtdcommunity_` or `rtdbusiness_` prefix) when omitted, otherwise defaults to Read the Docs Community. |
| `pdf` | no | Path to the PDF file to upload. |
| `epub` | no | Path to the ePub file to upload. |
| `htmlzip` | no | Path to the HTML zip file to upload. |
| `privacy-level` | no | Privacy level for the version (`public` or `private`). Read the Docs for Business only. |
| `version-name` | no | Override the inferred branch/tag name or pull request number. |
| `version-type` | no | Override the inferred version type: `branch`, `tag`, or `external`. |
| `commit` | no | Override the inferred commit hash. |
| `verbose` | no | Show debug output. Defaults to `false`. |

### API tokens

With a project token, the project and the Read the Docs platform are inferred from the token, so `project-slug` and `api-url` can be omitted:

```yaml
- uses: readthedocs/upload-action@v1
  with:
    token: ${{ secrets.READTHEDOCS_TOKEN }}
    html: _build/html
```

Tokens starting with `rtdcommunity_` upload to Read the Docs Community (`https://app.readthedocs.org`),
and tokens starting with `rtdbusiness_` upload to Read the Docs for Business (`https://app.readthedocs.com`).
The platform is inferred from tokens of any scope, but only project tokens infer the project, `project-slug` is required with other tokens.
`project-slug` and `api-url` still take precedence when given.

User tokens don't carry this information, `project-slug` is required and `api-url` defaults to `https://app.readthedocs.org` when using one.

### Version metadata

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
      - uses: actions/checkout@v5
      - uses: actions/setup-python@v6
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
      - uses: actions/setup-node@v6
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

Pin to the major version so you get fixes automatically:

```yaml
- uses: readthedocs/upload-action@v1
```

Dependabot can keep the pin up to date like any other action.

## License

[MIT](./LICENSE)
