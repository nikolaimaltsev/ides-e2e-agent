# E2E Test — Pass 4: Finalize & Review

Reusable instructions for the final pass of an E2E test implementation in
this repo. This pass first cleans up the code after Pass 2/3 findings, then
reviews it against the principles and sibling patterns, and ends with a
ready-to-merge decision.

## Purpose

- Finalize code after Pass 2 and Pass 3 findings.
- Remove unnecessary draft code and stale TODOs; keep only the minimal set
  of stable selectors.
- Review the result against `.agents/e2e-test-principles.md` and the closest
  sibling tests / page objects.
- Decide and report whether it's ready to merge.

## Rules

- No broad refactoring.
- No backend changes.
- No destructive scenarios in the test itself.
- All touched files must report `errors: []` via
  `mcp__jetbrains__get_file_problems`.

## Cleanup checklist

- Remove any `// TODO Pass 3:` markers added in Pass 2 that have been
  resolved (most should be).
- Remove obsolete commented-out code left from the draft phase.
- Remove draft/restating comments; keep comments that explain non-obvious why.
- Reconcile naming and style with sibling tests / page objects (test name
  casing, `@Step` description format, indentation, import order).
- Drop redundant `data-test` attributes if Pass 3 showed an existing anchor
  was enough (see principles, esp. "reuse built-in component affordances"
  and "use existing helpers").
- Confirm FE additions are minimal — every new `data-test` should be
  justified by an entry in the Pass 2 / Pass 3 decision table.
- Apply patterns symmetrically across mirrored components (e.g., two similar
  dialogs should share the same parent class and the same close approach).

## Review checklist

Assess the finalized code against `.agents/e2e-test-principles.md` and sibling
conventions — including but not limited to:

- Test classes follow sibling-file conventions (DSL chain shape, annotation
  style, naming).
- Dialog POs inherit `Dialog`; table POs inherit `Table` /
  `TableWithPagination`.
- API responses and request bodies use typed Jackson data classes, not
  hand-rolled JSON or per-field `JsonPath`.
- Selectors use `elementByDataTest` / `ByDataTestValues` /
  `byDataTestContains`, never raw `[data-test*='...']` strings.
- Built-in component affordances reused over custom hooks; test data scoped
  to where it's used; constants over magic literals.

## Required output

Save a single report at `.agents/automation/<task-id>-pass-4-scenario-<n>.md`
with:

### 1. Cleanup done in this pass

Bulleted list of edits — often "none" if Pass 2 + 3 were clean.

### 2. Design approach & reused patterns

Short writeup of the chosen design and the sibling patterns / base classes /
helpers reused.

### 3. Files changed / created

Code files only, one line each with a brief description. Separate "modified"
from "new".

### 4. Principles & consistency review

Pass / note per relevant principle and sibling-convention check (from the
review checklist above).

### 5. Verification results

| Source                                                   | Outcome      |
| -------------------------------------------------------- | ------------ |
| Pass 3 DOM verification                                  | ...          |
| Manual Gradle run (if performed)                         | ...          |
| `mcp__jetbrains__get_file_problems` on all touched files | `errors: []` |

### 6. Remaining risks

Anything not fixed in this PR but worth flagging — pre-existing FE warnings,
asymmetries with other tests, on-prem / cloud constraints, etc.

## Status

End the report with a clear "ready to merge" / "blocked on X" status so the
user can decide whether to push.