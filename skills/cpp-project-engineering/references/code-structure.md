# C++ Code Structure And File Ownership

## Scope

Apply these rules to new or touched C++ source and headers. Converge the touched
area; do not launch a repository-wide style migration merely because this
reference is active.

## Hard Checklist

Before editing and before finalizing, check the touched area for:

1. the changed behavior, owning path, required information, consumers, and
   established guarantees;
2. current evidence for every new abstraction, helper, state, check, fallback,
   representation, or wrapper;
3. responsibility placed at the earliest boundary that owns the needed
   information, without duplicate validation or analysis downstream;
4. natural context-sized naming;
5. class or struct layout;
6. semantic include choice and project include-path style;
7. helper placement: member, module-private file, or anonymous namespace;
8. the minimum necessary namespace and qualification depth;
9. supported-domain, precondition, and failure ownership;
10. declaration, implementation, test, and build-registration landing points;
11. affected callers, serialized forms, installed surfaces, and target wiring;
12. no unnecessary copies, repeated computation, dead intermediates, or
    speculative performance structure.

## Design Before Local Mechanics

Start from the owning behavior path and current contract. Local helpers,
wrappers, aliases, private types, and files come after the path they serve is
known. Search the existing execution path, repository utilities, standard
library, and configured dependencies before creating another mechanism. Carry
that path and its established guarantees into local choices and review. Judge
each added part by the responsibility still missing there, while preserving
all applicable layout, ownership, and mechanical rules.

Use the direct language mechanism. Do not contort local code to solve a separate
header-exposure, compile-coupling, or imagined ABI concern.
Hiding dependencies, reducing compile coupling, and shrinking header exposure
are deliberate behavior-preserving refactors under explicit direction. Resolve
a real concern at the boundary or interface; when local code needs contortion,
inspect that boundary first.

Use `typename` for ordinary template type parameters, including packs.
Keep this spelling consistent in declarations, definitions, and examples.
In C++20 and later, put constraints for named templates in one `requires`
clause on a separate line immediately after the template parameter list, even
for a single named concept. Reference and combine concepts in that clause;
use this explicit form by default instead of `template <Concept T>` or
abbreviated `Concept auto` parameters. Use a trailing clause only when syntax
or scope requires it, such as a constrained non-template member of a class
template or a constraint needing function parameter names. Keep constraint
form, ordering, and placement consistent between matching declarations and
definitions; preserve constrained expressions, cv/ref forms, and overload
behavior when changing an existing spelling.

For an internal callback, consumer, or visitor, default to a template parameter
taken by value and constrain it when the effective standard supports
constraints. Invoke it as a stable lvalue; do not use a forwarding reference
for repeated callbacks. Callers with heavy state pass
`std::ref`.

Do not replace that clear template with hand-written `void*` and a
function-pointer trampoline merely to move a body into a `.cpp`; doing so keeps
the template entry, adds indirection, and gives up inlining without becoming
simpler. Depart from the template default only for a measured instantiation
cost or a real ABI or plugin boundary. Then use an established mechanism such
as `std::function` or an abstract interface and report the deviation. An
external C ABI callback is interop, not this anti-pattern.

Keep the consumer inlineable through the constrained template:

```cpp
template <typename Consumer>
    requires std::invocable<Consumer&, double, std::span<ScanRun const>>
void scan(Schedule const& schedule, Consumer consumer) const;
```

This still needs a template entry while adding manual erasure solely to move
the loop into a `.cpp`:

```cpp
template <typename Consumer>
    requires std::invocable<Consumer&, double, std::span<ScanRun const>>
void scan(Schedule const& schedule, Consumer consumer) const {
    auto ref = std::ref(consumer);
    scanImpl(schedule, std::addressof(ref), &trampoline<decltype(ref)>);
}
void scanImpl(Schedule const&, void*, void (*)(void*, double, std::span<ScanRun const>)) const;
```

Keep behavior and state together when they share ownership and reasons to
change. Split only at a real responsibility, dependency, lifetime, or consumer
boundary. A new conceptual name or hypothetical future consumer is not evidence
for another entity.

## Naming

Name a symbol within its limiting scope. Read its name together with its type,
namespace, class, declaration, nearby comment, and local code.

- Do not repeat the namespace, class, or established domain context in a name.
- Keep role, unit, ownership, or behavior words when they supply a necessary
  distinction within that scope; omit words whose meaning it already establishes.
- Do not encode a full definition, every precondition, or implementation detail
  into one identifier so it can stand alone outside its context.
- Prefer a short verb or noun for a function, such as `from`, `as`, `parse`, or
  `instant`; overloads, parameter types, the class, and the namespace carry the
  remaining distinctions. Choose the word from the operation's function, what it
  shares with related APIs, and its context, and keep related names parallel.
- Established local and domain abbreviations are valid when unambiguous. Expand
  them when several similar values are in scope or the role or unit would be
  hidden.
  Examples include `param`, `def`, and `iter` in a clear local scope.
- Derive production names from repository and domain roles, not task labels,
  phase names, explanatory diagrams, or pseudocode names.
- Public names must remain clear to callers who do not know private
  implementation structure.
- Keep acronyms uppercase, such as `ID`, `ISO`, and `UTF8`. An acronym that
  begins a camelBack or snake_case name is lowercase, such as `isoWeek` and
  `user_id`.

Prefer:

```cpp
namespace project::filling {
class EdgeTable;
}

struct ScanEdge {
    double y_min = 0.0;
    double inv_k = 0.0; ///< dx/dy
};
```

Avoid names such as `FillingEdgeTable` inside `project::filling` or
`scan_edge_y_min` inside `ScanEdge` when the added words repeat the scope.

Where the owner already establishes the directory and relative-path context,
use `findLatestRefs()` rather than
`findLatestRelativeReferencesInsideCurrentDirectory()`.
Prefer `DateTime::from(file_time)` and `value.asZone(zone)` over
`DateTime::fromFileTimeType(file_time)` and `value.withZoneSameInstant(zone)`;
pair a free `timeT(time_point)` with a member `value.timeT()` rather than giving
them unrelated names.
Use `inv_k` with the `dx/dy` comment above instead of encoding its definition
as `step_value_of_x_when_y_increases_one` or
`inverse_value_of_the_scan_edge_slope`.

## Class And Struct Layout

Use logical dependency order rather than mechanically grouping access blocks:

1. nested declarations needed by later members or interfaces;
2. data members, grouped by semantic role;
3. public functions;
4. protected functions;
5. private helper functions.

Declare special members first among the public functions, in the order
constructors, destructor, copy, move. Declare a destructor only together with
all copy and move operations, even when each is `= default`; otherwise declare
constructors only.

Multiple access blocks are allowed when a public nested declaration must appear
before private data. A nested enum, alias, or helper type that is itself public
API may also appear there even when later declarations do not depend on it. Do
not hoist unrelated local declarations merely to group types, add an access
label only to restate the type's default access, or append a new field away from
its semantic peers.

```cpp
class Widget {
    class Impl;

    std::unique_ptr<Impl> impl_;
    int                   retry_count_ = 0;

public:
    explicit Widget(Config const& config);
    [[nodiscard]] bool ready() const noexcept;

protected:
    void onStateChanged();

private:
    void rebuildCache();
};
```

When the nested declaration is public, multiple access blocks preserve the
same dependency order:

```cpp
class Widget {
public:
    class Impl;

private:
    std::unique_ptr<Impl> impl_;

public:
    explicit Widget(Config const& config);
};
```

A struct needs no access label to restate its default:

```cpp
struct LayoutOptions {
    int width  = 0;
    int height = 0;

    [[nodiscard]] bool valid() const noexcept;
};
```

Keep short members inline. Move a definition out of a header when it is
genuinely complex or would expose heavy dependencies, not by reflex. A
header-defined template may call a non-template private member defined in the
`.cpp` when only the declaration must remain visible.

## Includes

The formatter owns sorting, grouping, spacing, and indentation. Manual review
owns dependency choice and path meaning.

- In project headers, use the project's root-qualified include path, including
  for same-module project headers.
- In `.cpp`, use the shortest unambiguous path from the source directory for a
  sibling or child implementation header.
- In `.cpp`, use the root-qualified path for a cross-module header.
- Include what the file semantically depends on; do not depend on accidental
  transitive includes.
- Follow the project's established ordering and formatter rather than manually
  creating a competing include layout.

The project-specific root prefix belongs in repository instructions, not in
this reference.

For a `.cpp`, local implementation includes and cross-module includes have
different roots:

```cpp
#include "component.h"
#include <string>
#include <third_party/library.hpp>
#include "detail/local_helper.h"
#include "project/config/error.h"
```

In a project header, both same-module and cross-module includes are rooted:

```cpp
#pragma once
#include "project/base/value.h"
#include "project/module/options.h"
```

## Helper Placement

First decide whether a helper should exist. It earns its place when it removes
real complexity, names a meaningful algorithm step or data seam, centralizes an
existing stable rule, matches local decomposition, or needs direct testing.
Delete a helper that only renames
an expression, forwards without simplifying the caller, or creates a second
place to understand the same behavior.

Keep a short, self-evident, single-use block inline. Give a substantial
algorithm phase its own free function even at one call site when that keeps the
data seam visible. Use an in-body lambda only for a short local closure.

Place an earned helper by the first matching rule:

1. It needs member state, virtual dispatch, or a by-name call from a
   header-defined template: private member.
2. The same stable semantics already exist in sibling implementation files:
   centralize in the nearest module-private header and paired `.cpp` where
   applicable, and update all current callers in the same change.
3. It carries substantial independent behavior that needs direct tests:
   module-private header and paired `.cpp` where applicable.
4. Otherwise: anonymous namespace in the owning `.cpp`.

A pure value transformation does not become a member merely because member data
is passed to it. Similar-looking logic with different owners or reasons to
change is not duplication. Every declaration added to a header is a demand on
the reader and must earn that wider surface.

## Namespaces And Qualification

- Add a namespace layer only for a real module boundary or name collision.
  Remove each layer in thought and retain it only when lookup or ownership would
  become less clear.
- When one `.cpp` implements one module namespace, place its anonymous namespace
  inside that module namespace. Preserve a top-level anonymous namespace when a
  file genuinely serves several independent namespace areas or moving it would
  be unrelated churn.
- Do not introduce or change a root-namespace lift during routine edits. Lift a
  type only as an intentional public API decision; never lift `detail` or other
  internal-only symbols.
- From the current lexical scope, remove leading namespace components one at a
  time and keep the shortest unambiguous spelling.
- Headers use qualified names instead of namespace-scope `using` declarations
  or `using namespace`. Use a very small function-local alias when a local
  simplification is genuinely needed.
- Use a short namespace alias when repeated qualification becomes material
  local noise or it resolves a real ambiguity; do not hide ownership.
- Every project symbol emitted by a macro replacement list uses expansion-safe
  root qualification so invocation-scope declarations, aliases, and
  argument-dependent lookup cannot change the selected symbol.

When every definition in a file belongs to one namespace layer, use compact
namespace syntax. When both a parent and child need definitions, use one nested
block that preserves their hierarchy instead of bouncing between repeated
namespace blocks. Do not reorganize existing blocks solely for style.

In a production `.cpp`, keep the shortest clear qualification at one or a few
uses. Add a specific `using` declaration at the top of the owning namespace
only when repeated qualification is material noise or the declaration resolves
an ambiguity; apply the same threshold to a namespace alias. Do not add a
file-scope `using namespace`. In tests, apply the same local cost test while
preserving established style outside the touched lines.

For one module, nest file-local helpers inside its namespace:

```cpp
namespace project::module {
    namespace {
        [[nodiscard]] bool validName(std::string_view name) noexcept {
            return !name.empty();
        }
    }
}
```

A top-level anonymous namespace remains appropriate when the file serves
independent namespace areas, or preserving its existing local structure is
clearly better:

```cpp
namespace {
    [[nodiscard]] bool validName(std::string_view name) noexcept {
        return !name.empty();
    }
}
namespace project::module {
}
```

An explicitly designed public root alias can take this form:

```cpp
namespace project::transport {
    class ChannelPolicy;
}
namespace project {
    using Channel = transport::SyncIO<transport::ChannelPolicy>;
}
```

Internal symbols must not be lifted this way:

```cpp
namespace project {
    using detail::InternalCacheNode;
    using detail::UnsafeCodecState;
}
```

Use a compact namespace when it contains every definition:

```cpp
namespace project::module {
    void normalize();
}
```

Use the actual hierarchy when both levels need definitions:

```cpp
namespace project {
    void normalize();
    namespace module {
        void encode();
    }
}
```

Do not bounce between blocks for that case:

```cpp
namespace project::module {
    void normalize();
}
namespace project {
    void topLevel();
}
namespace project::module {
    void encode();
}
```

The enclosing namespace already supplies the project context:

```cpp
namespace project::slicer {
    mesh::FacetList clip(mesh::Facet const& facet, contour::Loop const& loop);
}
```

Inside `project::slicer`, `mesh::Facet` names the sibling module's type. If
`Facet` belongs directly to `project`, use `Facet` without a prefix.

These repeated qualifiers add nothing to lookup:

```cpp
namespace project::slicer {
    project::mesh::FacetList clip(project::mesh::Facet const& facet, project::contour::Loop const& loop);
}
```

## Interface And Pipeline Ownership

For a public wrapper with a private implementation, explicitly decide which
layer owns lifecycle checks, caller-decidable validation, normalization,
serialization, common logging, state, and dependency-specific behavior.

- The public layer may own behavior common to every implementation when the
  contract requires it.
- Implementation-specific state machines, device or backend limits, timeouts,
  error mapping, and native source-of-truth reads stay with the implementation
  that has the required information.
- Metadata or capability queries do not receive readiness checks, locks, or
  caching mechanically; each addition needs a contract or concurrency reason.
- A native backend remains the source of truth when it provides reliable
  readback. Add cache state only to fill a real API gap, update it only after
  successful operations, and define invalidation.
- Do not repeat validation, locking, logging, or recovery in both wrapper and
  implementation.

For staged compilers, data pipelines, or runtimes, name the owner of each
semantic property. A later stage must not repair work an earlier stage owns, and
an optional analysis or optimization must not become an undeclared execution
gate. Before adding an opcode, pass, representation, registry, or index, check
whether the existing composition represents the required behavior.

Project-specific safety promises, state transitions, language semantics, and
backend behavior remain project contracts; do not infer them from this general
ownership model.

## Supported Domain And Failures

Express an API's supported domain through its shape, names, documentation, and
preconditions. Design for that current domain, not hypothetical callers, scale,
or execution contexts.

Classify a condition before adding handling:

| Condition | Handling |
| --- | --- |
| Failure possible during valid supported use | Handle at the owning boundary through the API's error form |
| Violation of an explicit, already-established programmer precondition | Assert at the appropriate boundary; release behavior outside the precondition is not promised |
| Property already guaranteed by an earlier boundary | Rely on it; an assertion may document it, but do not add duplicate validation or fallback |
| State impossible after validation | Assert or use the project's established unreachable mechanism |

Do not replace a documented runtime rejection with a new precondition merely to
avoid handling it. Recovery, rollback, restoration, and strong exception
guarantees are contract features, not automatic implementation upgrades. A
rejection is complete when it reports through the API's promised error form;
post-failure state is unspecified unless the contract states otherwise or an
external side effect requires cleanup. Use [error-handling.md](error-handling.md)
for error-form and diagnostic rules.

Prefer an early return or early error for a rejectable or trivial case so the
main path stays direct. Skip that shape only when the condition is genuinely
unclear or the state needed to decide it is too complex to establish early.

## Files And Landing Plan

A construct is an independent class, function family, or module boundary with
its own responsibility and reason to change. A separately named design step,
local helper, member, or cohesive implementation detail is not a construct
merely because it has a name. Adding members or functions to an existing owner
needs no separate landing plan.

Before creating a construct, or changing which file independently owns it,
identify four landing points:

1. declaration;
2. implementation;
3. tests;
4. build registration.

Mark a point not applicable only with a concrete reason. Derive the landing plan
as if the changed construct were built fresh and converge that construct in the
same change. This required physical reorganization is part of changing its
ownership; it is not permission to reorganize neighboring constructs.
Report pre-existing misplacements discovered nearby; change them only when the
current task already alters that construct's status.

Then apply these rules:

- Every out-of-line definition of a header declaration lives in the
  same-directory `.cpp` with the same basename as that header.
- A library `.cpp` implements its own header and file-local helpers. Program
  entry points, example sources, and test sources may stand alone without a
  paired header. Any other exception requires a stronger project rule.
- A header-only construct is legitimate when it has no out-of-line
  implementation; do not create an empty paired `.cpp`.
- One file owns one coherent boundary or a tightly related small family. Split
  when a second real responsibility or dependency boundary appears, not when a
  line-count threshold is crossed.
- Shared module-private behavior lives at the nearest common consumer boundary,
  commonly under the project's established private/detail area. A flat module
  normally uses `<module>/detail/`; a functionally divided module keeps shared
  details there and function-owned details under
  `<module>/<function>/detail/`, not `<module>/detail/<function>/`. Do not
  create a directory level or namespace merely because the source tree permits
  it. Both directory forms normally use the module's `detail` namespace.
  Do not add `detail` structure to tests, benchmarks, or examples.
- Tests mirror the project module and target ownership rather than inventing a
  parallel taxonomy.
- Rename an existing file only when its construct ownership changes; do not
  rename it merely to normalize naming during an unrelated behavior change.
- Module-facing declarations stay outside `detail`; installation status does
  not determine their source placement. Implementation support inseparable
  from one owning header may use that header's local `detail` namespace.
- Every new source and header enters the owning target according to the
  project's CMake and header-exposure rules.
- When ownership changes, move existing declarations, definitions, tests,
  callers, and registration coherently; do not leave an accidental second home.

## Cost And Clarity

Prefer existing utilities, the standard library, and configured dependencies.
A custom primitive needs a current performance, precision, or domain reason.
When lifetime is established, prefer a view or span over an unnecessary copy.
Avoid repeated computation and dead intermediates. These are ordinary craft
and need no benchmark. Add optimization, concurrency, caching, or complex
allocation only for a current requirement. Do not trade readable ownership and control flow for a
micro-optimization without measurement or a stated constraint. Performance
optimization and concurrency need evidence from the task's actual workload.

## Tool-Covered Items

Use the configured tools for their mechanical baselines; these need only brief
reminders in style documentation:

- `clang-format`: indentation, line width, braces, pointer/reference alignment,
  spacing, include sorting, and namespace end comments;
- `clang-tidy`: naming case/prefix/suffix, trailing `_` on private or protected
  members, `explicit`, namespace comments, and readability/correctness checks;
- `.editorconfig`: newline, encoding, and trailing whitespace.
