---
name: review-code
description: Review the current branch's code changes for correctness bugs, missing edge cases, performance issues, and code quality. Optionally cross-checks implementation against a PRD, BE TD, and/or FE TD to verify requirement traceability. Surfaces findings as a structured report. Use when you want a thorough code review of local changes before pushing, or to validate an MR before merging.
---

# Review Code

Review the **current branch's changes** (or a specific MR) for bugs, correctness issues, performance problems, and code quality concerns. When design documents are provided (PRD, BE TD, FE TD), also verify the implementation actually delivers what was designed — requirement traceability in both directions.

The review is surgical — focused on what changed, not a full codebase audit.

## Execution Flow

### Step 1: Gather the Diff

Use any available tools and MCP servers to fetch the diff and commit metadata (MR URL, branch name, title, description).

If no tool or MCP is available, fall back to use `git diff`

If still empty, stop and tell the user there is nothing to review.

### Step 2: Fetch Design Documents (if provided)

For each document provided (`prd`, `be_td`, `fe_td`):
- Use available tools and MCP servers to fetch the content.
- Read all embedded images (mockups, diagrams, flow charts) — they often contain requirements not in the text.
- If a document links to other referenced documents, fetch those too.

Skip this step entirely if no design documents are provided.

### Step 3: Understand the Context

**Repo conventions (extract first):**
- Identify the repo's established patterns by reading key files: state management structure, API layer conventions (how services/hooks are named and organized), component patterns, file and folder naming.
- This becomes the baseline for flagging convention deviations in the diff.
- Do NOT flag linting or formatting issues — those are the linter's job, not this review's.

**Diff context:**
- Identify which files changed. Group them by domain (e.g. components, hooks, API layer, tests, config).
- For each changed file, read the full file (not just the diff) to understand surrounding context: how the changed code fits into the larger module, what invariants exist, what the caller expects.
- Check the git log for related recent changes in the same area to avoid flagging intentional changes or re-raising already-fixed issues.

### Step 4: Review the Changes

Evaluate each change against the categories below. Only flag findings relevant to **what actually changed** — do not audit unchanged code.

**Correctness & Bugs:**
- Logic errors
- Race conditions
- State mutation: mutating shared objects or values that should be immutable

**Edge Cases & Error Handling:**
- API error states not handled (no error branch, no loading state)
- Missing fallback for async failure paths

**Performance:**
- Unnecessary repeated computation that could be cached or memoized
- Expensive operations triggered more often than needed
- N+1 API calls: looping over items and making one request per item

**Code Quality & Readability:**
- Duplicated logic that already exists in the repo (check for existing utilities or shared modules)
- Dead code introduced by the change (imported but unused, unreachable branches)
- Magic numbers/strings without named constants
- Variable or function names that are misleading given what the code actually does
- Complex expressions or deeply nested logic that could be extracted into a well-named variable or function
- Conditions that read backwards or require double-negation to understand

**Security:**
- User-controlled input interpolated into HTML or SQL without sanitization (XSS, injection)
- Credentials, tokens, or PII logged or exposed in responses
- Permissions or auth checks missing on a gated action

### Step 5: Cross-Check Against Design Documents (only if documents were provided in Step 2)

Skip this step entirely if no `prd`, `be_td`, or `fe_td` was provided.

**If `prd` is provided — requirement coverage:**
- Extract every FE requirement from the PRD that is in scope for this repo.
- For each requirement, check whether the changed code implements it.
- Flag: requirements present in the PRD but absent from the diff (not implemented).
- Flag: behaviors implemented in the diff that have no corresponding PRD requirement (scope creep or undocumented feature).
- Do not flag requirements already implemented before this branch — only assess what the current diff adds or changes.

**If `be_td` is provided — API contract alignment:**
- Identify every API call in the diff (service calls, HTTP clients, data-fetching utilities).
- For each call, verify against the BE TD:
  - Endpoint path and HTTP method match the BE TD definition.
  - Request parameters (query params, body fields, headers) match the BE TD contract.
  - Response fields accessed in the code exist in the BE TD's response shape.
  - Error codes/states handled in the code match what the BE TD defines.
- Flag: calls to endpoints not defined in the BE TD.
- Flag: response fields accessed that are absent from the BE TD response shape.
- Flag: error handling for codes the BE TD does not define (or missing handling for codes it does define).

**If `fe_td` is provided — design decision traceability:**
- Check that the component structure, state management approach, and data flow described in the FE TD are reflected in the changed code.
- Flag: components, hooks, or state slices the FE TD specifies that are missing from the diff.
- Flag: significant deviations from the FE TD's design (different state shape, different component split, different API call trigger point) — not style deviations, only structural ones that would affect behavior.
- Flag: code in the diff that contradicts an explicit FE TD decision without an obvious reason.

### Step 6: Output the Report

---

## Code Review Report

### Summary
2–3 sentences. State what changed (files/domains) and the overall assessment (clean / has issues / has bugs).

### Findings

Use this format for each finding:

**Category — file:line**
- **Issue:** One sentence describing the bug or problem.
- **Why it matters:** One sentence on the impact or failure mode.
- **Suggestion:** Concrete fix or the minimum change needed. Code snippet if helpful.

### Design Traceability
(Only include if `prd`, `be_td`, or `fe_td` was provided. Omit sub-sections that are fully aligned.)

**PRD Coverage** (only if `prd` provided):

**[Missing / Scope Creep] Requirement**
- Gap: what is not implemented or what was added without a corresponding PRD requirement.

**BE API Alignment** (only if `be_td` provided):

**[Mismatch] API call in file:line**
- Gap: what differs from the BE TD contract.

**FE Design Alignment** (only if `fe_td` provided):

**[Deviation] Area — file:line**
- Gap: how the implementation differs from the FE TD design decision.

---

## Tone & Style

- Use bullet points — one idea per bullet, short sentence.
- Be direct: "This will crash when X is undefined" not "This might potentially have issues."
- No praise or filler. Skip findings that are not actionable.
- Do NOT rewrite logic that works — suggest only the minimal change needed to fix the issue.
