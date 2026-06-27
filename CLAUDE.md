# E2E Test Overview — IDE Services Server

## Goal

Implement E2E test(s) for the scenario(s) described by the user in a
separate prompt. Tests live in:

- **Test classes:** `tbe-ui-tests/src/test/kotlin/com/jetbrains/tbe/uitests/e2e/...`
- **Page objects:** `tbe-ui-tests/src/main/kotlin/com/jetbrains/tbe/uitests/ui/tbe/pages/...`
- **API clients + typed models:** `tbe-ui-tests/src/main/kotlin/com/jetbrains/tbe/uitests/http/tbe/...`
- **Frontend (only when adding stable `data-test` attributes):** `tbe-ui/workspaces/ides/src/...`

## Scope

Implement only the scenario(s) described in the prompt. Don't bundle
adjacent fixes, refactors, or polish unless the user asks for them.

## General rules

- one task consists of several scenarios
- **Important:** tests within one task must be grouped in a single file, unless there are existing files for some scenarios,
  or special conditions/concerns
- **Do not add backend changes.**
- **Make sure not to break other people's code with your changes**
- **After every code change, verify inline errors via JetBrains IDE MCP.**
  Call `mcp__jetbrains__get_file_problems` on each edited file before moving on.
- **Important:** before considering implementation check and infer style and convention reference
  **`.agents/e2e-test-principles.md`**
- use Jetrains IDEA MCP tools if feels faster or more convenient or efficient than cli tools.
  You don't have to but feel free to use it if needed.

## Implementation approach

- Use a **4-pass workflow**. Each pass has a clear purpose, set of rules, and
  concrete deliverable.

### Pass 1 — High-level assessment + test design options

- Read and infer instruction for the pass: **[`.agents/e2e-test-pass-1.md`](e2e-test-pass-1.md)**
- Confirm deliverables are complete and ask for confirmation before moving to the next pass.

### Pass 2 — Codebase review + draft patch

- Read and infer instruction for the pass: **[`.agents/e2e-test-pass-2.md`](e2e-test-pass-2.md)**
- Confirm deliverables are complete and ask for confirmation before moving to the next pass.

### Pass 3 — Browser / MCP verification

- Read and infer instruction for the pass: **[`.agents/e2e-test-pass-3.md`](e2e-test-pass-3.md)**
- Confirm deliverables are complete and ask for confirmation before moving to the next pass.

### Pass 4 — Finalize & Review

- Read and infer instruction for the pass: **[`.agents/e2e-test-pass-4.md`](e2e-test-pass-4.md)**
- Confirm deliverables and report results.
