# C++23 Guidance

Use C++23 to simplify result handling, enum conversion, callable ownership,
and multidimensional or compile-time code when the active toolchain supports
the selected facility.

The examples below are representative, not exhaustive. The usable feature set
is determined by the effective standard, compiler, standard library, project
constraints, and local build evidence.

## Good default candidates

- Use `std::expected` for an agreed value-or-error interface when it clarifies
  calling code, and `std::to_underlying` for explicit enum-to-integer conversion
  that would otherwise repeat the underlying type or a cast convention.
- Use `std::move_only_function` for stored callbacks that own move-only state
  when an owning, type-erased callable is needed.
- Use explicit object parameters to share member implementation across
  receiver forms when that removes real overload duplication.
- Use `std::mdspan` when the toolchain and problem justify a borrowed
  multidimensional view with explicit extents and layout; keep storage
  ownership with the existing owner.
- Prefer direct result checks or short monadic composition with `optional` or
  `expected` according to which makes propagation and control flow clearer.

## Value-or-error results

Use `expected<T, E>` when failure belongs in the return contract. Choose a
stable error model that carries information callers can use; reserve
`optional<T>` for absence without an error. A factory returning `expected`
can expose recoverable construction failure while preserving the object's
invariant. Keep an established exception contract unless changing it is part
of the task. Neither the type nor `[[nodiscard]]` forces callers to inspect it.

For example, a parser can distinguish empty, malformed, and out-of-range input
without changing an output argument on failure:

```cpp
#include <charconv>
#include <expected>
#include <string_view>
#include <system_error>

enum class CountError { empty, syntax, range };

[[nodiscard]] constexpr std::expected<unsigned, CountError>
parse_count(const std::string_view text) noexcept {
    if (text.empty()) [[unlikely]] {
        return std::unexpected(CountError::empty);
    }
    unsigned count{};
    const auto [end, error] =
        std::from_chars(text.data(), text.data() + text.size(), count);
    if (error == std::errc::result_out_of_range) [[unlikely]] {
        return std::unexpected(CountError::range);
    }
    if (error != std::errc{} || end != text.data() + text.size()) [[unlikely]] {
        return std::unexpected(CountError::syntax);
    }
    return count;
}
```

Check the active state before dereferencing or reading `error()`. `value()`
uses an exception on failure and may not fit an exception-free caller.
Transporting a value or error can itself allocate or throw; `expected` does
not make arbitrary operations non-throwing or cheaper.

| Operation | Meaning |
| --- | --- |
| `and_then(f)` | On success, continue with a function returning another `expected` |
| `transform(f)` | On success, map the value and wrap the result |
| `or_else(f)` | On failure, run a recovery/error path returning an `expected` |
| `transform_error(f)` | On failure, map the error representation |

Check the value/error type constraints and value category passed to each
function. `optional` also has `and_then`, `transform`, and `or_else` for its
absence model. Prefer explicit branching when that makes propagation,
logging, or recovery easier to follow. Verify monadic support separately from
the presence of the base type.

## Output and receiver-dependent code

Use `std::print` and `std::println` for simple console output when library
support is confirmed and their destination and error behavior fit. `print`
writes formatted output; `println` additionally appends a newline. A newline
is not the explicit flush performed by `std::endl`. Preserve required stream
formatting, localization, encoding, and custom sink behavior. Standard
formatting and `{fmt}` overlap but are not interchangeable in every feature;
keep an existing dependency when required capabilities still use it.

Use explicit object parameters only when they materially clarify a member-like
API. They can consolidate const/non-const member logic, support a recursive
lambda, or replace CRTP used only to discover the derived receiver. Preserve
genuine inheritance relationships and overload contracts.
For accessors, first decide whether temporary receivers are allowed. Restrict
to lvalues when a borrowed result should not outlive a temporary; use
`std::forward_like<Self>` when a required cv/ref propagation is not provided
by the member access itself. Forwarding categories does not extend lifetime.

```cpp
class Sample {
    int value_{};
public:
    // A reference is available only from a named, live receiver.
    constexpr decltype(auto) value(this auto& self) noexcept {
        return (self.value_);
    }
};
```

The parentheses preserve the reference in `decltype(auto)`; const receivers
produce a const reference. Use `if consteval` when constant-evaluated and
runtime paths legitimately differ. Use expanded constexpr support to simplify
implementation or testing; do not duplicate paths solely to use it.

## Ranges and containers

Choose ownership and evaluation timing separately. When a returned pipeline
must remain lazy, it can own its source through `owning_view` or another
suitable owning range; its elements and callbacks may still borrow elsewhere.
Materialize with [`std::ranges::to`](https://eel.is/c++draft/range.utility.conv.to)
when an owned result container is needed immediately. With suitable input and
operations, the shape is:

```cpp
auto selected = records
    | std::views::filter(is_selected)
    | std::views::transform(project_value)
    | std::ranges::to<std::vector>();
```

This is an expression sketch: the source and callable lifetimes must remain
valid until materialization finishes. Element copies/moves and allocation are
part of the operation. `views::enumerate` adds indices, `views::zip` stops at the
shortest input, and `views::chunk(n)` groups consecutive elements with a positive
chunk size. Validate equal-length requirements explicitly when pairing data
with `zip`.

`std::flat_map` and `std::flat_set` keep sorted storage, with vectors as the
default backing containers. They can suit bulk-built, read-heavy data.
Single-element insertion and erasure are linear; iterators/references do not
have the stability of node-based maps. Review proxy reference behavior in
generic callers. Measure lookup, construction, mutation, and memory use for
the real workload instead of choosing by a fixed element-count threshold or
speedup.
See [flat_map](https://eel.is/c++draft/flat.map.overview).

For `mdspan`, choose extents, layout, and accessor to match existing storage.
It does not allocate or validate an arbitrary backing buffer. Use
multidimensional subscript syntax to simplify access only when the type's
model and compiler support are clear.

## Generators and library modules

After choosing coroutines, use `std::generator` for synchronous pull-based
production when that control flow is clearer than an iterator or simple loop.
It is not an async task or parallel execution facility. Inspect frame
allocation, borrowed parameters, yielded references, single-pass consumption,
and early destruction; creating a lazy generator does not necessarily execute
its body immediately.

`import std` requires a compiler, standard library, module artifact, and build
pipeline that work together. A C++23 flag alone is insufficient. Standard
library modules do not export preprocessor macros; keep the headers needed
for such macros. Verify dependency scanning, mixed header/module use, and
downstream consumers before an authorized migration. Retain headers where
they already meet the task without build changes.

## Support checks

Check the selected feature's semantics, feature-test macro, and vendor
implementation evidence. Choose supported facilities for the current code;
support and ecosystem readiness for `std::expected`, `std::mdspan`, `std::print`,
and newer ranges can lag the language mode, so its flag alone is not the
acceptance signal.
