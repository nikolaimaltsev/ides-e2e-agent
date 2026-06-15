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

| Helper | Scope | Match |
|---|---|---|
| `elementByDataTest("x")` | global | `[data-test*='x']` |
| `elementsByDataTest("x")` | global, collection | `[data-test*='x']` |
| `byDataTestContains("x")` | string for `find(...)` | `[data-test*='x']` |
| `byDataTestStrict("x")` | string for `find(...)` | `[data-test='x']` |
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

| Base class | Use it for |
|---|---|
| `Dialog` (`ui/tbe/blocks/Dialog.kt`) | Ring UI modal dialogs. Provides `checkDialogVisible`, `closeDialog`, `checkDialogClosed`, and a default locator scoped to `ring-dialog-container`. |
| `Table` / `TableWithPagination` (`ui/tbe/blocks/`) | Data tables. Provides `dataRows()`, `checkIfTableHasElementsCount(Int)`, pagination helpers. |
| `CompositeSelenideElement` | Generic block when no specialized base fits. |
| `TBEPage`, `BaseConfigurationPage` | Full pages — wait-for-progress, header / tabs, etc. |

**Why:** less duplicated code; consistent Allure step names across tests; the
test gets shared improvements (e.g. waits) for free.

---

## 3. Reuse built-in close mechanisms over custom Cancel buttons

If a UI library component already provides a standard close affordance —
e.g., Ring UI's `ring-dialog-close-button` (the X in dialog headers) — use it
via the parent class's `closeDialog()` rather than tagging a custom Cancel
button with a project-specific `data-test`.

```tsx
// ❌ Don't add a custom data-test to a Cancel button when the X already exists
<Button onClick={onCloseAttempt} data-test="cancel-button">Cancel</Button>
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

## 4. Type API responses with Kotlin data classes, not JsonPath

In test API clients under `http/tbe/`, deserialize responses into typed
Kotlin data classes via Jackson. Don't write one `getX(): String` accessor
per field that re-parses the JSON each call.

```kotlin
// ❌ Don't — one HTTP call per accessor, magic-string field paths
class LicenseClient(private val client: HttpClient, private val logger: Logger) {
  fun getLicenseId(): String          = JsonPath.read(getLicense(), "$.id")
  fun getActivatedByUsername(): String = JsonPath.read(getLicense(), "$.activatedBy.name")
  fun getLicenseStatus(): String      = JsonPath.read(getLicense(), "$.status")
  private fun getLicense(): String    = client.getWithSuccessCheck("/api/license")
}
```

```kotlin
// ✅ Do — one HTTP call, typed model, callers pick the fields they need

// http/tbe/models/license/License.kt
@JsonIgnoreProperties(ignoreUnknown = true)
data class License(val id: String, val status: String, val activatedBy: ActivatedBy)

@JsonIgnoreProperties(ignoreUnknown = true)
data class ActivatedBy(val name: String)

// http/tbe/LicenseClient.kt
class LicenseClient(
  private val client: HttpClient,
  private val logger: Logger,
  private val mapper: ObjectMapper,
) {
  fun getLicense(): License {
    val response = client.getWithSuccessCheck("/api/license")
    return mapper.readValue(response, License::class.java)
  }
}

// At the call site
val license = TBEApiClient.license.getLicense()
checkLicenseId(license.id)
checkAuthor(license.activatedBy.name)
```

Conventions:
- Place data classes under `http/tbe/models/<area>/`.
- Always annotate with `@JsonIgnoreProperties(ignoreUnknown = true)` so the
  model tolerates unknown response fields.
- Take the shared `mapper: ObjectMapper` via constructor and wire it from
  `TBEApiClient.kt` (`KotlinModule` is already registered).

**Why:**
- Compile-time field references instead of magic strings.
- One round trip instead of N — multiple fields read from one response.
- Easy to extend: add a field to the data class, callers use it.
- Matches the established pattern in `PluginsClient`, `ToolsClient`,
  `LVClient`, `AutomationTokensClient`, etc.

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

## Quick checklist before opening a PR

- [ ] No raw `[data-test*='...']` strings — used `elementByDataTest` / `ByDataTestValues` / `byDataTestContains` instead.
- [ ] Dialog POs extend `Dialog`; table POs extend `Table` / `TableWithPagination`.
- [ ] Custom close / Cancel mechanisms removed in favor of the inherited `closeDialog()` where the X is sufficient.
- [ ] API client responses deserialized into typed data classes under `http/tbe/models/...`, not parsed via `JsonPath`.
- [ ] Symmetric components implemented symmetrically (same parent, same conventions).
- [ ] Test data declared at the narrowest scope (local `val` for single use), matching sibling style.