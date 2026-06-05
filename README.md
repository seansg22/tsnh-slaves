# tsnh-slaves

Agent skills for frontend engineers. Works with Claude Code, Cursor, Codex, and other AI coding agents.

## Install

```bash
npx skills@latest add seansg22/tsnh-slaves
```

## Skills

### review-prd
Analyze a PRD from a frontend developer's perspective: extract the UI/FE requirements that impact the repo, and surface ambiguities or missing details before work begins.

### review-be-td
Review a Backend Technical Design from a frontend developer's perspective: verify the BE TD exposes everything the FE needs, and surface gaps or mismatches before FE development begins.

### review-fe-td
Review a Frontend Technical Design against its PRD and BE TD to verify UI requirement coverage, API contract alignment, and surface gaps before FE implementation begins.

### review-code
Review the current branch's code changes for correctness bugs, missing edge cases, performance issues, and code quality. Optionally cross-checks implementation against a PRD, BE TD, and/or FE TD to verify requirement traceability.
