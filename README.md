# Architecture Review

A Claude Code plugin that scans any codebase and generates an interactive HTML architecture diagram with plain-English explanations and risk analysis.

Built for PMs, founders, and vibecoders who want to understand what a system looks like without reading every file.

## What you get

- **Interactive architecture diagram** -- components grouped by category (frontend, API, database, auth, etc.) with connection tags showing how they relate
- **Risk analysis** -- flags like "sessions stored in primary DB" or "no caching on read-heavy endpoints", calibrated to your project's scale
- **Plain-English explanations** -- every component and risk explained with real-world analogies, not jargon

## Install

### From a marketplace

If the marketplace is already configured:

```
/plugin install architecture-review
```

### From GitHub

```
/plugin marketplace add lucycrx/architecture-review
/plugin install architecture-review
```

### Local testing

```bash
claude --plugin-dir ./architecture-review
```

## Usage

Open any project with Claude Code, then run:

```
/architecture-review:architecture-review
```

Claude will scan the codebase and open an `architecture-review.html` file in your browser.

## What it detects

**Component categories:** Frontend, API layer, Database, Cache, Auth, External services, Background jobs, Storage

**Risk patterns:** Session storage, missing caching, blobs in database, and more -- each with severity ratings, scale thresholds, and suggested fixes

## Supported frameworks

Next.js, Express, Django, Rails, FastAPI, Laravel, Spring Boot, and most common web frameworks. Detection is file-based, so it works with any project that follows standard conventions.

## License

MIT
