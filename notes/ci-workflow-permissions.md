# CI Workflow Permissions – Local Pipeline Rationale

_Context: Turn 4 / lacardlabs CLI_

## Background

- Reusable workflow `LacardLabs/.github/.github/workflows/ci.yml@main` currently runs CodeQL as part of its job matrix.
- CodeQL requires the `security-events: write` scope and the ability to invoke the workflow-run API.
- Repositories that consume the reusable workflow must explicitly grant these permissions in each referencing workflow. Without them, runs fail before the first job executes.

## Options considered

### 1. Grant elevated permissions (keep the reusable workflow)
- ✅ Minimal change: add `permissions:` block in local workflow and continue using the shared definition.
- 🔻 Widens the trust boundary. Any future change to `LacardLabs/.github` inherits those privileges automatically.
- 🔻 Harder to debug when failures happen inside the shared workflow; errors surface as opaque permission faults.
- 🔻 Not every repo needs CodeQL; granting org-wide security scopes for simple unit tests is disproportionate.

### 2. Inline repo-specific workflow (chosen)
- ✅ Keeps permissions at GitHub’s default for the repo (checkout, tests only).
- ✅ CI is self-contained; failures are easier to trace and contributors can reproduce locally.
- ✅ Avoids the cross-repo dependency and accidental execution of jobs irrelevant to this project.
- 🔻 Slight duplication: each repo defines its own minimal test workflow.

## Decision

Inline a lightweight workflow in `lacardlabs/.github/workflows/ci.yml`:
- Checkout
- `actions/setup-python@v5` (3.12)
- `pip install -e .[dev]`
- `pytest -q`

This mirrors the test matrix we actually care about while leaving future room to add linting or packaging jobs **within the same repo** where permissions remain scoped.

## Follow-up

- If we later want org-wide CodeQL or additional reusable jobs, introduce them as optional steps that can run without elevated scopes, or host them within this repo where permissions are explicit.
- Communicate this guideline in PR reviews to avoid reintroducing the shared workflow without matching permissions.
