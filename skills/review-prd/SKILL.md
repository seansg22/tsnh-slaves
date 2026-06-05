---
name: review-prd
description: Analyze a PRD to extract frontend engineering requirements scoped to the current repo, and surface ambiguities or missing details that should be clarified before work begins. Use when the user shares a PRD (URL, Confluence link, file, or raw text) and wants a structured breakdown of what the frontend needs to change.
---

# Review PRD

Analyze a PRD from a **frontend developer's perspective**: extract the UI/FE requirements that impact this repo, and surface open questions that must be answered before development starts.

---

## Execution Flow

### Step 1: Fetch & Parse the PRD

Use available tools and MCP servers to fetch the PRD content from whatever source the user provided (URL, Confluence link, file path, or raw text). Also use available tools and MCP servers to fetch and read all images embedded in the PRD (mockups, diagrams, screenshots) — they often contain requirements not written in the text.

For every Figma link found in the PRD (design specs, prototypes, component links), fetch and read the Figma design using available Figma MCP tools. Extract layout, component structure, spacing, colors, interaction states (hover, focus, error, empty, loading), and any annotations. Treat Figma designs as the source of truth for visual and interaction requirements.

After fetching the main PRD, scan it for references to other PRDs (linked URLs or named documents). For each referenced PRD, fetch its content, read all its images and Figma links, and fetch all their MRs as well.

Fetch the current repo's recent MRs (last 6 months) for ongoing work that may intersect with the current PRD.

### Step 2: Understand the Repo Context

- Get a lightweight picture of what the current repo owns: pages, components, state management, API layer, key domain concepts.
- This anchors which PRD requirements are in-scope for this FE repo vs. owned by other repos/teams.
- Scan recent MRs from the last 6 months to identify ongoing projects: their purpose, the areas of the codebase they touch, and their current status. Use git and available repository tools and MCP servers for this.

### Step 3: Extract FE Requirements

For each requirement in the PRD, focus on what the **frontend** needs to build or change:
1. Determine if it touches this FE repo (in-scope) or is owned by another team (backend, other FE repo, etc.). Discard out-of-scope items — do not list or elaborate on them.
2. For in-scope items, classify:
   - **New feature** — net-new page, component, or user flow
   - **Change** — modification to existing UI behavior, display logic, or user interaction
   - **Removal** — deprecation or deletion of existing UI
   - **Config / flag** — feature flag, A/B experiment, or environment config
3. **Group related requirements** — e.g. all changes to one page, all new components, all routing/config changes — so the table stays concise and scannable rather than one row per micro-detail.
4. Map each group to the likely affected area (page, component, hook, state slice).

### Step 3.5: Flag Design vs. Implementation Misalignments

Skip this step entirely if the PRD explicitly mentions updating the UI or redesigning visuals — those misalignments are intentional and already accounted for in the requirements.

Only run this step when the PRD makes no mention of UI design changes (e.g. it's a purely functional or data change). In that case, compare the design assets from Step 1 against the current implementation to surface regressions or drift the PRD author may not have noticed.

For each mismatch, record:
- What the design specifies
- What the current implementation does
- Which file/component is affected

Focus on user-visible differences only: layout, copy, component states, interaction flows, missing elements, extra elements, or wrong defaults. Ignore minor pixel-level variance — flag structural and behavioral differences that a user would notice or a PM would care about.

If no Figma or design assets were found in the PRD, skip this step.

### Step 4: Surface Concerns

Each concern maps to one table row with columns: #, PRD Section, Issue, Question.

Concerns are about **what the PRD requires** — not about how to implement it. Do not raise concerns about component choice, API design, field names, endpoint names, code structure, or any other technical detail. If the PRD says something should be displayed or done, that is enough — questions about *how* are for the engineer to solve during implementation.

A concern is valid only if the **product behavior itself** is unclear: what should happen, when, for whom, or under what conditions.

Common concern triggers:
- A requirement is contradicted or duplicated elsewhere in the PRD
- A user-facing behavior or edge case is left unspecified (e.g. what happens on error, what the default state is, what empty state looks like)
- The scope boundary is ambiguous — it is unclear whether a UI element or flow is this repo's responsibility
- A phasing or timeline conflict (e.g. a feature is listed in both Phase 1 and Phase 1.5)
- A dependency on BE or another team's deliverable is implied but not explicitly confirmed in the PRD
- PRD touches the same area as an ongoing project (branch/MR from Step 2) — flag the conflict, name the branch/MR, and ask who owns sequencing

### Step 5: Output the Report

Produce a structured report with these sections:

---

## PRD Analysis Report

### Summary
2–3 sentences maximum. State what this FE repo specifically needs to build. Skip background, context, and anything owned by other teams.

### In-Scope Requirements

Group related items (e.g. all changes for one page, all new components, all config changes) — do not list every micro-detail as a separate entry. Use this format for each group:

**[Type] Requirement Group**
Affected area: page, component, or hook

### Design vs. Implementation Misalignments

List mismatches found between the PRD's design assets (images, Figma) and what is currently implemented. Omit this section if no design assets were found. Use this format for each mismatch:

**[Component/Page]** What the design shows vs. what the current code does. File: `path/to/file.tsx`

### Open Questions

Only include concerns that directly affect what this FE repo needs to build. Discard concerns that are purely about other teams' scope. Order by priority (blocking first, then high risk, then nice-to-have clarity). Use this format for each concern:

**[PRD Section]** Issue description and specific question to clarify.

---

## Tone & Style

- Use bullet points heavily — one idea per bullet, short sentence.
- Avoid long prose paragraphs. If a thought needs more than one sentence, break it into bullets.
- Be direct and concrete; avoid vague statements like "may need changes."
- Concerns are for genuine ambiguities only — if something is clear, skip it.
- Do NOT generate a TD or write any code — this skill is analysis only.
