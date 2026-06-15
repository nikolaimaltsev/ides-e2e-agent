# E2E Test — Pass 4: Final Cleanup and Hardening

Reusable instructions for Pass 4 of an E2E test implementation in this repo.


## Purpose

- Finalize code after Pass 2 and Pass 3 findings.
- Remove unnecessary draft code and stale TODOs.
- Keep only the minimal set of stable selectors.
- Ensure tests follow existing project style.
- double check against principles: .agents/e2e-test-principles.md

## Rules

- No broad refactoring.
- No backend changes.
- No destructive scenarios in the test itself.
- All touched files should report `errors: []` via
  `mcp__jetbrains__get_file_problems`.

## Cleanup checklist

- Remove any `// TODO Pass 3:` markers added in Pass 2 that have been
  resolved (most should be).
- Remove obsolete commented-out code left from the draft phase.
- Remove comments that you added if any - code must be self-explanatory
- Reconcile naming and style with sibling tests / page objects (test
  name casing, `@Step` description format, indentation, import order).
- Drop redundant `data-test` attributes if Pass 3 showed an existing
  anchor was enough (see principles, esp. "reuse built-in close
  mechanisms" and "use existing helpers").
- Confirm FE additions are minimal — every new `data-test` should be
  justified by an entry in the Pass 2 / Pass 3 decision table.
- Apply patterns symmetrically across mirrored components (e.g., two
  similar dialogs should share the same parent class and the same close
  approach).

## Required Pass 4 output


### 1. Cleanup done in this pass

Bulleted list of edits — often "none" if Pass 2 + 3 were clean.

### 2. Final files changed

Code files only. One line per file with a brief
description and diff stats. Separate "modified" from "new".

### 3. Style / consistency checks

Confirm:

- Test classes follow sibling-file conventions (DSL chain shape,
  annotation style, naming).
- Dialog POs inherit `Dialog`; table POs inherit `Table` /
  `TableWithPagination`.
- API responses are typed Jackson data classes, not parsed via
  `JsonPath`.
- Selectors use `elementByDataTest` / `ByDataTestValues` /
  `byDataTestContains`, never raw `[data-test*='...']` strings.

### 4. Verification results

| Source | Outcome |
|---|---|
| Pass 3 DOM verification | ... |
| Manual Gradle run (if performed) | ... |
| `mcp__jetbrains__get_file_problems` on all touched files | `errors: []` |

### 5. Remaining risks

Anything not fixed in this PR but worth flagging — pre-existing FE
warnings, asymmetries with other tests, on-prem / cloud constraints,
etc.

## Status

End the report with a clear "ready to merge" / "blocked on X" status so
the user can decide whether to push.
