# AtlasTint interface rules

The rules for AtlasTint's components, styling, visual direction and states, moved from `AGENTS.md`. Read it before editing a component, a style or a token.

## Contents

- [Component and styling conventions](#component-and-styling-conventions)
- [Visual direction](#visual-direction)
- [Color system](#color-system)
- [Desktop support and responsive behavior](#desktop-support-and-responsive-behavior)
- [Accessibility](#accessibility)
- [Performance](#performance)
- [Error, empty, and loading states](#error-empty-and-loading-states)

## Component and styling conventions

- Prefer semantic HTML and native browser behavior.
- Use shadcn/Base UI primitives for complex accessible behavior instead of rebuilding dialogs, menus, selects, tooltips, or popovers.
- Use native elements when a primitive adds no value.
- Use the existing `cn()` helper for class composition.
- Use the project’s established variant utility for finite visual variants.
- Add `data-slot` to reusable component parts.
- Express states through native attributes and meaningful `data-*` attributes.
- Icon-only buttons require an accessible name.
- Every interactive element requires visible `:focus-visible` treatment.
- Do not use generic `div` elements as buttons, links, checkboxes, or listbox options.
- Text fills its column, with no width cap of its own; a cap goes only on a container (a card, a toast, the page column), never on an element with text classes. The page reserves the scrollbar space with `html { scrollbar-gutter: stable }`. `scripts/text_width_check.py` checks both in `pnpm lint` and `pnpm build` ([#104](https://github.com/martonpaulo/atlastint/issues/104)).

All reusable visual values must come from semantic design tokens where practical:

- color
- typography
- spacing
- size
- radius
- shadow
- border width
- motion duration
- easing
- map fill
- map boundary
- focus treatment
- selected treatment
- tooltip surface

Do not introduce one-off hardcoded hex values or arbitrary dimensions when an existing semantic token fits.

## Visual direction

The product is an editorial cartography workspace, not a generic SaaS dashboard.

Maintain one visual direction through delivery:

- map-first composition
- spacious but information-dense desktop layout
- warm neutral canvas
- crisp geographic outlines
- restrained saturation
- deterministic semantic palettes
- strong type hierarchy
- subtle depth
- one deliberate light appearance
- minimal ornament

Avoid:

- glassmorphism
- neon gradients
- random rainbow fills
- gratuitous animation
- excessive cards inside cards
- tiny low-contrast labels
- decorative charts unrelated to map progress
- large empty hero sections inside the application
- redesigning the visual language during unrelated tasks

The map should own most of the viewport. Controls must remain discoverable without competing with geography.

## Color system

Default mode: hierarchical deterministic palette.

- World base hue comes from continent or documented geographic group.
- Brazil base hue comes from official geographic region.
- Spain base hue comes from autonomous community.
- Child variation is deterministic and independent of selection order.
- Unselected regions use a neutral semantic fill.
- Hover, focus, selected, partial, and unavailable states must be distinguishable.
- Selection cannot rely on color alone.
- Custom colors must still preserve readable boundaries and focus indication.

Supported modes:

- hierarchical palette
- single accent with deterministic tonal variation
- selection order (persisted as `chronology`)
- per-entity custom color

Never generate random colors at render time.

## Desktop support and responsive behavior

The application is intentionally desktop-only for the initial product.

Minimum supported viewport:

- width: 1024 CSS pixels
- height: 700 CSS pixels

Below either threshold:

- do not render the main workspace
- render a polished, accessible explanation
- state that a larger browser window or desktop display is required
- reevaluate when the viewport changes
- do not use user-agent sniffing as the primary gate

Desktop-only does not mean fixed-pixel layout. Support:

- browser zoom
- different desktop aspect ratios
- split-screen desktop use when still above the minimum
- intrinsic sizing
- CSS Grid and Flexbox
- logical properties
- overflow without clipped controls

Do not create a hidden or broken mobile version.

## Accessibility

The searchable entity list is the primary keyboard and screen-reader interaction surface.

- Do not place hundreds of SVG paths into the tab order.
- The SVG must expose a useful title, description, and current preset context.
- All map actions must have an equivalent accessible list action.
- Announce meaningful selection changes through a restrained live region.
- Progress must expose current value, minimum, maximum, and readable text.
- Parent partial selection must expose its mixed state.
- Tooltips cannot contain information unavailable elsewhere.
- Preserve keyboard navigation, visible focus, zoom support, reduced motion, and sufficient contrast.
- Do not rely only on hover.
- Respect `prefers-reduced-motion`.
- Test at 200% browser zoom within the supported desktop viewport when practical.

## Performance

Optimize for real interaction latency, not theoretical micro-optimizations.

- Lazy-load each preset and its geometry.
- Do not load all maps at startup.
- Do not put large GeoJSON or TopoJSON objects in global state.
- Avoid global rerenders during hover, pointer movement, or tooltip positioning.
- Precompute or memoize only clearly expensive stable calculations.
- Do not scatter defensive `useMemo` and `useCallback` everywhere.
- Avoid blocking the main thread with runtime geometry simplification or broad data transformation.
- Use code splitting for non-initial routes and preset bundles.
- Keep the initial route useful before optional presets finish loading.
- Inspect the production bundle and generated asset sizes before completion.

Review and justify:

- initial JavaScript exceeding 300 KB gzip
- any individual compressed map asset exceeding 1.5 MB
- long tasks exceeding 50 ms during normal selection or search
- interaction latency visibly above one animation frame for ordinary toggles

These are review thresholds, not reasons to destroy geographic quality blindly.

## Error, empty, and loading states

Every user-visible asynchronous or fallible operation needs an intentional state:

- preset loading
- preset load failure
- missing geometry
- unavailable storage
- malformed persisted state
- unsupported future schema
- invalid import
- partial import compatibility
- empty search results
- no regions selected
- unsupported viewport

Do not silently swallow errors. Provide a recovery action when one exists.
