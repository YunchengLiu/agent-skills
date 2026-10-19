# Standard Selection

Establish the target standard before making version-specific recommendations.

## Source order

Use the strongest available evidence in this order:

1. Explicit user instruction
2. Project-declared language standard
3. Build system configuration and compiler flags
4. Toolchain evidence from the active environment
5. Local code patterns only as weak hints

## Working rules

- If the user says "target C++20" or similar, use that as the effective standard unless they later change it.
- If the project clearly declares a standard, align with it instead of guessing from isolated syntax.
- If compiler support and standard-library support differ, plan for the lower effective capability.
- If evidence is missing, conflicting, or too weak to trust, ask the user which standard to target.
- If the user asks for "modern C++" without a standard, do not silently jump to the newest version.
- Treat C++26 as an explicit target, not a default modernization baseline.

## Support checks

Library facilities need both compiler parsing support and a library
implementation; these can advance at different speeds. Check the actual
compiler version and library. `__cplusplus`, `/std:...`, or `-std=...` alone
does not establish library support.

Prefer implementation evidence in this order:

1. Feature-test macros from the relevant headers, often via `<version>`.
   Check values as well as presence: partial implementations can expose an
   earlier API revision.
2. Current vendor status pages for the active compiler and library. Use
   cppreference feature-test references as a supplement; prefer vendor
   evidence over memory when exact support matters.
3. Build evidence or a minimal local probe, rather than an assumption about
   newer facilities.

If support remains uncertain, ask, mark the recommendation as tentative, or
choose a supported alternative. Distinguish an adopted standard revision from
a proposal or later working draft. Use current sources rather than embedding
support tables in the skill.

Useful starting points are the [GCC C++ status](https://gcc.gnu.org/projects/cxx-status.html),
[libstdc++ status](https://gcc.gnu.org/onlinedocs/libstdc++/manual/status.html),
[Clang C++ status](https://clang.llvm.org/cxx_status.html),
[libc++ C++23 status](https://libcxx.llvm.org/Status/Cxx23.html), and
[Microsoft STL changelog](https://github.com/microsoft/STL/wiki/Changelog).
Choose the page for the actual compiler/library combination. A successful
probe in a newer local installation does not establish support for older
consumers.

## Output rule

Report the chosen standard briefly before proposing version-specific changes. If the standard remains uncertain, ask first.
