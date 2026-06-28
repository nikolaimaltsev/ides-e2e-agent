# E2E Test — Pass 1: High-level assessment and test design options

## Purpose

- scenarios assessment against the existing code base
- assess for relevance and feasibility
- high-level implementation plan
- when drafting a plan, mind the existing code patterns and helpers

# Input

- Youtrack task url, with several test scenarios
- Take into work one scenario from the task at a time (4-pass iteration)
- First non-implemented scenario from the task

## Required Pass 1 output

- Assess scenario as doable/non-doable and if it makes sense or not or should be adjusted/omitted
- Validate each step against real UI behavior/gating; if a step is unreachable as written, flag it and propose realigning the scenario (and the task text) to match real behavior
- High-level design and implementation options
- Spots in the code for the changes: helpers and test file: will you create new or fit into existing
- Raised objections and concerns, well-argued suggestions to alter or omit some parts of the original task (if any).
- No code or test changes in this pass — the report is the only file written.
- Save the report at `.agents/automation/<task-id>-pass-1-scenario-<n>.md`
