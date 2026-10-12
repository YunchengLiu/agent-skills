# Formatting Boundary

## Scope

Choose standard-library formatting or fmtlib by product role and consumer
boundary, not by the directory containing the code.

## Hard Rules

1. **Required public and installed APIs use standard-library formatting only.**
   Public signatures and macros, required formatter hooks, installed headers'
   required compile paths, exported targets, and package dependency files must
   not require fmtlib.
2. **Optional public fmtlib support follows fmtlib availability.** A public
   type with a `std::formatter` also specializes `fmt::formatter`. That
   specialization and any other optional public fmtlib extension are enabled
   only when `<fmt/format.h>` is available, detected with `__has_include` or the
   project's wrapper for it. Never add a fmtlib dependency, include path, or
   link target to enable them; they leave the normal public API usable without
   fmtlib and add no public or exported fmt dependency.
3. **Internal implementation and final terminal programs use fmtlib by
   default.** Preserve direct standard-library paths at real standard-library
   boundaries. Do not add adapters or duplicate formatter stacks merely to
   satisfy the default.
4. **Examples use standard-library formatting and output.** They demonstrate
   the required public API and must not add fmtlib includes or target
   dependencies.
5. **Tests match the behavior under test.** Use standard-library formatting for
   required standard-library public contracts and fmtlib for optional extension
   behavior or concise internal test diagnostics.
6. **Use the selected library's mature API directly.** Prefer its formatting,
   printing, append, and buffer facilities over mixing formatted strings into a
   second output abstraction without a local reason.
7. **Error construction follows [error-handling.md](error-handling.md).** This
   reference chooses the library boundary; error handling decides whether a
   fixed message, project helper, or custom constructor is warranted.

## Dependency Visibility

fmtlib linkage follows the same boundary: private for internal implementation
and final terminal programs, absent from examples and required public/exported
dependencies. Before adding an unguarded include or a target dependency, verify
that fmtlib is present in the project's dependency declaration and give the
owning target the narrowest correct visibility.

The required public std-only boundary takes precedence over the internal fmtlib
default. For an exported static library, `PRIVATE` does not remove the final
consumer's link dependency: CMake retains it for transitive linking. Check the
installed consumer path; use standard-library formatting in that required
implementation path when fmtlib would otherwise become a required exported
dependency. See [CMake static libraries](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#static-libraries).

If the project does not yet provide fmtlib, this preference does not by itself
authorize a dependency change. When the current task actually requires choosing
or adding the internal formatting facility, recommend fmtlib by default and
apply the project's dependency-change rules.
