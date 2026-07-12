# Contributing

Thanks for your interest in improving the Dispatcharr Plugin Workflow guide.

## Scope

This repository is a **guide**, not a plugin. Contributions should improve the accuracy, clarity, or completeness of the workflow documentation. Issues with the plugins themselves belong in their respective repositories — see the [Plugin References](reference/plugin-links.md) table.

## Ways to Contribute

- **Corrections.** Fix outdated steps, broken links, wrong defaults, or anything that no longer matches current plugin behavior.
- **Clarifications.** Reword sections that are confusing or ambiguous.
- **Screenshots.** UI captures of plugin configuration screens or output. Place them in the `docs/screenshots/` folder and reference them from the relevant page. (It must be `docs/screenshots/` — a file placed anywhere else is not built into the site and the image will 404.)
- **New sections.** Workflow tips, troubleshooting notes, or additional plugin coverage that fits the guide's scope.

## How to Submit

1. Fork the repository.
2. Create a branch for your change.
3. Make the edit. Keep the existing tone (no contractions, step-numbered actions, tables for option lists).
4. Open a pull request describing what changed and why.

## Previewing Locally

The site is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

```bash
pip install mkdocs-material mkdocs-git-revision-date-localized-plugin
mkdocs serve
```

Open `http://127.0.0.1:8000/` in your browser. The site auto-reloads when you save a Markdown file.

## Style Notes

- Use GitHub-flavored Markdown.
- Match the heading hierarchy used in the existing pages.
- When referencing plugin actions, use the exact button label as it appears in the Dispatcharr UI.
- Verify any default values or paths against the current plugin source before changing them.

## Reporting Issues

Open an issue if you find something wrong but cannot fix it yourself. Include the page, the problem, and (if possible) the corrected version.
