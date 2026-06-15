# E2E Test — Pass 2: Codebase Review + Draft Patch

Reusable instructions for Pass 2 of an E2E test implementation in this repo.

## Read first

This pass is only for **codebase review and a draft implementation**.

## Objective

Prepare a draft implementation of the scenario(s) the user described in the
prompt. The test environment is expected to be already set up (see
overview).

## Scope rules

Do:

- Review existing FE code for the route / feature under test.
- Review existing page objects in the same package and related areas.
- Review existing E2E style around the feature — start by reading the
  closest sibling test class.
- Reuse existing methods, base classes, and helpers as much as possible
  (see principles).
- Draft page-object / test changes in the working tree.
- Add minimal FE `data-test` attributes only if needed for robust
  selectors (see decision rules below).

Do not:

- Use browser / Playwright MCP.
- Add backend changes.
- Change setup / seed data (activation state, feature flags, user
  preloads).
- Add `@TestGroup`, `@OnPremOnly`, `@CloudOnly`, or
  `@OnTeamCityOnlyWithPreload` annotations unless explicitly requested or
  unless sibling tests in the same package consistently require them for
  determinism.
- Broadly refactor existing page objects or FE components.
- Claim the draft is final / verified.

## Codebase review checklist

Before drafting, identify existing patterns, incuding but not limited to:

- Opening the route under test — `asAdmin { openUrl(...); configurationPage().fooPage { … } }` chain or equivalent.
- Page-object base classes that fit the new component (`Dialog`,
  `Table`, `TableWithPagination`, `BaseConfigurationPage`, `TBEPage`).
- Alert / toast success assertions (`alert { checkIsSuccessful() }`,
  `alert("text") { ... }`).
- Dialog open / close patterns (`checkDialogVisible`, `closeDialog`,
  `checkDialogClosed`).
- Dropdown / menu patterns (Ring UI dropdown anchors, `ListDataItem`).
- Table patterns (`checkIfTableHasElementsCount`, `dataRows`,
  pagination).
- Selector helpers (`elementByDataTest`, `byDataTestContains`,
  `ByDataTestValues`) — never hand-roll `[data-test*='...']` strings.
- API client patterns — typed Jackson data classes under
  `http/tbe/models/<area>/`, shared `mapper: ObjectMapper`.
- Naming conventions for page-object methods and FE `data-test` values
  (kebab-case, semantic, scoped enough to be unique).
- DSL chain shapes

## Selector / `data-test` decision rules

Prefer existing page-object methods and stable selectors first.

**Add a new FE `data-test` only when all of these are true:**

1. The element is central to the tested flow.
2. No existing stable, specific selector is available.
3. The alternative would require brittle selectors — CSS module classes,
   Ring UI internals, DOM nesting, indexes, or generic text.

**Avoid:**

- CSS module class selectors.
- Ring UI internal DOM selectors (unless they're the standard like
  `ring-dialog-close-button` — see principles, "reuse built-in close
  mechanisms").
- Positional selectors ("first button"), unless safely scoped and
  unavoidable.
- Generic selectors such as `button[type=button]`.
- Adding `data-test` to every child element when one anchor on the
  parent is enough.
- Duplicate `data-test` attributes when a nearby stable anchor already
  works.

Visible product text may be asserted when the text itself is expected
behavior (e.g., a success-toast copy that is part of the spec).

## Required Pass 2 output


### 1. Findings

Briefly list relevant existing files / classes / methods discovered.

### 2. Draft changes

List files changed or proposed to change — separate FE, page objects,
tests, and API clients.

### 3. Selector / `data-test` decision table

| Element | Existing selector? | Decision | Reason |
|---|:---:|---|---|
| ... | yes / no | reuse / add `data-test` / defer | ... |

### 4. Assumptions needing browser verification

Things to check in Pass 3 — exact labels, dialog behavior, alert
timing, table ordering, `disabled` vs `aria-disabled`, async-rendering
races, redirect behavior, Ring UI token composition on `data-test`, etc.

### 5. Open questions / risks

Only unresolved items that affect implementation robustness.

## Important

This pass produces a useful draft and analysis, not a final verified
solution. Any uncertain selector, label, wait, or UI behavior must be
explicitly marked (TODO comment in the code or a row in the report) so
Pass 3 can verify it.
