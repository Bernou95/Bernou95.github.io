# Bernou95.github.io

Personal site of Jose Bernardo Martinez Morales, built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).

- Edit pages in `docs/` (`index.md` = About, `projects.md` = Projects); images and videos live in `docs/assets/`.
- Preview locally: `pip install mkdocs-material && mkdocs serve`, then open http://127.0.0.1:8000
- Publish: push to `main`. The GitHub Action in `.github/workflows/deploy.yml` builds the site into the `gh-pages` branch.
