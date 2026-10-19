# Adoption Principles

Choose modern facilities by the job they make clearer or safer. A supported
facility with a concrete local benefit is a normal implementation choice.

## Default decision checklist

Adopt within the effective standard and library support when it provides one
or more of these benefits:

- It clarifies intent, interfaces, contracts, or ownership.
- It improves correctness or resource safety by reducing a real class of bugs.
- It reduces accidental complexity or boilerplate without obscuring behavior.
- It keeps diagnostics and debugging reasonable for the team.

Preserve readability, diagnostics, and debuggability in every choice; keep
maintenance costs and relevant consumer requirements visible.

Prefer not to adopt when most of these are true:

- The gain is mostly stylistic.
- The change would expand scope far beyond the local problem.
- The feature would make public interfaces harder to support.
- The debugging or compile-time cost would rise materially.
- The same result is already expressed clearly with simpler code.

## High-value patterns

- **Resource ownership:** use RAII and standard ownership types so normal exit
  and failure paths share cleanup. Prefer values or `std::unique_ptr` for one
  owner; use `std::shared_ptr` when lifetime is actually shared.
- **Borrowed input:** use `std::string_view` for read-only text and `std::span`
  for contiguous elements whose owner outlives the call or retained view.
- **States and results:** use `std::optional` for value-or-absence,
  `std::variant` for a closed set of alternatives, and `std::expected` for a
  value-or-error contract. These types express different models; choose the
  one the caller needs and retain the agreed failure form.
- **Sequence work:** use named algorithms, ranges, projections, and standard
  container operations when they expose the operation more clearly than
  iterator plumbing or a custom helper.
- **Generic operations:** use `if constexpr` for type-dependent implementation
  and concepts for supported operations and useful diagnostics. Keep the
  constraint matched to what the implementation invokes.
- **Local intent:** use scoped initializers, structured bindings, and stronger
  types to make state, units, or distinctions clear where realistic misuse
  would otherwise remain.

## Match the check to the cost

Check lifetime at borrowed-view boundaries, compile-time and diagnostic cost
in template-heavy code, and execution cost in affected hot paths. At a stable
API or ABI boundary, verify the supported consumer's standard and toolchain
before exposing a newer facility. These checks qualify the selected change;
they do not replace selecting a useful modern expression.
