# AtlasTint testing and validation

How AtlasTint is tested and validated, moved from `AGENTS.md`. Browser acceptance is Chromium only: automated and manual browser checks use Chromium for Testing, never Brave.

## Contents

- [Testing strategy](#testing-strategy)
- [Validation order](#validation-order)
- [Artifacts and temporary processes](#artifacts-and-temporary-processes)

## Testing strategy

Use the lowest-cost test that proves the behavior.

### Unit tests

Required for complex or durable pure logic, including:

- search normalization and ranking
- aliases and abbreviations
- percentage formatting
- deterministic color assignment
- hierarchical parent/child selection
- mixed parent state
- persistence serialization
- schema migrations
- malformed storage recovery
- import validation
- unknown entity handling
- manifest and geometry invariants
- projection preference persistence

### Component tests

Use accessible queries to verify:

- selecting from search
- selecting from the list
- progress updates
- parent mixed state
- destructive confirmation
- keyboard behavior
- unsupported viewport handling
- storage failure messaging

### End-to-end tests

Keep Playwright focused on critical journeys:

- select an entity on the map
- select an entity through search
- reload and retain progress
- switch presets without data loss
- change world projection without losing selection
- select and partially deselect a Spanish parent group
- export and import valid progress
- reject invalid import data
- display and recover from the unsupported viewport screen
- render light regardless of the operating-system color scheme

For bug fixes, add a focused regression test when practical and ensure it would fail for the original defect.

Avoid tests that mirror implementation details, duplicate constants, or merely test framework behavior.

## Validation order

Run the narrowest relevant validation first, then expand according to scope.

Before completion, the full project must pass:

1. formatting check
2. lint
3. TypeScript check
4. unit and component tests
5. geographic data invariant checks
6. Playwright critical journeys
7. production build
8. bundle and asset inspection

Inspect a failure before rerunning. Never rerun an unchanged failed command without a relevant code, configuration, environment, or test change.

## Artifacts and temporary processes

- Store screenshots, traces, logs, coverage output, generated reports, and temporary validation files only under `artifacts/`.
- Do not delete pre-existing artifacts or user files.
- Remove current-task temporary artifacts before closeout unless they are a requested deliverable or needed as failure evidence.
- Stop development servers, preview servers, test watchers, browser sessions, and other task-specific processes before finishing.
