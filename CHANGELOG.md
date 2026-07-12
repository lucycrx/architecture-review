# Changelog

All notable changes to this project will be documented in this file.

## [1.2.0] - 2026-07-12

### Changed
- Learn-more links now point to the companion site at `sysark.dev` (was `systemdesignschool.com`)
- Glossary links resolve to full concept pages (`/concepts/<id>`) instead of a fragment on `/glossary`
- Report footer attribution updated to Sysark
- Base URL is now a single `SITE_BASE` constant in the HTML template

### Fixed
- Risk patterns referenced Build Story stages that don't exist (`online-store` stage-4, `ride-sharing` stage-4/5); remapped to valid stages so learn-more links resolve

## [1.1.0] - 2026-04-17

### Added
- Interactive SVG flow diagram with left-to-right data flow visualization
- Hover-activated animated connections — edges are greyed out by default, colored and animated on hover
- Color-coded dashed swimlane groups in flow diagram
- Interactive wizard at skill invocation — asks about goal, diagram style, and risk analysis preference
- Three diagram modes: flow diagram, component cards, or both
- Optional risk analysis — can be skipped via the wizard
- Flow tier assignment step for logical data flow grouping

### Changed
- Risk analysis step is now conditional based on wizard choice
- Updated JSON schema with `options`, `flowTiers`, and per-component `flowTier` fields

## [1.0.1] - 2026-04-01

### Fixed
- Circular dependency crash in `assignLayer` — added cycle detection (visited-set + max-depth guard)
- Silent rendering failures — added try/catch with visible error banner

### Changed
- Added DAG requirement to connection guidelines in SKILL.md

## [1.0.0] - 2026-03-30

### Added
- Initial release
- Containment-based architecture diagram with component cards and connection tags
- Risk analysis with severity ratings and scale-aware calibration
- Plain-English explanations with real-world analogies
- 5 theme options (Warm, Cool, B&W, Grey, Editorial)
- Detail sidebar with component info on click
- Keyboard navigation and accessibility support
- 8 component categories with Lucide icons
- Risk patterns library with framework-specific hints
