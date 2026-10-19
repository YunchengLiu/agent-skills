---
name: modern-cpp
description: >
  Choose and apply modern C++ language and standard-library facilities for
  clearer interfaces, ownership, lifetime, generic code, and safer everyday
  implementation. Use for C++ implementation, review, focused modernization,
  or standard and safety-tool decisions where these choices affect the result.
---

# Modern C++

Choose supported modern C++ facilities that improve correctness, ownership,
interfaces, or implementation clarity. Apply this judgment to both new code
and focused improvements in existing code.

Apply this skill to the C++ portion of mixed-language projects and to older
baselines within their supported facilities. Pure C work and build-system-only
design belong to their own workflows. Compiler and library diagnostics here
support C++ decisions; changing build configuration remains a scoped project
change.

## Working Defaults

Derive choices from required behavior, ownership, consumers, and toolchain
support. At design, implementation, and validation, compare each choice with
its removal or the simplest complete alternative. Retain what adds correctness,
clarity, or useful evidence while preserving contracts and project rules. When
a material tradeoff needs measurement, hold other conditions fixed and compare
the same acceptance signal.

- Establish the effective standard from the user's instruction, project
  declaration, build settings, and compiler/library evidence, in that order.
  Ask when material evidence remains missing or conflicting. Evaluate C++26
  only when explicitly requested.
- Verify implementation-sensitive facilities with feature-test macros, vendor
  documentation, or a focused compile probe; a language-mode flag alone does
  not prove library support.
- Respect runtime constraints and settled behavior. If hosted/freestanding
  operation, exceptions, RTTI, allocation, threading, debugging, sanitizers,
  public headers, or ABI remain unclear and affect the choice, ask before
  proposing library-heavy or boundary-crossing modernization.
- Prefer value semantics and RAII for ownership, views for clearly bounded
  borrowing, standard algorithms for sequence operations, and types that
  express meaningful states and results.
- Use the clearest supported form in touched code. Keep control flow,
  allocation, lifetime, and diagnostic costs understandable; keep state and
  behavior together when they share ownership and invariants.
- Match existing local style and keep changes focused. Preserve actual API,
  ABI, and consumer commitments at the boundaries that carry them; internal
  code and its callers can evolve together. Separate behavior changes from
  modernization unless the task includes them.
- Describe interfaces for callers: purpose, inputs, results, and relevant
  ownership, lifetime, and failure semantics.

## Work from the Required Behavior

1. **Establish the profile.** Identify the effective standard, active compiler
   and library, relevant callers, ownership, failure contract, and runtime
   constraints. Use repository evidence before asking for missing decisions.
2. **Choose the useful change.** For new code, express the model directly with
   supported facilities. For modernization, identify the existing behavior
   and the concrete defect, ambiguity, or maintenance cost being removed.
   Address ownership, lifetime, invalid states, and synchronization before
   cosmetic syntax changes when those issues are present in the requested scope.
3. **Apply a contained slice.** Update the representation and affected callers
   together. Use the task references below for selection criteria and examples;
   a replacement table is a set of candidates, not a rewrite mandate.
4. **Verify the claimed benefit.** Compile with the actual supported toolchain
   and exercise the affected behavior. Check invalid input, empty states,
   lifetime, cancellation, or allocation failure when those paths matter.
   Select diagnostics and measurements for the specific uncertainty.

Separate availability from suitability: a supported local improvement can be
used now; changes to ownership, error handling, concurrency, or consumer
requirements need a settled design first. Hardening and diagnostics can help
without a standard upgrade; select them for the platform and task. Novelty
alone does not justify architecture.

## Choose Facilities for the Task

Use these as starting points, then select from the effective standard's
facilities. Read only the references relevant to the task.

- **Ownership and interfaces:** use RAII, standard ownership types,
  `std::string_view`, and `std::span` to make storage and borrowing clear;
  use `std::optional`, `std::variant`, or `std::expected` for the state or
  result model they actually express. Read
  [adoption-principles.md](references/adoption-principles.md).
- **Everyday implementation:** use scoped initialization, structured bindings,
  named algorithms, ranges, and standard container operations to replace
  distracting mechanics. Apply the modern-expression, lifetime, efficiency,
  and readability lenses in
  [code-modernization.md](references/code-modernization.md).
- **Generic code and callbacks:** use templates, concepts, `if constexpr`, and
  standard invocation facilities to express the required operations. Read
  [style-principles.md](references/style-principles.md) for ownership,
  abstraction, and callback defaults.
- **Changing an existing representation:** compare candidates, conditions,
  and verification in
  [migration-patterns.md](references/migration-patterns.md).
- **Correctness and safety:** examine resource cleanup, view lifetimes, state
  access, integer operations, iterator validity, and synchronization in
  [safe-idioms.md](references/safe-idioms.md). Use these checks for the touched
  code and expand to a security review only when the task calls for one.
- **Diagnostics and hardening:** choose relevant warning, sanitizer, library,
  and binary checks from [compiler-hardening.md](references/compiler-hardening.md)
  when diagnosing undefined behavior, validating a safety-sensitive change,
  or evaluating requested hardening. Apply configuration changes only within
  the authorized scope.
- **Choosing a baseline or working under runtime constraints:** read
  [standard-selection.md](references/standard-selection.md) and
  [project-profile.md](references/project-profile.md) as needed.

The version references explain facilities introduced by each standard and
when to use them. Earlier facilities remain candidates at a newer baseline.

- [C++17](references/cpp17.md): scoped state, vocabulary types, and simpler
  compile-time branching.
- [C++20](references/cpp20.md): concepts, ranges, spans, formatting, comparisons,
  initialization, synchronization, and coroutine boundaries.
- [C++23](references/cpp23.md): results, formatting, explicit object parameters,
  flat containers, range materialization, generators, and library modules.
- [C++26](references/cpp26.md): feature-specific candidates when the user
  explicitly targets C++26 or asks about reflection, contracts, execution,
  hardening, or its smaller language and library additions.

## Check the Evidence

- Inspect ownership and access preconditions even when the code uses standard
  containers or smart pointers. These types reduce particular risks; they do
  not establish every lifetime, bounds, or concurrency guarantee.
- Distinguish successful compilation, useful diagnostics, exercised sanitizer
  coverage, and measured performance. Each supports a different claim.
- Qualify cost claims by workload and implementation. A wrapper's storage,
  deleter, reference counting, allocation, or template instantiation may matter;
  neither newer syntax nor a published benchmark proves a local speedup.
- Keep dated compiler-version tables and fleet-wide performance figures out
  of permanent adoption rules.

## Reporting

Briefly state the effective standard and its basis, the practical benefit,
evidence that the selected form fits, relevant lifetime, support, diagnostic,
or cost tradeoffs, and validation performed. Recommend one approach when
several are viable. State whether to apply it now, defer it, or record a future
migration; separate broader follow-up work from the requested change.

## Further Reading

The [Trail of Bits Modern C++ skill](https://github.com/trailofbits/skills/tree/5c99c03fb8a9ea568432ecbcba5d86e9f4ac2bf0/plugins/modern-cpp)
is a complementary reference for migration examples and safety-oriented topic
coverage. Consult the standard and vendor sources linked in the topic guides
for language semantics and current implementation support.
