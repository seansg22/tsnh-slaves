---
name: review-code
description: Review the current branch's code changes for correctness bugs, missing edge cases, performance issues, and code quality. Optionally cross-checks implementation against a PRD, BE TD, and/or FE TD to verify requirement traceability. Surfaces findings as a structured report. Use when you want a thorough code review of local changes before pushing, or to validate an MR before merging.
---

# Review Code

Review the **current branch's changes** (or a specific MR) for bugs, correctness issues, performance problems, and code quality concerns. When design documents are provided (PRD, BE TD, FE TD), also verify the implementation actually delivers what was designed — requirement traceability in both directions.

The review is surgical — focused on what changed, not a full codebase audit.

## Execution Flow

### Step 1: Gather the Diff and MR Description

Use any available tools and MCP servers to fetch the diff, commit metadata, and the full MR description (MR URL, branch name, title, description, linked tickets). If no tool or MCP is available, fall back to `git diff`. If still empty, stop and tell the user there is nothing to review.

### Step 2: Fetch Design Documents (if provided)

For each document provided (PRD/TRD, BE TD, FE TD), use available tools and MCP servers to fetch the content. Read all embedded images (mockups, diagrams, flow charts) — they often contain requirements not in the text. If a document links to other referenced documents, fetch those too.

Skip this step entirely if no design documents are provided.

### Step 3: Understand the Context

**Repo conventions (extract first):** Read the repo's key files to infer its actual rules — do not assume. Extract the rules the repo follows for: how data is fetched and which layer owns it, how state is managed and with what tools, how components are structured and split, how files and symbols are named, and what shared utilities or modules already exist. These extracted rules become the checklist for **Repo Conventions** findings in Step 4. Do NOT flag linting or formatting issues — those are the linter's job.

**Diff context:** Identify which files changed and group them by domain. For each changed file, read the full file — not just the diff — to understand surrounding context, existing invariants, and what callers expect. Check the git log for related recent changes in the same area to avoid flagging intentional changes or re-raising already-fixed issues.

### Step 3.5: Draw Architecture Flow Charts

Before reviewing individual findings, produce ASCII flow charts that map the structural shape of the changes. Output these charts as part of the report (under **Architecture Overview**) so the reader can orient themselves before reading findings.

Draw **all four** sections that are relevant to the diff. Omit a section only if the diff has zero changes in that area.

**Entry Points** — show which existing pages/components/hooks now trigger new behavior, what condition gates the new path, and what new component or action it leads to.

**New Component Tree** — show the full parent→child hierarchy of every new component introduced. For each leaf or orchestrator, annotate what it reads (store fields, props, composable) and what it calls (API, emit, action).

**Data Flow** — show each API endpoint touched by the diff: HTTP method + path, what triggers it, what state or local variable it populates, and which component consumes that state.

**Shared Lib Changes** — list every shared utility, type, enum, or component modified outside the feature's own directory. For each, one line: what changed and which new feature code depends on it.

Refer to `skills/review-code/mr_flow_chart_example.md` for the format and style.

### Step 4: Review the Changes

Evaluate each change against the categories below. Only flag findings relevant to **what actually changed** — do not audit unchanged code.

**Correctness & Bugs:** Look for logic errors, race conditions, and state mutation where values should be immutable.

**Edge Cases & Error Handling:** Look for API error states not handled, missing loading states, and missing fallbacks for async failure paths.

**Performance:** Look for unnecessary repeated computation that could be cached, expensive operations triggered more often than needed, and N+1 API call patterns.

**Code Quality & Readability:** Look for duplicated logic that already exists in the repo, dead code introduced by the change, magic numbers or strings without named constants, misleading names, and conditions that require double-negation to understand.

**Repo Conventions:** Using the rules extracted in Step 3, flag where the diff solves a problem in a way that is uncommon relative to how the rest of the repo solves the same problem. Only flag when the deviation is meaningful — when it would confuse a teammate or create inconsistency that costs maintenance effort. Do not flag stylistic differences that don't affect how the code is read or composed.

**Security:** Look for user-controlled input interpolated into HTML or SQL without sanitization, credentials or PII logged or exposed, and missing auth checks on gated actions.

### Step 5: Cross-Check Against Design Documents (only if documents were provided in Step 2)

Skip this step entirely if no PRD/TRD, BE TD, or FE TD was provided.

**If PRD/TRD is provided — requirement coverage:** Extract every FE requirement from the PRD that is in scope for this repo and check whether the changed code implements it. Flag requirements present in the PRD but absent from the diff, and behaviors implemented in the diff that have no corresponding PRD requirement (scope creep). Do not flag requirements already implemented before this branch.

**If BE TD is provided — API contract alignment:** Identify every API call in the diff and verify each against the BE TD: endpoint path and method, request parameters, response fields accessed, and error codes handled. Flag calls to endpoints not in the BE TD, response fields accessed that are absent from the BE TD, and mismatched error handling.

**If FE TD is provided — design decision traceability:** Check that the component structure, state management approach, and data flow described in the FE TD are reflected in the changed code. Flag components, hooks, or state slices the FE TD specifies that are missing from the diff, and significant structural deviations — not style, only deviations that would affect behavior.

### Step 6: Output the Report

---

## Code Review Report

### Summary
2–3 sentences. State what changed (files/domains) and the overall assessment (clean / has issues / has bugs).

### Architecture Overview
ASCII flow charts produced in Step 3.5. Always include all four sections (Entry Points, New Component Tree, Data Flow, Shared Lib Changes) that are relevant to the diff. This section comes before Findings so the reader is oriented before seeing individual issues.

### Findings

Use this format for each finding. Categories: `Correctness`, `Edge Cases`, `Performance`, `Code Quality`, `Repo Conventions`, `Security`.

**Category — file:line**
- **Issue:** One sentence describing the bug or problem.
- **Why it matters:** One sentence on the impact or failure mode.
- **Suggestion:** Concrete fix or the minimum change needed. Code snippet if helpful.

For **Repo Conventions** findings, replace **Why it matters** with **Repo norm** — cite where the established pattern lives so the author can see the gap:

**Repo Conventions — file:line**
- **Issue:** What the diff does that differs from the repo's established pattern.
- **Repo norm:** How the repo solves this elsewhere (file or module reference).
- **Suggestion:** The minimal change to align with the norm.

### Design Traceability
(Only include if PRD/TRD, BE TD, or FE TD was provided. Omit sub-sections that are fully aligned.)

**PRD Coverage** (only if PRD/TRD provided):

**[Missing / Scope Creep] Requirement**
- Gap: what is not implemented or what was added without a corresponding PRD requirement.

**BE API Alignment** (only if BE TD provided):

**[Mismatch] API call in file:line**
- Gap: what differs from the BE TD contract.

**FE Design Alignment** (only if FE TD provided):

**[Deviation] Area — file:line**
- Gap: how the implementation differs from the FE TD design decision.

---

## Tone & Style

Be direct: "This will crash when X is undefined" not "This might potentially have issues." No praise or filler. Skip findings that are not actionable. Do NOT rewrite logic that works — suggest only the minimal change needed to fix the issue.
