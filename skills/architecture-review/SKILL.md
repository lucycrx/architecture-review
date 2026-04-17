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

#### Step 1: Interactive wizard

Before scanning, ask the user a few quick questions to tailor the output:

```
Before I scan, a few quick questions:

1. What's your goal?
   a) Spot-check my own code (I built this)
   b) Understand someone else's code (new to this codebase)
   c) Learning / exploring (just curious how it works)
   d) Preparing for a handoff or review

2. Diagram style?
   a) Flow diagram — interactive SVG showing data flow between components with animated connections on hover (best for understanding how pieces connect)
   b) Component cards — grouped cards with connection tags (best for browsing individual components in detail)
   c) Both

3. Include risk analysis?
   a) Yes — flag potential architecture issues with severity ratings
   b) No — just show me the architecture map
```

How the wizard answers shape the output:
- **Goal** sets voice/depth: spot-check → concise, flag issues; understand others' code → more explanatory descriptions and analogies; learning → rich analogies and context; handoff → comprehensive with all details
- **Diagram style** determines which visualization(s) to generate (see Step 5.5 and Step 6)
- **Risk analysis** determines whether to run Step 4 and include the risk section

If the user skips the wizard or wants to get straight to it, use sensible defaults: goal=spot-check, diagram=both, risks=yes.

#### Step 2: Detect project type

Read the project root to identify the framework and language:
- `package.json` → Node.js (check for Next.js, Express, Fastify, Nest, etc.)
- `requirements.txt` / `pyproject.toml` / `Pipfile` → Python (check for Django, Flask, FastAPI)
- `Gemfile` → Ruby (check for Rails, Sinatra)
- `go.mod` → Go
- `Cargo.toml` → Rust
- `composer.json` → PHP (check for Laravel)
- `pom.xml` / `build.gradle` → Java/Kotlin (check for Spring)

Also check for deployment indicators: `Dockerfile`, `docker-compose.yml`, `vercel.json`, `.github/workflows/`, `fly.toml`, `railway.json`, `Procfile`. These signal production readiness and help calibrate risk severity.

#### Step 3: Scan the codebase

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

#### Step 4: Classify components

Read the component taxonomy file at `./component-taxonomy.json` (relative to this skill file).

Map each discovered element to a taxonomy category. Each component gets:
- A category (frontend, api-layer, database, cache, auth, external-services, background-jobs, storage)
- A human-readable label (e.g., "PostgreSQL Database" not "database")
- A plain-English description of what it does in this specific project
- Its key files/locations in the codebase

#### Step 5: Run risk analysis (if requested)

Skip this step if the user chose "No" for risk analysis in the wizard.

Read the risk patterns file at `./risk-patterns.json` (relative to this skill file).

For each pattern in the library:
1. Check whether the pattern's signals match what the Explore subagent found
2. Use the framework-specific hints if available for the detected project type
3. If a pattern matches, assess severity based on the project's inferred scale:
   - **Side project / prototype**: Few users expected, no CI/CD, no Docker → lower severity for scale-related risks
   - **Growing app**: Some CI/CD, basic deployment config, multiple contributors → standard severity
   - **Production system**: Docker, monitoring, multiple environments, team workflows → full severity
4. For matched risks, prepare the full explanation (plain statement + analogy + consequence) and fix recommendation

#### Step 6: Assemble architecture data

Construct a structured JSON object with this shape (this is the contract for the HTML template):

```json
{
  "metadata": {
    "projectName": "string — from package.json name, directory name, or git remote",
    "framework": "string — detected framework",
    "language": "string — primary language",
    "scannedAt": "ISO timestamp",
    "scaleAssessment": "prototype | growing | production",
    "summary": "string — 2-4 sentence plain-English overview of the architecture",
    "componentCount": "number",
    "riskCount": { "critical": 0, "high": 0, "medium": 0, "low": 0 }
  },
  "options": {
    "diagramStyle": "flow | cards | both",
    "includeRisks": true
  },
  "components": [
    {
      "id": "string",
      "category": "string — from taxonomy",
      "label": "string — human-readable name",
      "description": "string — what it does in plain English",
      "files": ["string — key file paths"],
      "flowTier": "string — flow tier ID (required when diagramStyle is 'flow' or 'both')",
      "connections": [
        { "targetId": "string", "label": "string — relationship description" }
      ]
    }
  ],
  "flowTiers": [
    {
      "id": "string — unique tier ID (e.g., 'orchestration', 'workers', 'state')",
      "label": "string — display name (e.g., 'Orchestration', 'AI Workers')",
      "category": "string — taxonomy category for color-coding (e.g., 'api-layer', 'database')"
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

The `options` object reflects the user's wizard choices. The `flowTiers` array and per-component `flowTier` fields are only required when `diagramStyle` is `"flow"` or `"both"`. The `risks` array can be empty when `includeRisks` is `false`.

#### Step 6.5: Assign flow tiers (if flow diagram selected)

Skip this step if the user chose "Component cards" only in the wizard.

Analyze the component connection graph and assign each component to a **flow tier** — a logical column in the left-to-right data flow. Flow tiers are NOT the same as taxonomy categories. They represent a component's role in the primary data/control flow.

**How to assign tiers:**
1. Identify 3-6 natural groupings based on how data flows through the system. Typical tier patterns:
   - Entry points / orchestration on the left
   - Processing / workers in the middle
   - External services and data stores on the right
2. Assign each component to exactly one tier based on where it sits in the primary flow
3. Order tiers left-to-right following the direction of data/control flow
4. Pick a representative `category` from the taxonomy for each tier — this determines the tier's color in the diagram (it uses the existing `--cat-*` color tokens)

**Guidelines:**
- Aim for 2-4 components per tier. If a tier has 5+, consider splitting it.
- If a component connects equally to multiple tiers, place it in the tier where it initiates action (source side).
- External services and data stores typically go in the rightmost tiers.
- The tier `label` should be a short, descriptive name like "Orchestration", "AI Workers", "External Services", "State & Memory" — not a taxonomy category name.

Add the `flowTiers` array and per-component `flowTier` fields to the architecture data JSON.

#### Step 7: Generate the interactive HTML output

Read the HTML template at `./html-template.html` (relative to this skill file).

**Injection process:**
1. Read the full contents of `html-template.html`
2. Replace the entire contents of the `<script id="architectureData" type="application/json">` tag (between the opening and closing script tags) with the architecture data JSON from Step 5. The template ships with sample data for preview — overwrite it completely.
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

**Theme support:**
The template includes a theme switcher in the header with 5 options. The user's selection is persisted in `localStorage`. All themes use the same CSS custom property surface — only `:root` values change.

| Theme | Description |
|---|---|
| Warm (default) | Warm off-white background, amber/earth category tones |
| Cool | Blue-grey base, cooler category hues (slate, teal, purple) |
| B&W | True black and white, high contrast, fully desaturated categories |
| Grey | Neutral greys with very low saturation (5%) category hints |
| Editorial | Pulled from the impeccable design system: `#FAFAF8` base, institutional palette (teal, steel blue, forest green, golden ochre) |

**What the template renders:**
- Sticky header with project identity and clickable risk severity chips (clicking a chip smooth-scrolls to the first risk card of that severity and auto-expands it)
- Stats line (component count, risk counts) + 2-4 sentence architecture overview from `metadata.summary`
- Interaction hints (click, hover, click-to-jump)
- Architecture diagram (containment-based, light schematic style) with component cards showing category icons and connection tags
- Sticky detail sidebar (right side, appears on component click)
- Expandable risk report sorted by severity with smooth grid-template-rows expand/collapse transitions (each card has: plain-English explanation, analogy, consequence, evidence, fix recommendation, and "learn more" links to the companion website)

**How the flow diagram works — interactive SVG:**

When `diagramStyle` is `"flow"` or `"both"`, the template auto-generates an interactive SVG flow diagram from the `flowTiers` data. You do NOT need to write any SVG — the template's JavaScript computes all positions, edge paths, and interactivity automatically from the JSON.

- **Left-to-right layout**: Each flow tier becomes a column. Components are vertically distributed within their column.
- **Color-coded swimlanes**: Each tier gets a dashed-border box colored to match its category. The tier label is centered above the box in the tier's accent color.
- **Cubic Bezier edges**: Connections render as smooth curved paths between node anchor points.
- **Hover-activated flow**: All edges are greyed out and static by default. Hovering a node activates its connected edges with color and animated dashed flow. Unconnected nodes dim. This keeps the diagram clean and readable even with many connections.
- **Edge labels on top**: Connection labels (from the component's `connections` array) are positioned along each edge path and always render above nodes.
- **Tooltips**: Hovering a node shows its description, files, and connections in a tooltip.

**Do NOT attempt to:**
- Write SVG markup directly — the template generates it from the JSON data
- Use absolute positioning for diagram nodes
- Generate Mermaid or other diagram markup

**After generating the HTML**, also present a brief summary in the conversation:
1. One sentence: what the project is and overall health
2. Top 2-3 risks with their plain-English summary (if risk analysis was included)
3. Note that the full interactive report is open in the browser

#### Step 8: Offer to fix (if risk analysis was included)

Skip this step if risk analysis was not included.

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
