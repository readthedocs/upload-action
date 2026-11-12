# Changelog

## v1.0.0 (unreleased)

- Initial composite action that uploads pre-built documentation to Read the Docs
  by running the [readthedocs-cli](https://github.com/readthedocs/readthedocs-cli).
- Infer the project and the Read the Docs platform from project tokens.
  `project-slug` is now optional when using a project token.
- Runs `readthedocs-upload==1.0.0a1` from PyPI.
