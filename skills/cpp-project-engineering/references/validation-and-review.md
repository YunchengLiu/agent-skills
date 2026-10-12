# Validation And Review

## Choose Evidence For The Change

Choose checks from the behavior and integration affected by the task. Each
check should cover a distinct acceptance signal or risk.

| Change | Useful evidence |
| --- | --- |
| Documentation or instructions | Semantic review, links, examples, and applicable lint |
| Source comments or layout | Comment review and configured formatter |
| Behavior or implementation | Build and tests for the affected behavior and integration |
| Public headers or dependency wiring | Build and a consumer check at the affected boundary |
| Installation, export, or packaging | Install/package and its real consumer path |
| Performance or concurrency | Reproduction or measurement under the stated workload |

Use the project's existing presets, environment entry points, formatter, and
test suites. Select a representative development configuration; add compilers,
platforms, release, sanitizers, and other configurations where they check a
supported requirement that remains unverified.

Follow the project's iteration, pre-commit, and comprehensive-validation
checkpoints. During iteration, build the affected targets and run the tests or
executions needed for the changed behavior and integration. Expand to affected
consumers when the change crosses their boundary. Formatting belongs at the
project's specified checkpoint, or before delivery when none is specified;
it need not precede every development build unless the task or tooling requires
it. Complete the project's required integration scope at its delivery or
pre-commit checkpoint. After a fix, resume from failed or invalidated steps;
reuse valid evidence and repeat or broaden checks only for a changed artifact,
failure, unresolved concern, or unmet project requirement.

## Static Analysis

Run expensive lint once functionality is stable and the task calls for final
validation. Use the configured entry point and select affected production
translation units, including those that instantiate changed headers.

Fix valid diagnostics and use a local suppression for demonstrated tool noise
under project policy. Report ambiguous diagnostics or design changes for
judgment. For wrapped or multi-line code, prefer `NOLINTNEXTLINE` or
`NOLINTBEGIN` / `NOLINTEND`; use `clang-format off/on` only for the
smallest region where formatting would otherwise detach the suppression.
Inspect its placement after formatting; do not rerun analysis solely because
formatting ran. Repeat analysis when a code change or unresolved diagnostic
calls for new evidence.

A compiler internal error is a tool failure. Preserve the source and build
configuration needed to reproduce it, retry according to the project workflow,
and report a persistent crash; do not change flags to hide it.

## Review

Read the diff with the declarations, callers, tests, and build wiring needed to
understand the task's complete behavior path and its consequences. Establish
which responsibilities and guarantees the surrounding system already fulfills
before judging local parts. A proposed extra check, abstraction, name qualifier,
or explanation needs a concrete gap in that context. Bias review attention in
roughly this order:

- committed APIs, ABI, serialization, installation, and consumer use;
- direct behavioral regressions, ownership, lifetime, ordering, error handling,
  and dependencies or responsibilities crossing module boundaries;
- numeric, resource, concurrency, and platform boundaries that the task touches;
- tests that prove the project's behavior and meaningful failures;
- adequacy of the validation evidence;
- necessity of each added construct and design concerns with a concrete
  near-term risk.

For implementation diffs and pre-delivery reviews, also check Implementation
Closeout steps 1-3 below. If the original landing plan is unavailable, inspect
the actual declaration, implementation, test, and build locations directly.

A finding needs a concrete consequence supported by the artifact. For ambiguous
wording, state the plausible readings and why the intended reader could choose
the wrong one. Treat missing evidence as uncertainty or a validation gap.
Check that each finding and its severity would remain the same without the
assignment's suggested conclusion or concern.

Report findings first, in severity order, with file/line evidence, the
consequence, and the smallest correction or verification when evident.
State explicitly when there are no substantive findings. Follow
with material questions and evidence gaps; optional improvements remain
secondary. Reuse existing validation evidence and keep review actions within the
requested scope.

## Implementation Closeout

Before delivery, perform the four checks below in order. Formatting, build,
test, lint, or static-analysis evidence does not replace them. Fix findings from
steps 1-3 before delivery; report a genuinely deferred item as a risk. For pure
documentation or planning work, apply only the checks that match the changed
artifact.

### 1. Construct Inventory

Confirm the actual change delivers the agreed outcome along its owning path
and consumers. Check that local choices still use the established context and
guarantees, and that any added responsibility has a current requirement or
concrete gap. Use this whole-path assessment for the inventory and later checks.

List each independent construct that the change created or whose file ownership
changed. Confirm its current requirement and the applicable declaration,
implementation, test, and build-registration landing points; mark a point N/A
only with a concrete reason. Helpers, state fields, comments, options, and other
non-construct additions are still checked for necessity and ownership in step
3, but do not acquire four artificial landing points.

### 2. Ripple Check

List every symbol whose behavior changed. For each:

- inspect its declaration and contract comment, confirm that they state the
  current behavior, and ensure a new or changed non-obvious component explains
  its role, result, and place in the surrounding system without requiring the
  reader to reconstruct its implementation;
- fix comments that the change falsified, preserve accurate and relevant
  maintenance comments with their logic, and apply the preservation rules in
  `comment-style.md`; check new or migrated implementation for missing
  non-obvious maintenance explanations as well as declaration comments;
- for behavior visible to other modules or callers, search the repository for
  comments and examples that describe the old behavior and fix those the change
  falsified; for behavior private to one implementation file, bound the search
  by its paired declaration and tests;
- name the test that proves the new behavior, or report the gap;
- classify compatibility from actual evidence: user or design requirements,
  supported install or export status, publication and stability promises,
  external consumers, and stable serialized or ABI forms. Confirm committed
  surfaces remain compatible or that the user approved the break. Do not infer
  stability merely because an implementation-support header is installed, and
  do not add a shim solely for an uncommitted predecessor;
- review dependency-shape and runtime-visible-string changes as explicit ripple
  surfaces. Confirm each is required and preserves any applicable commitment,
  or that the user approved the break.

Also search affected callers, implementations, tests, bindings, generated
artifacts, public and installed headers, exports, packages, runtime deployment,
and serialized or wire forms. A local green test does not prove a cross-boundary
change is coherent. If the change removes the last use of a dependency, remove
its include as well.

### 3. Line-Level Rule And Ablation Review

Inspect every changed line against the references routed by the main skill and
the project's closer rules. For each material construct and validation action,
compare its removal or the simpler baseline. Remove it when behavior and
independent evidence remain. Do not weaken a hard rule, explicit guarantee, or
project-required check to reduce the diff.

For a mechanical checklist, compare each changed line with its removal or the
simpler form, then check every retained line against that checklist.

When a defect is found during the work or a review, including one the user
reports, identify its root cause or the rule it violates, then check the whole
current diff, including new files, for the same class. When implementing, fix
the instances within the authorized scope together with their affected callers,
tests, and documentation, and report the others; a review reports all of them.

### 4. Evidence

Record exactly which formatter, builds, tests, static analysis, release,
sanitizer, install, package, hardware, or manual checks ran. State failures,
limitations, and intentionally skipped lanes. Do not convert “not run” into an
implicit pass.

## On-Demand Conformance

When asked to check conformance, the implementing agent repeats closeout steps
1-3 on the requested scope and reports each finding as
`file:line | reference section | conclusion`.

## Delivery

Report the completed changes and their reasons, validation that actually ran,
and key design decisions. Add minimal usage examples and tests when helpful;
update documentation for API changes. Report actual performance, security, and
compatibility risks, rollback, and verification gaps; state when none apply.
Use the project's required report shape. Describe evidence as checkable facts;
a bare claim that self-checks passed is insufficient.

During small back-and-forth corrections, do not repeat the full report.
Do not paste full-file replacements in the response unless the user asks.

When committing is requested, use the project's convention and derive the scope
from the owning target or delivered component.
