# jonaslindemann.github.io

This site is built with Zensical and published to GitHub Pages through GitHub Actions.

## Deployment

The workflow in [.github/workflows/deploy-pages.yml](.github/workflows/deploy-pages.yml) builds the site in a Conda environment named `zensical-env` and publishes the generated output to the `gh-pages` branch.

To keep the deployment working, make sure:

- the repository has GitHub Pages enabled,
- the Pages source is set to the `gh-pages` branch,
- and the workflow has permission to push to that branch.
