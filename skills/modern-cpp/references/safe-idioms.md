# Safe C++ Idioms

Use this reference when choosing representations or operations to address a
concrete ownership, lifetime, state, numeric, or concurrency requirement.
Establish the effective standard and runtime constraints first. Apply the
[style defaults](style-principles.md), including `const`, truthful specifiers,
and the existing callback rules, to every example adapted into project code.

## Give each resource an owner

- Prefer direct values and containers for owned data. Introduce dynamic
  allocation when lifetime, size, polymorphism, or another actual requirement
  calls for it; smart pointers are not a reason to allocate.
- Use `std::unique_ptr` for exclusive dynamic ownership and `std::make_unique`
  for ordinary construction. Pass references or observer pointers when a
  function only uses the object; pass ownership types when it participates in
  the ownership contract.
- Use `std::shared_ptr` when independent owners must extend an object's
  lifetime. Account for allocation and reference-counting costs. Construct
  related owners from the existing control block, never independently from
  the same owning raw pointer.
- Use `std::weak_ptr` for observation of shared ownership and to break owning
  cycles. Obtain and retain a strong owner with `lock()` before use; a prior
  `expired()` check cannot reserve the object's lifetime.
- Shared ownership does not synchronize the pointee. Separate `shared_ptr`
  instances can share ownership concurrently, but conflicting access to one
  instance needs synchronization or supported `std::atomic<std::shared_ptr<T>>`
  in C++20. Protect mutable pointee state separately. See
  [shared ownership semantics](https://eel.is/c++draft/util.smartptr.shared).
- Bind foreign resources to their matching release operation immediately.
  Use a custom deleter for pointer-shaped resources; use an existing scoped
  handle type for integer or platform handles. Respect invalid-handle values,
  allocator pairing, release-thread requirements, and ownership transfers.

C++17: a C file resource can retain its own cleanup operation without a new
general-purpose resource framework. A null result still means opening failed.

```cpp
#include <cstdio>
#include <memory>

struct FileCloser {
    void operator()(std::FILE* const file) const noexcept {
        std::fclose(file);
    }
};

[[nodiscard]] auto openInput(const char* const path) noexcept {
    return std::unique_ptr<std::FILE, FileCloser>{std::fopen(path, "rb")};
}
```

Here `path` must be a valid null-terminated path. Destruction releases the
resource but cannot report close failure; a contract requiring flush or close
errors needs an explicit checked operation before final cleanup.

Prefer the Rule of Zero: members that own their resources supply correct
cleanup and copy/move behavior. If a type must customize a special member,
consider all five together. Define or delete copying according to ownership,
preserve invariants in moved-from objects, and mark moves `noexcept` when they
cannot throw. Defaulted moves are insufficient when related members need
coordinated updates. Avoid throwing destructors.

## Borrow only while storage remains valid

- Use `T&` for required observation and `T*` for nullable observation. A null
  pointer can correctly mean "no matching existing object"; replacing it with
  `std::optional<T>` changes the result to an owned value. An established
  `optional<reference_wrapper<T>>` convention can express optional borrowing.
- Use C++17 `std::string_view` for borrowed text and C++20 `std::span<T>` for
  borrowed contiguous elements. Use `span<const T>` for read-only elements;
  a `const span<T>` only makes the view object itself const.
- A view carries size, not ownership or general bounds enforcement. In C++20
  and C++23, `span::operator[]` requires a valid index; check untrusted indices
  and subranges before access. A pointer/count constructor still requires
  valid storage for that count. See [span access](https://eel.is/c++draft/span.elem).
- A `string_view` need not end at a null terminator. Supply a length-aware API
  or create an owned terminated string at a C boundary that requires one.
- Returning a view is useful when it borrows from a documented caller-owned,
  member-owned, or static range. Specify invalidating mutations and lifetime
  requirements. Return an owning value when the function creates storage or
  the result must survive its owner; never return a view into a destroyed
  local or temporary. Check retained callbacks and asynchronous work too.

C++17: this result intentionally aliases the input. The caller keeps the
underlying text alive and stable for every use of the result.

```cpp
#include <string_view>

[[nodiscard]] constexpr auto prefixBefore(const std::string_view text,
                                         const char delimiter) noexcept
    -> std::string_view {
    return text.substr(0, text.find(delimiter));
}

static_assert(prefixBefore("key:value", ':') == "key");
```

## Check the state that the type expresses

- Use C++17 `std::optional<T>` for a value that may be absent and C++23
  `std::expected<T, E>` for a value or an explicit error. Preserve the agreed
  failure contract; absence and failure are not interchangeable.
- Check presence or success before dereferencing. Neither type forces a
  check: `operator*` and `operator->` have preconditions. `value()` instead
  reports a wrong state by throwing; use it only when that failure behavior
  fits the project. Access `expected::error()` only in the error state.
  See [optional observers](https://eel.is/c++draft/optional.observe) and
  [expected observers](https://eel.is/c++draft/expected.object.obs).
- Use `std::variant` for a closed set of runtime alternatives. Prefer
  `std::visit` when all alternatives need behavior and `std::get_if` for an
  optional type match. `std::get` throws on a mismatch. Account for
  `valueless_by_exception` where throwing updates can produce that state;
  visitation checks types but does not make every operation non-throwing.
- Use templates when the type is fixed at compile time. Use `std::any` only
  when an open set of runtime payload types is required and the receiver has
  an explicit type contract. A variant usually makes a known set clearer;
  `any` moves type checks to runtime and may allocate. Check the project's
  exception, allocation, and RTTI constraints before choosing type erasure.

## Make conversions and representation access deliberate

- Prefer named casts that state the intended conversion. `static_cast` makes
  a conversion visible; it does not validate a numeric range or dynamic type.
  Use `dynamic_cast` for checked polymorphic navigation when RTTI and its
  failure forms fit. Casting away const does not permit modifying an object
  that was originally const.
- For representation copying, use C++20 `std::bit_cast` between equally sized,
  trivially copyable types. Check destination representations and initialization
  of relevant source bits. It does not perform numeric conversion, establish
  an aliased object's lifetime, validate a wire format, or fix endianness.
  Avoid dereferencing a reinterpreted pointer to violate aliasing rules. See
  [bit-cast requirements](https://eel.is/c++draft/bit.cast).
- Prefer `enum class` for scoped alternatives without implicit integer
  conversion. Validate externally supplied values against the accepted domain;
  a cast to an enum does not ensure a named enumerator. C++23
  `std::to_underlying` expresses the reverse conversion clearly.
- Use `explicit` when an implicit conversion would violate the interface's
  intent, and `[[nodiscard]]` where an ignored result is likely a mistake.
  These declarations support the contract; they do not validate input values.

## Validate numbers before conversion or arithmetic

- For C++20 integer conversions, check `std::in_range<Destination>(value)`
  before casting. Use `std::cmp_equal`, `std::cmp_less`, and related functions
  for mathematical comparisons across signedness. Their supported operands
  are signed or unsigned integer types, excluding `bool`, `char`, wide/UTF
  character types, enums, and floating point. See
  [integer utilities](https://eel.is/c++draft/utility.intcmp).
- Check domain constraints separately from representability: a representable
  negative length is still invalid. For floating-to-integer conversion, check
  finiteness and the actual destination range before converting.
- Conversion checks do not protect earlier arithmetic. Validate addition,
  multiplication, subtraction, division, and shifts before the operation that
  can overflow or violate a precondition. Casting the finished result to a
  wider type is too late; calculate in a proven sufficient type or use checked
  operations. Signed overflow is undefined; unsigned wrap is defined but often
  wrong for sizes and offsets.

C++23: this value-or-error interface checks representation before conversion.
Additional count limits belong in the domain's contract.

```cpp
#include <expected>
#include <utility>

enum class CountError { outOfRange };

[[nodiscard]] constexpr auto checkedCount(const long long input) noexcept
    -> std::expected<unsigned int, CountError> {
    if (!std::in_range<unsigned int>(input)) [[unlikely]] {
        return std::unexpected{CountError::outOfRange};
    }
    return static_cast<unsigned int>(input);
}
```

Choose rejection, a wider result, modular arithmetic, or saturation from the
required behavior. Saturation deliberately clamps a result and can hide an
invalid allocation size or offset; use it where clamping is the intended
domain behavior. C++26 saturation facilities remain subject to explicit C++26
selection and verified support; see [C++26 guidance](cpp26.md).

## Initialize meaningful state and bound compile-time claims

- Initialize scalars and members before use, preferably with their intended
  value at declaration. Use default member initializers for valid defaults.
  Zero initialization is useful only when zero is an acceptable state.
- Braced initialization rejects narrowing conversions, but its overload
  selection can differ from parentheses, especially with `initializer_list`.
  Choose the form that constructs the intended value.
- Apply `constexpr` when the implementation supports constant evaluation as
  written. A `constexpr` function can still execute at runtime; it is not a
  runtime overflow or bounds check. A required constant expression rejects
  disallowed operations on the evaluated path, not bugs in every possible call.
- Use C++20 `consteval` when calls must be evaluated at compile time. Retain
  runtime validation for runtime inputs and respect compiler evaluation limits.
  Use C++20 `constinit` for required static initialization when the object may
  remain mutable. See [constant expressions](https://eel.is/c++draft/expr.const)
  and [constant initialization](https://eel.is/c++draft/dcl.constinit).

## Preserve iterator and range validity

Prefer a named algorithm when it states the operation clearly. C++20
`std::erase_if` expresses filtering directly; in C++17 use a container-appropriate
erase/remove pattern or the iterator returned by `erase`. Check each container's
invalidation rules for insertion, erasure, growth, and rehashing. Indices also
need valid bounds and can identify a different element after mutation.

Keep predicate effects compatible with the algorithm and container. Algorithms
do not make mutation during traversal safe automatically. Range views may
evaluate lazily and retain references to storage or predicates; preserve those
lifetimes until evaluation finishes. To preserve lazy evaluation beyond a
local source's lifetime, make the pipeline own its source where supported;
check any borrowing inside its elements and callbacks separately. Materialize
when the contract calls for an immediately evaluated result container.
`borrowed_range` does not extend underlying storage
lifetime. Retain the [callback defaults](style-principles.md) in custom traversal
helpers; see [algorithm choices](code-modernization.md) for operation selection.

## Coordinate shared state and thread shutdown

- Use named RAII lock objects. `std::lock_guard` or C++17 `std::scoped_lock`
  suits scoped ownership; `std::unique_lock` supports condition-variable waits
  and controlled unlock/relock. An unnamed temporary releases at the end of
  its expression.
- Acquire several distinct mutexes together with `std::scoped_lock` when they
  protect one operation. Its multiple-lock algorithm avoids deadlock during
  that acquisition; it does not resolve cycles involving already-held locks,
  reentrant callbacks, waits, or thread joins. Keep critical sections bounded
  and follow the program's lock ordering. See
  [scoped locking](https://eel.is/c++draft/thread.lock.scoped).
- Use C++20 `std::jthread` for an owned worker whose lifetime ends with its
  scope. Its joinable destructor requests stop and joins; the worker must
  cooperate. A blocking operation may never observe the request, and joining
  while holding a mutex the worker needs can deadlock. Arrange shutdown and
  member destruction so borrowed state outlives the worker. Catch and transport
  worker failures through the agreed channel; an escaping thread exception
  terminates the process. See
  [joining threads](https://eel.is/c++draft/thread.jthread.class).
- Use atomics for independent atomic state or a precisely defined protocol;
  use mutexes when several fields share an invariant. `volatile` supplies
  neither inter-thread synchronization nor atomicity. Start with the default
  sequentially consistent ordering; weaken it only with a documented ordering
  argument. Relaxed operations do not publish unrelated memory. Release/acquire
  needs a matching synchronization relationship, not merely those labels on
  two unrelated operations. Atomic operations need not be lock-free. See
  [memory ordering](https://eel.is/c++draft/atomics.order).
- Use a C++20 `std::latch` for a one-time countdown and `std::barrier` for
  repeated phases with participating threads. Use a condition variable for a
  changing predicate protected by a mutex; wait with a predicate to handle
  spurious wakeups and avoid losing the condition between checking and waiting.
  Notifications are not stored work items.
- Make cancellation wake blocked workers. C++20 `condition_variable_any` has
  stop-token-aware predicate waits; ordinary `condition_variable` waits need
  an explicit shutdown predicate and notification protocol. Check the predicate
  result after a stop or timeout. See
  [interruptible waits](https://eel.is/c++draft/thread.condvarany.intwait).
