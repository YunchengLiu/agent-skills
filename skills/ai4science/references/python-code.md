# Python Code

## Implementation

- Write clear, idiomatic Python with direct control flow and explicit data
  transformations. Reuse established APIs; add helpers or abstractions only when
  they simplify a current need.
- Pass required paths and parameters through explicit arguments or simple
  configuration variables. Read the specified input directly; do not silently
  substitute files, materials, datasets, or parameters.
- Validate required formats, units, shapes, and ranges at the boundary that
  owns them. Use concise assertions for internal research invariants, keep
  assertions enabled, and rely on established guarantees downstream.
- Let native exceptions propagate with their tracebacks. Catch only for
  meaningful recovery or required interface translation; manage resources with
  `with` or `finally`. Diagnose failures before retrying, and avoid speculative
  fallback or recovery paths.
- Use types, comments, and docstrings to explain non-obvious interfaces,
  assumptions, units, and numerical conventions; avoid redundant ceremony.
- When files, devices, or execution are inaccessible, preserve the intended
  interface and report what remains unverified. Do not change scientific inputs
  or recovery logic to accommodate agent access limitations.

## Formula Notation and Comments

- In formula-driven code, use the notation of the domain, paper, or algorithm
  when it is clear in the local context and does not conflict with repo
  conventions. Names such as `x`, `y`, `x1`, `y2`, `alpha`, `beta`, `k`, and
  `phi` can be the clearest link between the formula and its implementation.
  Preserve meaningful short names rather than expanding them for length alone.
- A local mathematical function can retain its name from the formula. Give a
  function with an independent computational role a name that identifies that
  role; retain existing clear names during review or refactoring.
- For a core calculation, briefly state what it computes, the corresponding
  formula, and the units or conventions needed to read it. Add a source or
  equation number for a non-obvious external method. Use readable text or math
  notation; include derivation details only when needed to understand the code.

For example, with positive `sigma` established at the owning input boundary:

```python
import numpy as np


def gaussian(x, y, sigma):
    """Evaluate g = exp(-(x² + y²) / (2σ²)); x, y, sigma share length units."""
    return np.exp(-(x**2 + y**2) / (2 * sigma**2))
```

The symbols and formula make this local calculation readable without verbose
parameter names or repeated boundary checks. Follow the repo's comment language
and document additional array conventions when they affect the interface.

## Ruff Configuration

- Use Ruff as the standard Python linter and formatter. On the first relevant
  code change in a maintained repo without Ruff settings, merge
  [`../assets/ruff-pyproject.toml`](../assets/ruff-pyproject.toml) into its
  `pyproject.toml`; create a minimal file if needed. Do not add packaging, hooks,
  or CI solely for linting. Run Ruff directly on one-off scripts.
- Respect existing project-specific settings. Use the shared template as the
  default when introducing or updating Ruff configuration, and resolve material
  differences in the project's context. Derive Python compatibility from the
  project's requirements.
- Ruff is normally available in the research environment. If missing,
  recommend installing it and obtain approval before changing the environment.
  If declined, manually check comparable issues where practical and report that
  Ruff was not run.

## Lightweight Verification

- After implementing or refactoring Python code, run `ruff check` and
  `ruff format --check` on the affected files. Use safe automatic fixes and
  formatting where useful, review the resulting diff, and rerun affected checks.
- Treat the configured Ruff checks as the lint and formatting acceptance
  standard. Do not add subjective style gates or suppress valid diagnostics to
  obtain a pass.
- Run relevant `pytest` tests, representative numerical calculations, or
  `ty check` when they add evidence for the changed behavior. Static checks do
  not establish scientific correctness.
- Keep checks proportional to the change and avoid unrelated repository-wide
  cleanup. When required behavior and applicable checks pass, finish; report
  meaningful unresolved issues or verification that could not be performed.
