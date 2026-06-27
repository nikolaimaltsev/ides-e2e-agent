# E2E Test Principles — IDE Services Server

Generic, reusable rules for writing E2E tests and supporting page objects /
API clients in this repo. Derived from PR review feedback on IDES-11445
(PR #504, reviewer: Olga). Apply these on every new test and page object.

---

## 1. Use existing `data-test` helpers, not raw CSS strings

**Don't** hand-roll CSS attribute selectors when looking up an element by its
`data-test` value.

```kotlin
// ❌ Don't
private val keyValue    = find("[data-test*='code-wrapper']")
private val cancelButton = find("button[data-test*='cancel-button']")
```

**Do** use the helpers from `selenide/SelenideUtils.kt`:

```kotlin
// ✅ Do
private val keyValue     = elementByDataTest("code-wrapper")
private val cancelButton = elementByDataTest("cancel-button")
```

Available helpers:

| Helper                       | Scope                          | Match                              |
| ---------------------------- | ------------------------------ | ---------------------------------- |
| `elementByDataTest("x")`     | global                         | `[data-test*='x']`                 |
| `elementsByDataTest("x")`    | global, collection             | `[data-test*='x']`                 |
| `byDataTestContains("x")`    | string for `find(...)`         | `[data-test*='x']`                 |
| `byDataTestStrict("x")`      | string for `find(...)`         | `[data-test='x']`                  |
| `ByDataTestValues("x", "y")` | `By` locator class, composable | `[data-test~='x'][data-test~='y']` |

**Why:** a single source of truth for selector format, smaller diff per page
object, easier to refactor selector conventions later.

---

## 2. Inherit project-specific base classes for known UI patterns

When the page object models a dialog, table, page, or any block that already
has a project base class, **extend that base class**. Don't re-implement
common behavior on top of `CompositeSelenideElement` from scratch.

```kotlin
// ❌ Don't
class LicensePublicKeyDialog(
  selector: String = "div[data-test*='license-public-key-dialog']",
) : CompositeSelenideElement(element(selector)) {
  fun checkIsOpen()       { rootElement.shouldBe(Condition.visible) }
  fun checkDialogClosed() { rootElement.shouldNot(Condition.exist) }
  // …dialog-specific methods…
}
```

```kotlin
// ✅ Do
class LicensePublicKeyDialog : Dialog(ByDataTestValues("license-public-key-dialog")) {
  // Only dialog-specific methods here.
  // checkDialogVisible() and checkDialogClosed() come from Dialog.
}
```

Known base classes worth looking at before writing a new PO:

| Base class                                         | Use it for                                                                                                                                         |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Dialog` (`ui/tbe/blocks/Dialog.kt`)               | Ring UI modal dialogs. Provides `checkDialogVisible`, `closeDialog`, `checkDialogClosed`, and a default locator scoped to `ring-dialog-container`. |
| `Table` / `TableWithPagination` (`ui/tbe/blocks/`) | Data tables. Provides `dataRows()`, `checkIfTableHasElementsCount(Int)`, pagination helpers.                                                       |
| `CompositeSelenideElement`                         | Generic block when no specialized base fits.                                                                                                       |
| `TBEPage`, `BaseConfigurationPage`                 | Full pages — wait-for-progress, header / tabs, etc.                                                                                                |

**Why:** less duplicated code; consistent Allure step names across tests; the
test gets shared improvements (e.g. waits) for free.

---

## 3. Reuse built-in component affordances instead of adding custom hooks

When a UI library already exposes a standard affordance for an action
(closing, submitting, clearing), reach it through the existing base-class
method or standard selector rather than tagging a project-specific `data-test`
or writing a custom PO method — **as long as that affordance is reliably
selectable** (if it isn't, a minimal `data-test` is still fine, see #1). The
most common case is a dialog's close X vs. a custom Cancel button:

```tsx
// ❌ Don't add a custom data-test to a Cancel button when the X already exists
<Button onClick={onCloseAttempt} data-test="cancel-button">
  Cancel
</Button>
```

```kotlin
// ❌ And don't add a custom clickCancel() method to the PO
fun clickCancel()       { cancelButton.click() }
fun checkDialogClosed() { rootElement.shouldNot(Condition.exist) }
```

```kotlin
// ✅ Do use the inherited closeDialog() — it clicks the standard X and
//    asserts the dialog is closed in one call.
publicKeyDialog {
  checkDialogVisible()
  checkKeyIsNotEmpty()
  closeDialog()
}
```

**Why:** Cancel and X usually wire to the same handler (`onCloseAttempt`) so
they exercise the same code path. Adding a custom `data-test` creates a
redundant local convention when a universal one already works. Bonus: the
FE diff shrinks (no FE change needed for the dialog at all).

---

## 4. Use typed data classes for API responses and requests

In test API clients under `http/tbe/`, model both directions with Jackson data
classes via the shared `ObjectMapper` — `mapper.readValue(...)` for responses,
`mapper.writeValueAsString(...)` for request bodies. Don't hand-roll JSON
strings for requests, and don't re-parse responses with per-field `JsonPath`
accessors.

```kotlin
// http/tbe/models/<area>/Thing.kt
@JsonIgnoreProperties(ignoreUnknown = true)        // tolerate unknown response fields
@JsonInclude(JsonInclude.Include.NON_NULL)         // omit nulls for partial PATCH bodies
data class Thing(val id: String, val name: String? = null)

fun getThing(): Thing = mapper.readValue(client.getWithSuccessCheck("/api/thing"), Thing::class.java)
fun patchThing(data: Thing) = client.patchWithSuccessCheck("/api/thing", mapper.writeValueAsString(data).toJsonBody())
```

Conventions:

- Place data classes under `http/tbe/models/<area>/`.
- Annotate with `@JsonIgnoreProperties(ignoreUnknown = true)`; add
  `@JsonInclude(NON_NULL)` when the same model serves partial request bodies.
  Note: `NON_NULL` can't emit an explicit `null`, making "absent" and "null"
  indistinguishable — for endpoints where `null` means "clear the field," keep a
  dedicated method (raw body) for that case.
- Take the shared `mapper: ObjectMapper` via constructor, wired from
  `TBEApiClient.kt`.

**Why:** compile-time field references instead of magic strings, one round trip
instead of N, trivially extensible, and consistent with `PluginsClient`,
`ToolsClient`, `LVClient`, etc.

---

## 5. Apply the same pattern symmetrically across mirrored components

When two page objects model two symmetric UI elements (e.g., a "view X"
dialog and a "regenerate X" dialog), keep their structure mirrored. If one
inherits `Dialog`, both should. If one drops a custom `cancel-button`
`data-test` in favor of the standard close, both should.

**Why:** asymmetry is a maintenance tax. Future readers shouldn't have to
remember "this dialog uses pattern A, that one uses pattern B." A reviewer
who fixes one will expect the other to follow.

---

## 6. Scope test data to where it's used; follow sibling declarations

Declare test constants and fixtures at the narrowest scope that works — a local
`val` inside the test for single-use data, a class property when it's shared
across tests in the file. Don't reach for a companion `const val` just to name a
value; it implies a shared constant when it usually isn't. Match how the closest
sibling test already declares similar data.

**Why:** scope communicates intent, and matching neighbouring style keeps a file
readable without per-test exceptions.

---

## 7. Prefer named constants over magic literals

Reference an existing constant instead of repeating a literal. For standard
protocol/library values (HTTP header names, media types, status codes) use the
constants the framework already provides (e.g. ktor's `HttpHeaders.Accept`,
`ContentType.Application.Json`) rather than `"Accept"` / `"application/json"`.
For a value repeated within a file (a URL path, a fixed key) extract one local
`const val`. Single-use domain data still stays local — this is about de-duplicating and
naming well-known values, not promoting every literal to a constant.

**Why:** one source of truth, no typos, and the intent of the value is visible
at a glance.

---

## 8. Prefer the lowest-blast-radius change

When a change can be done several ways, pick the one that touches the fewest
shared / other-owned files. Favor backward-compatible edits (e.g. add a default
parameter, keep the existing name/signature) so existing callers stay untouched
and only your own code adapts.

**Why:** fewer conflicts, less risk to others' code, smaller review.
**Boundary:** a tie-breaker among equally-correct options — never over
correctness or the reviewer's intent.

---

## Quick checklist before opening a PR

- [ ] No raw `[data-test*='...']` strings — used `elementByDataTest` / `ByDataTestValues` / `byDataTestContains` instead.
- [ ] Dialog POs extend `Dialog`; table POs extend `Table` / `TableWithPagination`.
- [ ] Built-in component affordances reused instead of custom hooks where reliably selectable (e.g. dialog X via `closeDialog()` over a custom Cancel).
- [ ] API client responses deserialized into typed data classes under `http/tbe/models/...`, not parsed via `JsonPath`.
- [ ] Symmetric components implemented symmetrically (same parent, same conventions).
- [ ] Test data declared at the narrowest scope (local `val` for single use), matching sibling style.
- [ ] No magic literals for standard/repeated values — used framework constants (headers, media types) or a single `const val`.
