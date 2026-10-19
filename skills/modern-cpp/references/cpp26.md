# C++26 Guidance

Use this reference when the user explicitly targets C++26 or asks about a
named C++26 facility. Select by a concrete benefit and verify that feature's
compiler and library support.

For version attribution, use a versioned standard text. The
[N5050 draft](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/n5050.pdf)
is the C++26 final draft identified by the
[editors' report](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/n5051.html).
The linked online working-draft sections are convenient navigation, but can
later contain changes for another standard. Match the wording and feature-test
revision to the implementation being used.

## Useful candidates

- **Pack indexing, `Ts...[I]`:** select a parameter-pack element with a valid
  constant index in range when recursive helpers or tuple indirection serve
  only that purpose.
  See [pack indexing](https://eel.is/c++draft/expr.prim.pack.index).
- **`std::inplace_vector<T, N>`:** use a variable-size contiguous container when
  fixed capacity is part of the problem. Elements are stored inside the object,
  wherever it lives. Match insertion to the required full-capacity behavior:
  ordinary insertion can throw when full, `try_` insertion reports capacity
  failure, and unchecked insertion requires available space. Element
  construction can still throw. See the
  [container overview](https://eel.is/c++draft/inplace.vector.overview).
- **`std::function_ref`:** use a non-owning callable wrapper when bounded
  borrowing and type erasure are needed. The callable must survive every
  invocation: a temporary can be valid for an immediate synchronous call,
  while storing the wrapper for later generally needs a longer-lived owner.
  Retain the internal repeated-callback template default when erasure has no
  concrete benefit. See the
  [callable wrapper](https://eel.is/c++draft/func.wrap.ref).

These are starting points, not a complete feature list. For another named
facility, consult its standard wording and implementation evidence.

## Reflection and generated structure

Use static reflection for a concrete metadata operation: enumerator names,
member traversal, diagnostic output, or a deliberately defined mapping.
`^^T` produces reflection information; `std::meta::info` carries it, and a
splice such as `[:reflection:]` refers to the reflected construct in code.
Expansion statements (`template for`) can specialize work for each reflected
element. Check the selected queries and their required arguments instead of
copying syntax from an older proposal.

Member queries such as `nonstatic_data_members_of` take an access context.
Choose `access_context::current()` or `unprivileged()` according to the intended
boundary; use `unchecked()` only when deliberately bypassing access checks
is part of the design. A member reflection can subsequently permit access
through a splice, so reflection is not automatically a privacy boundary.
See [reflection access contexts](https://eel.is/c++draft/meta.reflection.access.context)
and [splice expressions](https://eel.is/c++draft/expr.prim.splice).

For serialization, preserve field selection, stable external names, versioning,
validation, supported types, and wire-format compatibility. Member discovery
does not establish those policies. Reflection can remove repetitive metadata
work; it does not automatically replace a protocol schema, framework-generated
behavior, or a tool such as a UI meta-object compiler. Compare a bounded use
with explicit code or an existing generator, including compile time,
diagnostics, debugging, and downstream support.

## Contracts and enforcement

Use `pre`, `post`, and `contract_assert` to express programming contracts when
the project has chosen their enforcement and failure policy. Preconditions
describe caller obligations, postconditions describe completion guarantees,
and an assertion expresses a condition at a point in the body. Keep the
expressions free of required side effects.

| Evaluation semantic | Consequence |
| --- | --- |
| `ignore` | The check is not evaluated |
| `observe` | A violation is reported through the handler; execution may continue |
| `enforce` | A violation invokes the handler and terminates if the handler returns |
| `quick_enforce` | A violation terminates without invoking the handler |

The implementation defines how a semantic is selected; inspect the actual
build and handler. A declaration alone does not establish enforcement. Retain
ordinary input validation and recoverable error paths regardless of contract
settings. Review declaration consistency and the standard's restrictions on
the function form before adopting contracts at an API boundary. Replacing
`assert` requires preserving the intended release/debug behavior, not just
changing syntax. See [contract assertions](https://eel.is/c++draft/basic.contract).

## Initialization and library hardening

The distinction between erroneous and indeterminate values improves treatment
of some uninitialized automatic storage. It does not make uninitialized reads
correct, choose a meaningful default value, or remove all undefined behavior.
Dynamic storage and `[[indeterminate]]` cases still need their specific rules.
Initialize objects to valid states; use the narrow propagation rules for
uninitialized-friendly byte types only when implementing low-level storage
operations. See [indeterminate and erroneous values](https://eel.is/c++draft/basic.indet).

Hardened library preconditions cover selected operations, not all C++ memory
access. In the C++26 wording, a hardened implementation checks such conditions
with terminating semantics; violating them in a non-hardened implementation
remains undefined behavior. This is distinct from recoverable checked access
such as `span::at`, which reports an out-of-range index through an exception.
Verify whether the selected library, build mode, and operation provide the
intended check. Neither mechanism fixes dangling storage or data races.
See [library specifications](https://eel.is/c++draft/structure.specifications)
and [compiler-hardening.md](compiler-hardening.md).

## Execution model

Evaluate the senders/receivers facilities in `std::execution` only for a
requested asynchronous or parallel composition problem. Establish value,
error, and stopped completion paths; scheduler affinity; operation-state
lifetime; cancellation; and shutdown before selecting an implementation.
Keep a simpler synchronous or established executor design when it already
meets the requirement.

The C++26 specification includes scheduler facilities such as `run_loop`,
`inline_scheduler`, and `parallel_scheduler`; availability of the required
backend and operations still depends on the library. Test the needed facility
and integration, not just the presence of `<execution>`, which also contains
older execution-policy facilities. See
[execution control](https://eel.is/c++draft/exec) and
[parallel scheduler](https://eel.is/c++draft/exec.par.scheduler).

## Other storage and smaller facilities

| Facility | Useful task | Adoption condition |
| --- | --- | --- |
| `std::indirect<T>` | Owning indirection with value copying | Copies copy the object rather than share identity; account for allocation, moved-from state, and the required copy semantics. |
| `std::polymorphic<T>` | Value semantics for an owned object of a derived type | Preserve dynamic type during copying; check allocation and the hierarchy's behavioral contract. |
| `std::hive` | Frequent insertion/erasure with stable surviving elements | Check operation-specific invalidation, including the past-the-end iterator and storage-reorganizing operations; it is not a contiguous random-access container. |
| `#embed` | Compile-time inclusion of a binary resource | Keep resource paths, size, encoding interpretation, build dependencies, and reproducibility explicit. |
| Name-independent `_` declarations | Unused bindings or a scoped RAII object | Use the supported declaration forms; `_` is not a general pattern-matching wildcard. Keep a meaningful name when needed for explanation or access. |
| `= delete("reason")` | A useful diagnostic for a forbidden overload | Explain the actual caller mistake; verify compiler support. |
| Saturating arithmetic | An operation whose result should clamp | Use `saturating_add/sub/mul/div/cast` from the C++26 wording; verify the implemented revision. Clamping changes semantics, and division still requires a valid divisor. |
| Atomic `fetch_min` / `fetch_max` | Atomic extremum update | Preserve old-value return use, memory ordering, and lock-free requirements. |
| `views::concat` | One traversal over successive ranges | Check compatible element/reference types and all underlying lifetimes. |
| `<simd>` | Data-parallel operations on batches of values | Verify target support, layout, alignment, and measured results; the interface alone does not promise particular machine instructions. |
| `<linalg>` | Basic linear algebra over multidimensional views | Match extents, layout, aliasing, arithmetic, and execution semantics; verify an optimized backend rather than assuming one. |

For saturation spelling changes and `inplace_vector` insertion return types,
consult the [C++26 motion record](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/n5047.html).
Use a small compile probe for the exact API; an implementation may expose an
earlier proposal revision. Keep the chosen capacity and failure policy explicit
when migrating from a third-party fixed-capacity container. A standard
replacement does not by itself justify removing a dependency used elsewhere.

## Support checks

- Verify compiler and standard-library support for the selected facility;
  partial language-mode support is insufficient.
- Keep adoption local to the useful feature. State the required compiler and
  library evidence before using it in shared or long-lived code.
- When the active toolchain lacks the facility, identify a supported
  alternative or the toolchain decision needed. Add a code fallback only when
  an actual supported toolchain or consumer requires both paths.
- Check consumer requirements for a supported API or ABI boundary. C++26
  remains an explicit choice, not the default baseline for general C++ work.
