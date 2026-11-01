# Repository Guidelines

## Project Structure & Module Organization

This repository contains Enrique Catalá’s Jekyll blog and portfolio, configured for GitHub Pages and `enriquecatala.com`.

- `_posts/` contains published Markdown articles; `_drafts/` holds unpublished content.
- `_portfolio/` contains portfolio entries; `tag/` contains tag landing pages.
- `_layouts/` defines page templates; `_includes/` holds shared Liquid/HTML fragments.
- `_sass/` contains styles imported through `css/main.scss`; `js/` contains site JavaScript.
- `img/` and `assets/` hold images, downloads, and Tipue Search resources.
- `_config.yml` controls site settings, plugins, pagination, and collections.
- `_site/` is generated output and must not be edited manually.

## Build, Test, and Development Commands

- `docker compose up` starts the configured Jekyll server with file watching at `http://localhost:4000`.
- `docker compose down` stops and removes the development containers.
- `docker compose run --rm jekyll jekyll build --safe` builds the site without starting the server.
- With Ruby and Bundler installed, use `bundle install` followed by `bundle exec jekyll serve` or `bundle exec jekyll build --safe`.

The Docker image declares Jekyll 4.2.2, while `Gemfile.lock` pins Jekyll 3.9.5 through `github-pages`. Check the actual runtime when investigating differences between local and hosted builds.

## Coding Style & Naming Conventions

Follow the surrounding file’s indentation and line endings; use two spaces for new YAML and HTML blocks. Keep reusable markup in `_includes/` and styles in `_sass/`.

Name new posts `YYYY-MM-DD-descriptive-slug.md`. Include YAML front matter matching nearby articles, such as `layout`, `title`, and `tags`. Preserve existing published URLs. Use descriptive image alt text and Jekyll URL filters for internal links. No formatter or linter configuration is established.

## Testing Guidelines

There is no dedicated automated test suite or coverage requirement. Build successfully, then inspect affected pages locally. Check navigation, images, mobile layout, and any changed search or pagination behavior. Verify that front matter parses and linked assets exist.

## Commit & Pull Request Guidelines

History uses short, informal action summaries such as `fix img` and `upgrade bio`; Conventional Commits are not required. Prefer specific summaries and keep changes focused.

For pull requests, describe the change, affected pages, and validation performed. Link relevant issues and include screenshots for visible layout changes. Exclude generated output and local runtime state from commits.
