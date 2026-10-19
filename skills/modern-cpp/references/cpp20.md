# C++20 Guidance

Use C++20 facilities to express borrowed data, generic operations, sequence
work, and thread lifetime directly.

The examples below are representative, not exhaustive. The usable feature set
is determined by the effective standard, compiler, standard library, project
constraints, and local build evidence.

## Good default candidates

- Replace pointer/count interfaces with non-owning `std::span` views when
  storage is contiguous, owned by the caller, and borrowing fits the contract.
- Constrain generic operations with concepts that match the expressions used
  and clarify intent and diagnostics. Use whole-range algorithms and
  projections for member-based lookup or ordering to remove repeated iterator
  or comparator mechanics.
- Use `std::erase` and `std::erase_if` for container cleanup and filtering, and
  `std::source_location` for diagnostics and tracing, including caller-site
  information that previously required macros.
- Prefer `std::jthread` for an owned, joinable worker whose lifetime should end
  with its scope; use `std::stop_token` when its work supports cooperative stop.

## Other useful library facilities

- `std::bit_cast` when representation-preserving conversion is intended
- Calendar, timezone, and chrono formatting facilities when the project already deals with time-heavy logic
- `std::ssize` when signed sizes improve correctness
- [`std::cmp_*` and `std::in_range`](https://eel.is/c++draft/utility.intcmp)
  for mixed-sign integer comparisons and representability checks before integer
  conversion
- [`std::midpoint`](https://eel.is/c++draft/numeric.ops.midpoint) for an
  overflow-free midpoint and [`std::lerp`](https://eel.is/c++draft/c.math.lerp)
  for interpolation with defined endpoint and monotonicity guarantees;
  preserve the required rounding and numeric semantics

## Concepts that match the call

Start with standard concepts such as `integral`, `floating_point`, `regular`,
`invocable`, `predicate`, and the range concepts. Use a local `requires` clause
for a one-off requirement; name a concept when the name expresses a useful
contract or a repeated constraint. Check the same constness, reference form,
return type, and exception requirement that the implementation uses.

For named templates, declare type parameters with `typename` and put the
constraints in a separate `requires` clause immediately after the template
parameter list, including simple constraints such as `requires std::integral<T>`.
Use named concepts within that clause. Reserve trailing clauses for syntax or
scope requirements, and keep matching declarations and definitions consistent;
see [template constraint style](style-principles.md#template-constraint-style).

For a callback invoked repeatedly, retain the by-value, lvalue-invocation rule
from [style-principles.md](style-principles.md):

```cpp
#include <concepts>
#include <functional>
#include <span>
#include <type_traits>

template <typename F>
    requires std::invocable<F&, const float&>
constexpr void visit_samples(const std::span<const float> samples, F callback)
    noexcept(std::is_nothrow_invocable_v<F&, const float&>) {
    for (const auto& sample : samples) {
        std::invoke(callback, sample);
    }
}
```

Constraints check expressions; semantic requirements such as a strict weak
ordering still need a correct implementation. Replacing SFINAE can affect
overload ordering, so compile representative accepted and rejected calls.
Check downstream diagnostics for supported template interfaces.

## Ranges, views, and spans

Range algorithms perform their operation when called. Views usually describe
work performed during traversal; they are not precomputed results. Use a named
algorithm or projection for a complete operation, and a short pipeline when
lazy selection or transformation is clearer that way. The
[standard sorting algorithms](https://eel.is/c++draft/sort) show range and
projection forms. Keep view lifetimes and lazy evaluation clear.

| Task | C++20 facility | Check |
| --- | --- | --- |
| Select / transform elements | `views::filter` / `transform` | Predicate and transform lifetime, repeated evaluation, side effects |
| Read a prefix / skip a prefix | `views::take` / `drop` | Requested count and underlying traversal cost |
| Split a sequence | `views::split` | Delimiter and source lifetime; subranges borrow storage |
| Flatten nested ranges | `views::join` | Inner-range lifetimes and resulting iterator category |
| Order or find by a member | Range algorithm with a projection | Ordering semantics and iterator invalidation |

C++23 adds `enumerate`, `zip`, `chunk`, and container materialization; C++26
adds `concat`. Use the appropriate version guide before adopting them.

For algorithms returning iterators into temporary ranges, respect
[`borrowed_range` and `dangling`](https://eel.is/c++draft/range.dangling).
Borrowed-range status does not extend the lifetime of underlying storage.

`std::span<T>` views mutable contiguous elements; `span<const T>` views them
read-only. `const span<T>` makes the view object const, not its elements.
Dynamic extent carries a runtime size; fixed extent expresses an interface
requirement. Verify construction preconditions, backing storage lifetime,
empty access, and index bounds. A span does not own storage or establish that
its pointer/count input is valid. Its C++20 indexing is unchecked; checked
access and implementation hardening are separate choices. See
[span](https://eel.is/c++draft/views.span) and
[safe-idioms.md](safe-idioms.md).

## Formatting and diagnostics

When supported and preferable to ad-hoc formatting, use `std::format` for owned
formatted text and `format_to` for a suitable output iterator. A raw output
pointer still needs sufficient capacity; `format_to_n` can bound output but
does not append a terminator. Keep format strings separate from untrusted data
and preserve encoding, locale, precision, and failure behavior. Runtime format
strings need the appropriate runtime formatting API and error handling.

For a domain type, reuse a standard formatter when its formatting options fit:

```cpp
#include <format>

struct RecordId { unsigned value; };

template <>
struct std::formatter<RecordId> : std::formatter<unsigned> {
    auto format(const RecordId id, std::format_context& context) const {
        return std::formatter<unsigned>::format(id.value, context);
    }
};
// std::format("record {:04}", RecordId{7}) produces "record 0007".
```

`std::print` is C++23. Existing `{fmt}` or stream-based code may provide needed
features or broader support; compare actual APIs and dependencies before
replacing it. Use `source_location::current()` as a default argument when a
diagnostic wrapper needs the caller's location, and preserve the logging
system's formatting, source-location, and allocation contract.

## Comparison and initialization

- For a type with a natural total or partial ordering, default `<=>` when
  base/member declaration order expresses the intended comparison.
  A defaulted spaceship can supply equality; a hand-written spaceship alone
  does not supply `operator==`. Preserve domain-specific equivalence, and
  account for partial ordering, including floating-point NaN.
- Use designated initializers in member declaration order when they clarify
  aggregate construction. Keep intended default member initializers; omitted
  members follow aggregate initialization rules, and designators cannot be
  arbitrarily reordered.
- Use `consteval` for required constant evaluation and `constexpr` for code
  that can also execute at runtime. A runtime call to a `constexpr` function
  still needs valid inputs and overflow-safe operations.

## Synchronization and coroutine boundaries

Use scoped locks for mutex ownership. Select `latch` for a one-time countdown,
`barrier` for repeated coordinated phases, and a condition variable for waiting
on a changing predicate. Atomics require an ordering argument justified by the
communication protocol; `volatile` is not thread synchronization.

For `jthread`, establish how a stop request reaches blocked work, how long join
can take, and which captured objects outlive the worker. Auto-join does not
prevent a deadlock caused by destruction while holding a needed lock. See
[safe-idioms.md](safe-idioms.md) for synchronization checks.

Use coroutines only when the project explicitly wants that execution model
and has mature runtime support. They can simplify resumable sequences,
asynchronous I/O, or state machines, but do not supply a scheduler,
parallelism, or a general async task type by themselves. Prefer an established
library's coroutine types; add an awaiter only for a required integration.
Review suspension-point lifetimes, captures, cancellation, frame allocation,
and destruction. C++23 `std::generator` serves synchronous lazy sequences.

## Support checks

Treat `std::format`, chrono formatting or timezone pieces, and heavier ranges
facilities as implementation-sensitive even in nominal C++20 projects. Check
feature-test macros, vendor support notes, and the active toolchain.
Keep modules opt-in unless the project explicitly wants them and the toolchain
story is mature enough; apply the same toolchain requirement to coroutines.
