# AtlasTint architecture and code rules

The rules for AtlasTint's code, geographic data and state, moved from `AGENTS.md`. Read it before editing `apps/web/src/`, `packages/` or `scripts/geo/`.

## Contents

- [Product mission](#product-mission)
- [Frozen technical direction](#frozen-technical-direction)
- [Dependency policy](#dependency-policy)
- [Architecture boundaries](#architecture-boundaries)
- [Geographic data contract](#geographic-data-contract)
- [Preset policy](#preset-policy)
- [Geographic preprocessing](#geographic-preprocessing)
- [SVG and D3 rules](#svg-and-d3-rules)
- [Search rules](#search-rules)
- [State and persistence](#state-and-persistence)
- [React and TypeScript conventions](#react-and-typescript-conventions)
- [Documentation](#documentation)

## Product mission

AtlasTint is a desktop-first, local-first web application for selecting, coloring, searching, and tracking geographic regions on high-quality interactive SVG maps.

Initial presets:

- World sovereign states
- Brazilian federative units
- Spanish provinces, autonomous communities, and autonomous cities

The product priority order is fixed:

1. Correct geographic identity and selection behavior
2. Reliable persistence and data integrity
3. Clear, fast, accessible interaction
4. Excellent visual hierarchy and cartographic legibility
5. Runtime performance
6. Implementation speed and maintainability
7. Additional features

Do not trade a higher-priority item for a lower-priority item.

## Frozen technical direction

Use the scaffolded versions unless a task explicitly requires an upgrade.

- React 19
- TypeScript in strict mode
- Vite
- TanStack Router with file-based routing
- Tailwind CSS v4
- shadcn/ui using Base UI primitives
- Zustand for genuinely shared client state
- Zod for persisted and imported data validation
- D3 Geo for projection and SVG path calculation
- `d3-geo-projection` for additional projections
- `d3-zoom` only for map zoom and pan behavior
- TopoJSON in static assets, converted with `topojson-client`
- Biome for formatting and linting
- Vitest and React Testing Library
- Playwright for critical end-to-end behavior
- pnpm

Do not add without an explicit, demonstrated need:

- backend or server runtime
- database or ORM
- authentication
- React Query
- Axios
- Redux
- Mapbox, Google Maps, Leaflet, OpenLayers, or tile services
- canvas or WebGL map rendering
- PWA or service worker support
- analytics or telemetry
- animation libraries
- CSS-in-JS
- Storybook
- a second component library
- a second state-management library
- a second validation library
- a second formatter or linter
- Turborepo or Nx

## Dependency policy

- Prefer browser APIs and small focused packages.
- Before adding a dependency, verify that the existing stack or platform cannot solve the problem cleanly.
- Explain why a new runtime dependency is required and which existing alternatives were rejected.
- Do not add a dependency for a one-function utility.
- Import only the D3 modules needed. Never depend on the umbrella `d3` package.
- Keep geographic preprocessing dependencies in development tooling, not in the browser bundle.
- Do not duplicate shadcn conventions with parallel class-merging or variant systems.
- Use the scaffolded `cn()` helper and the variant utility already used by shadcn components.
- Do not introduce `tailwind-variants` or a second `tailwind-merge` abstraction unless replacing the current convention is explicitly approved.

## Architecture boundaries

Keep these concerns independent:

- geographic source data
- optimized geometry assets
- entity manifests and stable identifiers
- projections and viewport fitting
- selection rules
- search and aliases
- color assignment
- durable progress state
- transient interaction state
- persistence and migrations
- import and export
- React presentation

Rules:

- Business rules must not live in route components or visual components.
- Components render prepared state and dispatch explicit actions.
- Keep reusable pure logic independent from React.
- Keep local interaction state local.
- Use Zustand only when ownership spans multiple independent UI surfaces or must persist.
- Derive values instead of storing synchronized duplicates.
- Do not store map geometry, projected paths, search results, percentages, or other derivable data in Zustand.
- Do not store geographic geometry in `localStorage`.
- Keep infrastructure errors behind typed, actionable domain or UI results.
- Model impossible states out with discriminated unions, exhaustive handling, and narrow public APIs.

## Geographic data contract

Every preset must define a curated manifest. The manifest, not the raw geometry source, determines what is selectable and what counts toward progress.

Each selectable entity needs:

- a stable application-owned ID
- a stable geometry mapping
- canonical display name
- local-language names where relevant
- useful aliases
- standard code or abbreviation where available
- parent group where applicable
- explicit selectability

Never use these as durable IDs:

- translated display names
- array positions
- SVG path order
- color values
- unstable upstream labels without normalization

A multi-polygon entity is one logical entity. Selecting any polygon selects the whole entity.

Validate at build time or in the preprocessing pipeline that:

- every manifest entity expected to render has geometry
- every selectable geometry maps to exactly one intended entity
- no duplicate stable IDs exist
- no unexpected selectable geometry enters the application
- all parent references resolve
- every alias normalizes predictably
- configured totals equal the manifest-derived totals

## Preset policy

### World

- Default projection: Equal Earth.
- Optional projections: Natural Earth, Robinson, and Mercator.
- Primary progress policy: 195 sovereign states, defined in the preset manifest as the 193 UN member states plus the Holy See and the State of Palestine.
- Do not derive the total from the number of Natural Earth features.
- Territories and disputed geometries may be visible when necessary for map continuity, but they must not silently change the primary total.
- Any selectable non-primary entity requires an explicit documented policy.
- Preserve important islands and multi-polygon states.
- Small states must remain discoverable through search, focus, and optional callouts. Do not distort their real geometry merely to create a larger click target.

### Brazil

- Primary selectable units: 26 states and the Federal District.
- Parent groups: the five official geographic regions.
- Search must support full names and two-letter abbreviations.
- Use official or official-derived IBGE territorial meshes.

### Spain

- Primary progress units: 50 provinces plus Ceuta and Melilla as autonomous cities.
- Parent groups: 17 autonomous communities, with autonomous cities represented according to the preset manifest.
- Selecting a parent selects all of its selectable children.
- Partial child selection produces an indeterminate parent state.
- Provincial borders are visually lighter than autonomous-community borders.
- Include mainland Spain, Balearic Islands, Canary Islands, Ceuta, and Melilla.
- Use clearly labeled insets where required for legibility.
- Support Spanish names and relevant co-official local names and aliases.
- Use official or official-derived IGN/CNIG or INE boundary data.

## Geographic preprocessing

Heavy geometry work must happen before runtime.

The reproducible preprocessing pipeline must:

1. obtain or read the documented source dataset
2. verify the source version and attribution
3. normalize coordinate reference systems when necessary
4. map source identifiers to stable application IDs
5. remove unused properties
6. repair or reject invalid geometry deliberately
7. simplify conservatively
8. preserve shared borders
9. convert optimized assets to TopoJSON
10. run manifest and geometry invariants
11. produce deterministic output
12. record source and transformation metadata

Do not:

- simplify geometry on every browser load
- commit unexplained third-party files
- ship enormous raw datasets to the browser
- silently edit political boundaries
- silently drop small islands or enclaves
- treat generated geographic assets as hand-edited source files

Generated assets must be reproducible. Their checksums should remain stable when inputs and preprocessing options are unchanged.

## SVG and D3 rules

- React owns the SVG DOM.
- D3 calculates projections, paths, bounds, centroids, and zoom transforms.
- Do not let D3 create and mutate the full component tree imperatively.
- Use an SVG `viewBox` and preserve vector rendering.
- Calculate projected geometry only when preset, geometry, projection, or viewport configuration changes.
- A selection toggle must not recompute unrelated path data.
- Isolate high-frequency pointer and tooltip state from the main application store.
- Avoid updating React state on every pointer-move event when a ref or CSS transform is sufficient.
- Keep zoom transforms bounded and deterministic.
- Fit-to-view must account for configured insets and safe padding.
- Use `vector-effect="non-scaling-stroke"` where appropriate to preserve boundary legibility during zoom.
- Dynamic SVG `d`, transform values, calculated dimensions, and dynamic CSS custom properties are legitimate inline values. Static presentation belongs in semantic styles and tokens.

## Search rules

Search must be local, immediate, deterministic, and dependency-free unless profiling proves otherwise.

Normalize shared search text by:

- Unicode normalization
- case folding
- diacritic removal
- whitespace normalization
- punctuation normalization where appropriate

Search across:

- canonical name
- localized names
- aliases
- ISO or administrative codes
- abbreviations
- parent group names

Ranking order should normally favor:

1. exact canonical or code match
2. exact alias match
3. prefix match
4. token-prefix match
5. substring match

Do not add fuzzy-search dependencies for the initial product.

Search interactions must support:

- keyboard navigation
- Enter to toggle or activate the focused result
- Escape to clear or close predictably
- map highlighting
- map focus without changing selection
- parent context in each result

## State and persistence

Persist only durable user intent:

- selected entity IDs
- selection metadata such as order and selected timestamp
- optional visit date
- fill mode
- custom colors
- active preset
- projection preference
- non-sensitive UI preferences that should survive reloads

Do not persist:

- geometry
- projected SVG paths
- hover state
- open popovers
- current tooltip
- temporary search text unless intentionally specified
- calculated percentages
- transient errors

Persistence requirements:

- one versioned schema
- Zod validation at the storage boundary
- explicit migrations
- safe fallback for malformed JSON
- safe behavior when browser storage is unavailable or quota-limited
- unknown entity IDs ignored or reported, never fatal
- writes coalesced when rapid changes would otherwise create unnecessary work
- cross-tab synchronization through the native `storage` event when useful
- no credentials or sensitive data in browser storage

Components must not call `localStorage` directly. Use a persistence adapter with narrow typed methods.

Import and export requirements:

- JSON only for the initial implementation
- independent export and persistence schema versions
- no user-visible application release version
- export timestamp
- validated preset progress
- preview before destructive replacement
- clear validation errors
- no raw map geometry in exports

## React and TypeScript conventions

- Use strict modern TypeScript.
- Never use `any`.
- Avoid unsafe assertions. Validate at boundaries and narrow types.
- Use `import type` for type-only imports.
- Use named exports. Do not use default exports except where a framework-generated convention strictly requires one.
- Use lowercase kebab-case file names.
- Do not create barrel files for internal folders.
- Prefer absolute imports through the configured alias.
- Do not use `React.FC`.
- React 19 code must not introduce unnecessary `forwardRef` wrappers.
- Extend native element props when building native-element wrappers.
- Place the props spread last unless a deliberate protected prop must override consumer input.
- Keep public component APIs small and composable.
- Prefer composition over boolean-prop matrices.
- Pure functions use explicit inputs and outputs and do not read global state.
- Keep sorting, filtering, formatting, mapping, normalization, migrations, and color assignment in shared pure functions.
- Comments explain non-obvious constraints or provenance, not what the syntax already says.

## Documentation

Document durable decisions and policies, not a brittle inventory of exact directories and filenames.

Documentation must explain:

- product behavior
- geographic inclusion policy
- source attribution and versions
- stable ID rules
- persistence schema and migration policy
- how to regenerate map assets
- how to add a preset
- architectural boundaries
- quality and validation commands

Do not document an exact code tree unless it is generated or automatically verified. File-layout documentation becomes stale quickly.
