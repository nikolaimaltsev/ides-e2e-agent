# E2E Test — Pass 3: Browser / MCP Verification

Reusable instructions for Pass 3 of an E2E test implementation in this repo.


## Purpose

- Verify actual UI behavior and rendered text against the running app
  using Playwright MCP.
- Validate dropdowns, dialogs, alerts, tables, waits, and disabled /
  enabled transitions.
- Confirm whether the Pass 2 selector strategy actually reaches the DOM
  elements we expect.

## Environment

- Use **Playwright MCP** attached via CDP to a user-logged-in Chrome
  session (Chrome started with `--remote-debugging-port=9222` and a
  dedicated `--user-data-dir`).
- Local IDE Services server expected at `http://localhost:9998`.
- Log on as `toolbox.admin` once in the attached Chrome — Playwright MCP
  reuses the session, so no credentials handling is needed.
- Reference docs: <https://www.jetbrains.com/help/ide-services/get-started.html>.
  Use web search to navigate them if needed beyond source code.

## Rules

- Firstly, remind the user to run the test manually and wait for his feedback
- Drive the live UI only — no test runner, no Gradle.
- Do not broaden scope: verify only what Pass 2 drafted.
- **Never perform destructive actions** (regenerate public key, delete
  user, deactivate license, drop tables, etc.). If the scenario tests a
  destructive flow, stop short of the final confirmation step and verify
  everything up to it.
- Record differences between code-only assumptions and real UI behavior
  — selector composition (e.g., Ring UI prepending its own tokens to
  forwarded `data-test`), label wording, async timing, `disabled` vs
  `aria-disabled`, redirect behavior, console / network noise.

## Verification mechanics

For each Pass 2 assumption / selector, capture one or more of:

- DOM evidence — actual `data-test` attribute value, tag name, parent
  scoping. Use `browser_evaluate` with a targeted
  `document.querySelector` / `querySelectorAll` script and return a
  small JSON of fields you care about.
- Screenshot (optional, for UI states that aren't easily described in
  text).
- Console / network state (`browser_console_messages`,
  `browser_network_requests`) — call out any new errors caused by the
  change, and explicitly note pre-existing noise as such.

## Required Pass 3 output


### 1. Method

One short paragraph: server URL, auth mechanism (e.g., CDP attach to a
logged-in Chrome), test license / data state at the time of
verification.

### 2. Per-item results

A PASS / FAIL line per Pass 2 verification item, each with a DOM
snippet or brief note showing what was observed.

### 3. Confirmed selector strategy

A table or list mapping each PO field / method to its final selector,
ready for Pass 4.

### 4. Updated implementation plan (only if needed)

If a Pass 2 assumption failed, document the corrected approach.
Otherwise state explicitly: "no Pass 2 changes required."

### 5. Code changes (only if needed)

Only patch code in Pass 3 if verification reveals a defect. List any
such changes here with a brief rationale.

## Important

Pass 3 is for verification, not implementation. If everything passes, the
deliverable is just the report — no code changes — and Pass 4 picks up
clean.
