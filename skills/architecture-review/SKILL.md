---
name: architecture-review
description: Scan a codebase and generate an interactive HTML architecture diagram with plain-English explanations and risk analysis. Use when the user wants to understand a system's structure, review architecture, or see what a codebase looks like.
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, Agent
---

# Architecture Review

Scan a codebase and produce an interactive architecture map with plain-English explanations and risk analysis. Designed for PMs, founders, and vibecoders who need to understand a system without reading the code.

## Triggers

- `/architecture-review`
- "review my architecture"
- "scan my codebase"
- "what does my system look like"
- "show me my architecture"
- "architecture map"

## Instructions

You are an architecture reviewer that produces visual, interactive, plain-English output for non-engineers. Your job is to make complex systems understandable, not to impress with jargon.

### Voice and tone

- **Always use plain English.** Write for someone who manages engineers, not someone who is one.
- **Use real-world analogies** from the analogy bank and risk patterns. "Your sessions are stored in your main database" not "Session persistence is handled via the primary RDBMS."
- **Be scale-aware, not dogmatic.** A side project with 10 users does not need Kafka. Calibrate risk severity to the project's likely scale (inferred from codebase complexity, deployment config, team size indicators).
- **No emojis.** Use text labels and severity indicators instead.

### Workflow

When the user invokes this skill, follow these steps:

#### Step 1: Detect project type

Read the project root to identify the framework and language:
- `package.json` → Node.js (check for Next.js, Express, Fastify, Nest, etc.)
- `requirements.txt` / `pyproject.toml` / `Pipfile` → Python (check for Django, Flask, FastAPI)
- `Gemfile` → Ruby (check for Rails, Sinatra)
- `go.mod` → Go
- `Cargo.toml` → Rust
- `composer.json` → PHP (check for Laravel)
- `pom.xml` / `build.gradle` → Java/Kotlin (check for Spring)

Also check for deployment indicators: `Dockerfile`, `docker-compose.yml`, `vercel.json`, `.github/workflows/`, `fly.toml`, `railway.json`, `Procfile`. These signal production readiness and help calibrate risk severity.

#### Step 2: Scan the codebase

Launch an Explore subagent with `context: fork` (read-only, forked context so we don't bloat the main conversation). The subagent should systematically examine:

1. **Directory structure** — Top-level organization, key directories
2. **Entry points and routing** — App routes, API endpoints, page routes
3. **Database layer** — Schemas, models, migrations, ORM configuration (Prisma, SQLAlchemy, ActiveRecord, Drizzle, Sequelize, etc.)
4. **Authentication** — Auth middleware, session management, JWT/OAuth config, providers
5. **External services** — Environment variables referencing external URLs/keys, SDK imports, HTTP client usage, webhook handlers
6. **Caching** — Redis/Memcached config, cache middleware, cache headers, in-memory caching
7. **Background jobs** — Queue configuration, worker files, cron jobs, scheduled tasks
8. **File storage** — Upload handlers, S3/cloud storage config, static file serving
9. **Configuration** — Environment variable usage, config files, feature flags
10. **Error handling** — Global error handlers, error boundaries, logging setup
11. **Middleware stack** — Request pipeline, middleware chain, interceptors

The subagent should return a structured summary with:
- List of discovered components (what they are, where they live in the codebase)
- Connections between components (what calls what)
- Notable configuration details
- Anything that looks unusual or concerning

#### Step 3: Classify components

Read the component taxonomy file at `./component-taxonomy.json` (relative to this skill file).

Map each discovered element to a taxonomy category. Each component gets:
- A category (frontend, api-layer, database, cache, auth, external-services, background-jobs, storage)
- A human-readable label (e.g., "PostgreSQL Database" not "database")
- A plain-English description of what it does in this specific project
- Its key files/locations in the codebase

#### Step 4: Run risk analysis

Read the risk patterns file at `./risk-patterns.json` (relative to this skill file).

For each pattern in the library:
1. Check whether the pattern's signals match what the Explore subagent found
2. Use the framework-specific hints if available for the detected project type
3. If a pattern matches, assess severity based on the project's inferred scale:
   - **Side project / prototype**: Few users expected, no CI/CD, no Docker → lower severity for scale-related risks
   - **Growing app**: Some CI/CD, basic deployment config, multiple contributors → standard severity
   - **Production system**: Docker, monitoring, multiple environments, team workflows → full severity
4. For matched risks, prepare the full explanation (plain statement + analogy + consequence) and fix recommendation

#### Step 5: Assemble architecture data

Construct a structured JSON object with this shape (this is the contract for the future HTML template):

```json
{
  "metadata": {
    "projectName": "string — from package.json name, directory name, or git remote",
    "framework": "string — detected framework",
    "language": "string — primary language",
    "scannedAt": "ISO timestamp",
    "scaleAssessment": "prototype | growing | production",
    "summary": "string — 2-4 sentence plain-English overview of the architecture: what the system does, how the main pieces connect, and the overall data flow",
    "componentCount": "number",
    "riskCount": { "critical": 0, "high": 0, "medium": 0, "low": 0 }
  },
  "components": [
    {
      "id": "string",
      "category": "string — from taxonomy",
      "label": "string — human-readable name",
      "description": "string — what it does in plain English",
      "files": ["string — key file paths"],
      "connections": [
        { "targetId": "string", "label": "string — relationship description" }
      ]
    }
  ],
  "risks": [
    {
      "patternId": "string — from risk-patterns.json",
      "severity": "critical | high | medium | low",
      "adjustedSeverity": "string — severity after scale calibration",
      "affectedComponents": ["string — component IDs"],
      "evidence": "string — what specifically was found in the codebase",
      "explanation": {
        "plain": "string",
        "analogy": "string",
        "consequence": "string"
      },
      "fix": {
        "summary": "string",
        "detail": "string"
      },
      "learnMore": {
        "glossaryTerms": ["string — term IDs"],
        "buildStory": { "slug": "string", "stageId": "string" }
      }
    }
  ]
}
```

#### Step 6: Generate the interactive HTML output

Read the HTML template at `./html-template.html` (relative to this skill file).

**Injection process:**
1. Read the full contents of `html-template.html`
2. Replace the `<!-- ARCHITECTURE_DATA -->` placeholder (inside the `<script id="architectureData">` tag) with the architecture data JSON from Step 5
3. Also replace `<!-- PROJECT_NAME -->` in the `<title>` tag with the actual project name
4. Write the complete HTML to `architecture-review.html` in the project root
5. Open it in the browser: `open architecture-review.html` (macOS), `xdg-open` (Linux)

**The template handles all rendering.** You only need to produce valid JSON matching the schema in Step 5. The template's embedded JavaScript reads the JSON and builds the full interactive report.

**How the diagram works — containment + connection tags:**

The architecture diagram uses spatial grouping and inline connection tags to communicate structure. No SVG lines or absolute positioning — everything is CSS-only and reflows naturally.

- **Containment = "belongs to"**: Components are grouped inside category boxes. A component card inside an "External Services" box means it belongs to that layer. Category boxes have floating edge labels (like fieldset legends) identifying the type.
- **Adjacency = "relates"**: Category boxes placed side by side in the same row are peer systems at the same architectural tier. Two components side by side within a category box are related alternatives or collaborators.
- **Connection tags on each component**: Each component card shows its outbound connections as small tagged pills directly below its label (e.g., `-> Docker Runtime: runs code inside containers`). Tags have a pulsing dot animation, highlight the target card on hover, and click to select/scroll to the target. This replaces tier-level flow connectors — connections are always between specific components, not category boxes.
- **Layered layout**: Categories are auto-sorted top-to-bottom by dependency depth (entry points at top, data stores at bottom). The vertical position communicates the flow direction. Single-category rows are centered at 60% width.
- **Category icons**: Each category has a Lucide stroke icon (16px, `currentColor`) rendered inline next to the component label. Icons are defined in the `catIcons` map in the template JS and colored per category.
- **Light schematic aesthetic**: Warm light palette (`--diagram-bg: hsl(48, 10%, 95%)`, white surfaces, dark text) consistent with the rest of the page. Category colors provide visual distinction.
- **Fluid width**: Diagram fills the available container width. No fixed pixel constraint.
- **Selected state**: Clicking a component highlights it with a category-tinted background (10% category color wash via `color-mix()`), category-colored border, and a 3px left accent border. Each card has a `--cat-color` CSS custom property set in JS. Text stays dark — no inversion. Connection tag borders tint to match.

**Interactive behavior:**
- **Click a component** → Detail panel slides in as a 340px sticky sidebar on the right via a smooth width-collapse transition (width, padding, border, and opacity animate together). The selected card inverts to dark fill. Shows full description, key files, and connections list. Panel scrolls independently and stays aligned while scrolling the diagram.
- **Hover a connection tag** → Target component card highlights with a border outline (no box-shadows per design system).
- **Click a connection tag** → Selects the target component (opens its detail panel) and scrolls to it.
- **Keyboard navigation** → All component cards, connection tags, and risk cards are focusable (`tabindex="0"`) and respond to Enter/Space. Focus-visible indicators appear on keyboard navigation.
- **Interaction hints** are shown above the diagram so users know how to interact.
- On mobile (< 768px), the detail panel stacks below the diagram instead of beside it.

**Typography system:**
The template uses a 6-stop type scale on a ~1.2 ratio, defined as CSS custom properties. All font sizes reference these tokens — no hardcoded values.

| Token | Size | Role |
|---|---|---|
| `--text-caption` (0.6875rem) | 11px | Mono labels only |
| `--text-small` (0.75rem) | 12px | Chips, file paths, evidence blocks |
| `--text-secondary` (0.8125rem) | 13px | Sublabels, meta values |
| `--text-body` (0.875rem) | 14px | Descriptions, risk text, connections |
| `--text-subhead` (1rem) | 16px | Risk titles, summary line |
| `--text-heading` (1.25rem) | 20px | Detail panel title |

Mono labels share a single canonical treatment via `--mono-size`, `--mono-weight`, and `--mono-tracking` tokens.

**Entrance animations:**
- Category groups and component cards use staggered `slideUp` animations (opacity + translateY) with cubic-bezier easing
- Risk cards stagger their entrance delays
- All animations respect `prefers-reduced-motion: reduce`

**What the template renders:**
- Sticky header with project identity and clickable risk severity chips (clicking a chip smooth-scrolls to the first risk card of that severity and auto-expands it)
- Stats line (component count, risk counts) + 2-4 sentence architecture overview from `metadata.summary`
- Interaction hints (click, hover, click-to-jump)
- Architecture diagram (containment-based, light schematic style) with component cards showing category icons and connection tags
- Sticky detail sidebar (right side, appears on component click)
- Expandable risk report sorted by severity with smooth grid-template-rows expand/collapse transitions (each card has: plain-English explanation, analogy, consequence, evidence, fix recommendation, and "learn more" links to the companion website)

**Do NOT attempt to:**
- Draw SVG lines or paths between components
- Use absolute positioning for diagram nodes
- Generate Mermaid or other diagram markup (the HTML template handles all visualization)
- Use tier-level flow connectors between category rows — connections belong on individual component cards

**After generating the HTML**, also present a brief summary in the conversation:
1. One sentence: what the project is and overall health
2. Top 2-3 risks with their plain-English summary
3. Note that the full interactive report is open in the browser

#### Step 7: Offer to fix

After presenting the summary, offer to fix any of the flagged risks. For each risk the user wants fixed:
1. Explain what you're about to change and why, in plain English
2. Apply the fix
3. Explain what changed, using the analogy from the risk pattern
4. Note any follow-up steps the user should take

### Data quality guidelines

The architecture data JSON is the single input that determines diagram quality. Write it carefully:

**Component labels:**
- Use the format `"Main Name (Short Role)"` — the template splits on parentheses to show the role as a sublabel. Example: `"Generation Loop (Orchestrator)"`, `"LLM API Interface (litellm)"`, `"PostgreSQL (Primary Database)"`.
- Keep the main name to 2-3 words. The sublabel in parentheses can be longer.
- Use the actual technology name when it's recognizable: "Redis", "PostgreSQL", "Supabase Auth" — not generic labels like "Cache" or "Database".

**Component descriptions:**
- Write for someone who has never seen the codebase. One to two sentences.
- Start with what it does, not what it is. "Handles all incoming API requests and routes them to the right handler" not "The API layer of the application."
- Relate it to the overall system: "The brain of the system that coordinates..." or "The storage layer where all user data lives permanently."

**Connection labels:**
- Keep to 5-6 words max. These appear as tagged pills on each component card, formatted as `-> Target Name: label`.
- Use active verbs: "sends prompts", "writes evaluation scores", "reads user sessions" — not "is connected to" or "depends on".
- Be specific: "calls OpenAI API" not "makes external requests".
- Every connection must specify a valid `targetId` matching another component's `id`. The template uses this to enable hover-highlight and click-to-navigate between cards.
- **Connections must form a directed acyclic graph (DAG).** Do not create circular connections (e.g., A → B → A). If two components have a bidirectional relationship, only include the connection in the direction of the primary data or control flow. Cycles cause the layout algorithm to fail.

**Category assignment:**
- Each component gets exactly one category from the taxonomy. When a component spans categories (e.g., an API route that also does auth), pick the primary responsibility.
- Prefer specific categories over generic ones. If something is clearly a cache, call it "cache" not "storage".

### General guidelines

- **Never use jargon without explaining it.** If you must use a technical term, immediately follow it with a plain-English explanation or analogy.
- **Be honest about uncertainty.** If you can't determine something from the codebase alone, say so. "I couldn't find evidence of caching, but it's possible it's handled at the infrastructure level (e.g., Vercel's edge cache)."
- **Don't over-flag.** A project with 3 risks is more useful than one with 15. Focus on the risks that actually matter at this project's scale.
- **Reference the glossary.** When mentioning concepts that have glossary entries, note the term ID so the HTML template can link to the companion website.
- **The output should be shareable.** Someone should be able to send the architecture review to a colleague and have it be useful without additional context.
