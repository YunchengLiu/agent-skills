---
name: cpp-project-engineering
description: >
  Apply detailed personal C++ project conventions when implementing, refactoring,
  or reviewing source, comments, tests, errors, logging, CMake, and delivery.
  Covers how to organize, explain, integrate, and verify a complete change.
---

# C++ Project Engineering

Use these conventions for modern C++ projects. Read the references for the
surfaces being changed; project-specific contracts and configuration take
precedence.

For a repository entry point, copy [AGENTS.md](assets/AGENTS.md) into a new
project, or merge its routing into existing instructions. Add the project's
configuration and contracts there; keep the detailed shared rules here.

## Working Principles

- Derive the change from the required behavior, observable acceptance signal,
  existing execution path, consumers, and the responsibility still missing.
- Make the smallest complete change: implementation, affected callers,
  documentation, tests, and build registration must agree.
- Establish each property where the necessary information is owned; later
  stages rely on that guarantee.
- Use modern C++ and the configured libraries to express ownership, lifetime,
  and behavior directly. Respect the effective standard and toolchain.
- Treat the user's settled behavior, API shape, failure form, security, scale,
  and recovery guarantees as the design inputs. Implementation must be free of
  undefined behavior, lifetime and resource errors, and fragile constructs.
- Add an abstraction when it removes current complexity, encodes a present
  responsibility, or addresses an existing dependency boundary. Explain material
  architecture choices and resolve them with the user before implementing.
- Protect actual external API, ABI, installed/exported, and serialization
  commitments. A breaking change requires impact analysis, migration, rollback,
  and confirmation. Internal changes update all affected uses coherently.
  Reachable implementation-support headers are installed for compilation and
  do not thereby become stable interfaces.
  When an interface is clearly externally reachable but its stability is
  unclear, report the evidence and ask before breaking it or adding a shim.
- Apply first-principles reasoning and ablation as decisions arise and at each
  stage: compare each addition with removal, reuse, or a simpler form against
  the required outcome. Retain required invariants, useful explanations, and distinct
  evidence. Keep routine checks brief and reuse supported decisions unless the
  relevant facts change or a concern remains. Audit each retained changed line
  against applicable mechanical rules. Use `first-principles` for the full
  method when available; these principles also apply when using this skill alone.

## Design And Implementation

Work as the user's design-and-coding copilot: keep decisions and results easy
to follow and review. Production code, tests, and build changes are non-trivial
by default; scale design and closeout to the change. Only the user may designate
a specific typo, rename, or one-line comment/format task as trivial.

### Establish The Context

Read relevant instructions, code, tests, dependencies, and build configuration.
Identify the current behavior, effective standard, owning module, consumers,
and any constraints on exceptions, allocation, threading, or exported interfaces.
In an unfamiliar area, read an established project example for comment depth,
test style, file layout, and CMake wiring.

Assume readers know C++ and the basic subject, but not the component's specific
model or conventions. Public declarations and documentation must suffice for
callers; implementation and local comments together must let maintainers follow
the execution and understand its key reasons. Apply the reader and placement
rules in [comment-style.md](references/comment-style.md).

Use clear, idiomatic language in explanations and collaboration. State the
required behavior and conditions directly; use prohibitions for concrete
constraints. Present effective requirements and reasons. Discussion corrections
and temporary drafting notes must stay out of lasting documentation. For
comments, establish the content and structure before polishing the wording;
preserve both the facts and the mechanical rules.

### Shape The Change Top-Down

For both settled and unsettled designs, trace the task from input or entry
through the owning operation to its result and consumers. Establish behavior,
acceptance, the caller's view, existing facilities, and where guarantees are
established and consumed. Identify the remaining responsibility before choosing
private mechanics. Organize the work as evidence, owning operation, contract,
integration, then implementation detail. Carry these facts and boundaries into
implementation and review; reopen them only for a concrete gap or conflict.

When the design is unsettled, resolve missing behavior and boundaries. If
credible alternatives differ in observable behavior, public surface,
dependencies, or architecture, present at least two with pros and cons and
await confirmation. Make local mechanical choices directly.

Before creating an independent construct or changing its file ownership, name
declaration, implementation, test, and build-registration locations.

When a design is settled, use that design and write the applicable landing plan.
Check it against repository reality. An existing component's established
semantics and conventions are part of the settled design: a new member follows
them, and a case they already decide is not a reason to reopen them. Reopen only
a conflicting point supported by code, an existing contract, or a reproducible
failure. Present 1-3 grounded options with a recommended tradeoff; proceed on
independent parts and identify the blocked point explicitly. If missing
observable behavior blocks implementation, ask the user to choose. Make local
mechanical choices directly.

When a spec defines phases, complete the agreed phase and report decisions,
deviations, validation, and risks before moving to the next, unless several
phases were requested together.

### Implement And Verify

The authorized request and its necessary ripple define the scope: the affected
callers, documentation, tests, build registration, and other artifacts the
change requires, and the changes repository rules require for that work. Every
changed line and new construct must trace to that scope, including concrete
failures the work causes or that block it. Use the established context to judge
each part's contribution: names, types, comments, and code carry meaning
together, and downstream code uses guarantees already established at their
owner. Add a mechanism, qualification, or check for a remaining need; local
self-containment alone does not justify it. Applicable hard rules still govern
every retained part.

When concrete evidence shows that the request needs work beyond that scope,
report the expansion and obtain confirmation before making it; a report alone
grants no authorization. Report unrelated defects and improvement opportunities
separately without changing them. When the user corrects a premise, design
choice, or scope, recheck the implementation, tests, comments, configuration,
and remaining plan derived from it; revise what the correction invalidates and
keep the work that remains valid.

Keep the touched area coherent with the references below. Add tests for changed
behavior and meaningful supported boundaries. Verify third-party APIs against
the dependency manifest and target graph before using them.
For API documentation, prefer Context7 when available; otherwise use library
documentation, source, or web research to resolve uncertain API details.

Source comments follow the project language rules, defaulting to zh-CN;
runtime-visible strings use English. Accurate user-authored comments retain
their wording. Keep deliverables complete and free of placeholders, dead
code, unexplained fallback paths, secrets, and task-history text.

Keep refactors behavior-preserving unless behavior changes are part of the
request. For a failure-driven refactor, explain the root cause and correct the
design; do not silence the failure, weaken a valid test, or shift the problem to
a third party. Complete the closing checks in
[validation-and-review.md](references/validation-and-review.md) before delivery.

## References

Read the matching Hard Checklist or Rules first, then the relevant details.
The requirements combine when several surfaces are touched.
Deviate only when project instructions, a closer rule, or the user explicitly
requires it; report the deviation and its reason before proceeding.

| Surface | Reference |
| --- | --- |
| C++ naming, layout, includes, helpers, files, and landing plans | [code-structure.md](references/code-structure.md) |
| Comment model, Doxygen, mechanical format, and examples | [comment-style.md](references/comment-style.md) |
| Test coverage, GoogleTest names, comments, and assertions | [test-style.md](references/test-style.md) |
| Error forms, propagation, translation, diagnostics, and failure tests | [error-handling.md](references/error-handling.md) |
| Public standard-library formatting and internal fmtlib | [formatting-boundary.md](references/formatting-boundary.md) |
| Targets, sources, UTF-8, headers, dependencies, and install/export | [cmake-and-install.md](references/cmake-and-install.md) |
| Runtime logging ownership, levels, messages, and cost | [logging.md](references/logging.md) |
| Verification, implementation closeout, and evidence-based review | [validation-and-review.md](references/validation-and-review.md) |
