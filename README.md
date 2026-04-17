# Architecture Review

A Claude Code skill that scans any codebase and generates an interactive HTML architecture report with flow diagrams, component cards, plain-English explanations, and risk analysis.

Built for PMs, founders, and vibecoders who want to understand what a system looks like without reading every file.

## What you get

- **Interactive wizard** -- asks about your goal, preferred diagram style, and whether to include risk analysis before scanning
- **SVG flow diagram** -- interactive left-to-right data flow visualization with color-coded swimlanes, cubic Bezier edges, and hover-activated animated connections. Edges are greyed out by default; hover a component to highlight its connections and dim unrelated nodes.
- **Component cards** -- containment-based card layout grouped by category (frontend, API, database, auth, etc.) with connection tags showing how components relate
- **Risk analysis** (optional) -- flags like "sessions stored in primary DB" or "no caching on read-heavy endpoints", calibrated to your project's scale
- **Plain-English explanations** -- every component and risk explained with real-world analogies, not jargon
- **5 themes** -- Warm (default), Cool, B&W, Grey, and Editorial

## Install

### From GitHub

Add the plugin marketplace, then install:

```
/plugin marketplace add lucycrx/architecture-review
/plugin install architecture-review@lucycrx-architecture-review
```

### Local development

Clone the repo and point Claude Code at it:

```bash
git clone https://github.com/lucycrx/architecture-review.git
claude --plugin-dir ./architecture-review
```

After installing, reload plugins to activate:

```
/reload-plugins
```

## Usage

Open any project with Claude Code, then run:

```
/architecture-review
```

Claude will ask you a few questions:

1. **Your goal** -- spot-checking your own code, understanding someone else's, learning, or preparing a handoff
2. **Diagram style** -- flow diagram (SVG), component cards, or both
3. **Risk analysis** -- yes or no

Then it scans the codebase and opens an `architecture-review.html` file in your browser.

## What it detects

**Component categories:** Frontend, API layer, Database, Cache, Auth, External services, Background jobs, Storage

**Risk patterns:** Session storage, missing caching, blobs in database, untrusted code execution, single-file state stores, and more -- each with severity ratings, scale-aware calibration, and suggested fixes

## Supported frameworks

Next.js, Express, Django, Rails, FastAPI, Laravel, Spring Boot, and most common web frameworks. Detection is file-based, so it works with any project that follows standard conventions.

## License

MIT
