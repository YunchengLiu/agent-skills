# Migration Patterns

Use these patterns to modernize code incrementally.

## Start local

- Prefer a small migration that solves the current problem over a broad style sweep.
- Update a representation and its active callers together. Preserve any
  transition required by released consumers or an explicit stability promise
  at that boundary.
- Separate semantic changes from representation changes when possible.

## Common migration shapes

The tables cover recurring choices, including facilities inherited from earlier
standards. Choose a row because its benefit fits the current code, then check
standard and library support and its condition. Keep a working representation
when the condition is not met.

### Storage and ownership

| Existing shape | Candidate and minimum standard | Benefit and condition |
| --- | --- | --- |
| Locally owned `new` / `delete` | Value member, container, or `std::unique_ptr`; `make_unique` in C++14 | Couple cleanup to lifetime; use dynamic allocation only when ownership or size requires it. |
| Owning `new[]` / `delete[]` | `std::vector`, or `make_unique<T[]>` in C++14 | Keep size and cleanup together; array smart pointers do not provide checked indexing or store a size for callers. |
| Several raw pointers jointly managing one lifetime | `std::shared_ptr` / `make_shared` (C++11) | Share ownership only when needed; assess control-block cost, cycles, and pointee synchronization. |
| Shared ownership used only for observation | `std::weak_ptr` (C++11) | Observe a shared lifetime without extending it; lock and check before access. |
| C allocation or an OS handle | Existing owner or RAII wrapper with matching release operation | Preserve allocator/deallocator pairing and foreign API contracts. |
| Fixed C array used as a value | `std::array` (C++11) | Preserve extent and value semantics; retain C arrays where an ABI or language facility requires them. |
| Element-wise `memcpy` | Assignment or `std::copy` | Respect object semantics, destination size, and overlap restrictions; keep byte copying for valid representation-level operations. |
| `memset` used to initialize objects | Member/value initialization or `std::fill` | Initialize values according to their types; zero bytes are not a general object invariant. |
| Pointer/count or contiguous iterator pair | `std::span` (C++20) | Couple range and extent when the caller owns the data; preserve access preconditions and make borrowing explicit rather than hiding unclear ownership. |
| Read-only text pointer | `std::string_view` (C++17) | Carry a length for borrowing bounded by the call. Use `std::string` when storage or lifetime must cross that boundary; materialize a terminated string when a C API requires one. |

### Types, state, and results

| Existing shape | Candidate and minimum standard | Benefit and condition |
| --- | --- | --- |
| Sentinel or success flag plus value | `std::optional` (C++17) | Express value-or-absence, separate from error-bearing paths; preserve a nullable pointer when it intentionally observes an existing object. |
| Status plus output parameter | `std::expected` (C++23) | Clarify value-or-error flow; match the error type and handling style to the agreed failure contract, preserve error details, and update callers. |
| Foreign error indicator or `errno` | Checked boundary translated to the established error model | Check the API's failure signal and capture relevant error state promptly; successful calls need not reset `errno`. |
| Construction with a recoverable failure | Factory returning the existing result type, optionally `expected` | Keep a valid object invariant; retain exception-based construction when that is the agreed contract. |
| C-style cast | Appropriate named cast | Expose the conversion category; `static_cast` alone does not prove representability. |
| Aliasing-based type punning | `std::bit_cast` (C++20), or valid byte copying | Require matching sizes, suitable trivially copyable types, and a valid destination representation. |
| Manually tagged `union` | `std::variant` (C++17) | Track the active alternative; preserve layout-sensitive or low-level union uses when required. |
| Untyped `void*` protocol | Template, closed `variant`, or open `std::any` (C++17) | Choose by the actual set of types, ownership, and erasure boundary; preserve required C interfaces. |
| Unscoped enum / null literal | `enum class` / `nullptr` (C++11) | Clarify scope and overload selection; inspect affected conversions and ABI. |
| Surprising implicit construction | `explicit` constructor or conversion | Preserve conversions that are intentional parts of the API. |
| Important ignored result | `[[nodiscard]]` (C++17) | Diagnose likely misuse; the attribute does not enforce handling. |
| Macro constant | Typed `constexpr` value (C++11) | Express scope and type; retain macros for preprocessing decisions. |
| Opaque deleted overload | Reason-bearing deletion (C++26) | Improve caller diagnostics when the explicitly selected toolchain supports it. |

### Formatting, generic code, and sequence work

| Existing shape | Candidate and minimum standard | Benefit and condition |
| --- | --- | --- |
| `sprintf` / `snprintf` string assembly | `std::format` (C++20) | Return owned formatted text; preserve encoding, locale, truncation, and failure behavior. |
| `printf` or stream insertion chain | `std::print` / `println` (C++23) | Simplify plain formatted output; retain required stream state, custom destinations, and flushing. |
| SFINAE / `enable_if` | Concepts and `requires` (C++20) | State the actual supported expressions; check overload selection and diagnostics. |
| CRTP or cv/ref overload duplication | Explicit object parameter (C++23) | Share receiver-dependent implementation when base-type relationships are not otherwise required. |
| Hand-written comparison family | Defaulted `<=>` (C++20) | Use only when member-wise comparison is the intended equality and ordering. |
| Tuple machinery for pack selection | Pack indexing (C++26) | Select an existing pack element directly with a valid index. |
| Owning callable wrapper for a bounded call | Template by value, or `function_ref` (C++26) when erasure is warranted | Preserve the repeated-callback default in [style-principles.md](style-principles.md); borrowing must outlive invocation. |
| Copyable erasure blocking move-only captures | `std::move_only_function` (C++23) | Store move-only callable ownership; account for empty state and allocation. |
| Loop implementing a named sequence operation | Standard algorithm, range overload (C++20), or short view pipeline | Expose intent; retain a direct loop for clearer domain actions. |
| Erasure during invalidating traversal | `std::erase` / `erase_if` (C++20) | Use the container's supported erasure path; audit observers invalidated by the operation. |
| Repeated member comparator | Range algorithm with projection (C++20) | State the member key directly while preserving ordering. |
| Tree map/set dominated by reads | `std::flat_map` / `flat_set` (C++23) | Consider locality and bulk construction; check mutation costs and observer invalidation, then measure. |
| Third-party fixed-capacity vector | `std::inplace_vector` (C++26) | Standardize in-object storage only if capacity, insertion failures, and toolchain fit. |
| Owning indirection needing deep-copy value semantics | `std::indirect` / `polymorphic` (C++26) | Preserve value copying and, for polymorphic values, dynamic type; assess allocation and ABI. |
| Field/enum metadata macros or generated boilerplate | Reflection (C++26) | Evaluate a bounded metadata task; retain schema policy, protocol compatibility, and framework integration. |

### Concurrency and assertions

| Existing shape | Candidate and minimum standard | Benefit and condition |
| --- | --- | --- |
| Manual mutex acquisition/release | `lock_guard` / `unique_lock` (C++11), `scoped_lock` (C++17) | Tie release to scope; use `unique_lock` for waiting or controlled unlock, and inspect wider lock ordering. |
| Owned thread with manual join | `std::jthread` / `stop_token` (C++20) | Establish stop and join semantics; verify worker captures, blocking operations, and destruction order. |
| `volatile` used for communication | Mutex or `std::atomic` (C++11) | Establish synchronization; preserve legitimate platform-defined volatile I/O. |
| Hand-coded phase coordination | `std::latch` / `barrier` (C++20) | Use for one-shot counting or repeated phases; retain condition variables for changing predicates. |
| Compare/exchange loop computing an extremum | Atomic `fetch_min` / `fetch_max` (C++26) | Match the operation, return-value use, and memory ordering. |
| Internal assertions | Existing assertions, or contracts for an explicit C++26 task | Preserve enforcement expectations; always validate untrusted input through ordinary control flow. |

Read [safe-idioms.md](safe-idioms.md) for lifetime and correctness details,
the relevant version guide for usage examples, and
[compiler-hardening.md](compiler-hardening.md) for targeted diagnostic evidence.

### Stable consumer boundaries

- Check supported downstream language, toolchain, and ABI requirements before
  exposing a newer type in a committed interface. Inspect or ask when actual
  consumer requirements are unclear.
- When a transition is required, keep its scope and removal condition tied to
  those consumers. An internal declaration alone does not require an adapter.

### Narrow utility upgrades

- Prefer `enum class`, structured bindings, `if` with initializer, `[[nodiscard]]`, and similar local clarity improvements when they reduce ambiguity without changing architecture.

## Escalate instead of patching

When local cleanup repeatedly points to the same design problem, stop proposing isolated syntax upgrades and explain the better abstraction or boundary.

## Verify the migration

Compile the affected callers at the supported baseline. Exercise the preserved
success and failure behavior, empty states, ownership transfer, and invalidation
boundaries relevant to the change. For comparisons, overloads, formatting, and
container replacements, check observable semantics rather than assuming a
similar spelling means equivalent behavior. Use focused sanitizer runs or
measurements when the claimed benefit depends on lifetime or cost.
