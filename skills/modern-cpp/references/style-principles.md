# Style Principles

Apply these semantic style defaults across projects unless the repository defines stronger local rules.

## Core defaults

- Prefer simple interfaces with explicit data flow.
- Prefer value semantics for local ownership and regular types.
- Make borrowing, ownership, and lifetime visible in types and function signatures.
- Use RAII to make cleanup automatic and exception-safe.
- Prefer standard-library facilities over custom wrappers when they carry the right meaning.
- Use `typename` for ordinary template type parameters, including packs.
  Keep this spelling consistent in declarations, definitions, and examples.

## Template constraint style

- In C++20 and later, use explicit `typename` parameters followed immediately
  by one `requires` clause on its own line for constrained named templates.
  Keep this form even for one concept, such as `requires std::integral<T>`;
  `template <Concept T>` and abbreviated `Concept auto` parameters are not the
  default for named template interfaces.
- Define and reuse meaningful named concepts; reference them and combine
  related conditions in the `requires` clause.
- Use a trailing `requires` clause only when syntax or the required scope
  calls for it, such as a constrained non-template member of a class template
  or a constraint that needs function parameter names.
- Keep constraint form, ordering, and placement consistent between matching
  declarations and definitions. When changing an existing spelling, preserve
  the constrained expressions, cv/ref forms, and overload behavior.

## Abstraction rules

- Introduce abstraction to capture a stable concept, constraint, or invariant.
- Require a concrete benefit in correctness, ownership, lifetime, dependency
  direction, reuse, or caller clarity before adding a type, interface, or
  helper. Count construction, conversion, wiring, diagnostics, and navigation
  as costs.
- Keep closely related state and behavior together when they share ownership,
  lifetime, invariants, and reasons to change.
- Do not introduce abstraction only to look modern.
- Keep templates and concepts narrow and readable.
- Prefer direct code over generic machinery when the generic version does not materially improve reuse or safety.
- For an internal callback, consumer, or visitor called repeatedly, default to
  a template parameter taken by value, constrained when the effective standard
  supports constraints, and invoke it as a stable lvalue. Do not use a
  forwarding reference for that case; callers with heavy state can pass
  `std::ref`.
- In C++20 and later, express the invoked form in the constraint, for example
  `std::invocable<F&, Item&>` when calling `std::invoke(callback, item)`.
  In C++17, retain the by-value and lvalue-invocation defaults without concepts.
- Do not hand-roll `void*` plus a function-pointer trampoline merely to move
  that template body into a `.cpp`. Depart from the template default only for a
  measured instantiation cost or a real ABI or plugin boundary, then use a
  standard mechanism such as `std::function` or an abstract interface. When
  type erasure is warranted, choose a wrapper by ownership: `std::function`
  for copyable ownership, C++23 `std::move_only_function` for move-only
  ownership, or supported C++26 `std::function_ref` for bounded borrowing.

## Cost visibility

- Avoid APIs that hide allocation, ownership transfer, or expensive work.
- Be careful with lazy views, proxy types, and deeply composed pipelines in code that must be debugged often.
- Treat compile-time cost as a real cost.

## Contracts

- Use stronger types when they prevent a realistic misuse, preserve an
  important invariant, or establish a meaningful boundary. Semantic distinction
  alone does not require a wrapper type.
- Apply specifiers, qualifiers, attributes, and type deduction when they
  truthfully describe the code; omit them only when false, uncertain, or
  requiring a design change. In particular:
  - decide `noexcept` from the implementation and mark every function that
    cannot throw;
  - make unmodified locals and by-value parameters `const`, on the definition
    only when the declaration is separate;
  - mark error and exception branches `[[unlikely]]`;
  - use `[[nodiscard]]` where ignoring the result is likely a mistake, and
    `auto` where the initializer already shows the type;
  - use `constexpr` where the implementation supports compile-time evaluation
    as written.
- Keep error handling explicit. Choose mechanisms that match the contract and the calling style.

## Public documentation

- Describe APIs from the caller's perspective and make their intended use
  understandable without internal design context.
- Document copy, ownership, lifetime, concurrency, and failure details only when
  they affect correct use.
- Keep internal architecture classifications and planning vocabulary out of
  public names and comments unless they are established domain language.

## Change posture

- Prefer focused improvements that leave surrounding stable code alone.
- Suggest follow-up migrations separately when they require broader coordination.
