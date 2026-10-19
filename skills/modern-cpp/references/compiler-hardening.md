# Compiler Diagnostics and Hardening

Use this guide when a concrete correctness question or requested hardening work
calls for compiler, library, or binary evidence. Choose checks for the failure
being investigated. Ordinary C++ modernization does not authorize changes to
build settings, dependencies, or CI.

## Establish the Actual Configuration

Record the compiler driver and version, target architecture and OS, standard
library, C runtime, linker, language mode, and relevant dependencies. Clang can
use different libraries and linkers; its presence does not establish libc++ or
Linux behavior. Check both compilation and linking, then run on the target.

Separate three purposes: diagnostics expose potential or exercised defects, library
checks enforce selected preconditions, and deployment mitigations constrain
some consequences of defects. None establishes complete memory safety. Use
current vendor documentation and focused probes instead of fixed support or
overhead tables. Preserve required ABI, deployment, and performance constraints.

## Warnings and Analysis

Start from the project's warning policy. For GCC or Clang, `-Wall -Wextra -Wpedantic`
is a common baseline; evaluate `-Wconversion`, `-Wsign-conversion`, shadowing,
and other targeted diagnostics against the touched code. For MSVC, consider
`/W4` and relevant `/analyze` checks. `/permissive-` controls conformance behavior;
it is not a complete safety checker. Scope warning-as-error policy to code and
toolchains the project controls, and retain useful diagnostics from dependencies.

Investigate a warning before suppressing it. Uninitialized-use diagnostics vary
with optimization and control flow, and a clean warning run does not prove that
all reads, lifetimes, or synchronization are valid.

For C-style formatting, consider `-Wformat=2` to diagnose supported format-string
and argument mismatches. Clang's `-Wunsafe-buffer-usage` can identify buffer
operations for a focused migration; it does not prove that supplied lengths or
view lifetimes are valid. Check the actual compiler's diagnostics and API
annotations. See [GCC warning options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
and [Clang Safe Buffers](https://clang.llvm.org/docs/SafeBuffers.html).

For a requested iterator-invalidation investigation, an existing static-analysis
setup can complement runtime checks. For example,
[Itergator](https://github.com/trailofbits/itergator) supplies CodeQL models and
queries for this problem. Establish a usable analysis database and supported
container models, then confirm reported paths against actual invalidation rules.
Use such tooling when it adds evidence to the task; it is not a prerequisite
for ordinary local modernization.

## Select Deployment Mitigations by Platform

For GCC on GNU/Linux, inspect `g++ --help=hardened` before considering
`-fhardened`. Its bundle can change between compiler releases, and explicitly
specified options can prevent a bundled stronger option from taking effect.
It is not a portable Clang or Windows switch. Consult the
[GCC instrumentation options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html).

| Candidate | Decision and limitation |
| --- | --- |
| `-fstack-protector-strong` on supporting GCC/Clang targets | Protect selected stack frames against certain overwrites. Eligibility and generated code matter; this is not general bounds checking. |
| `-fstack-clash-protection` | Check target support for stack probing. Stack-clash protection addresses large growth past guard pages; stack canaries and `-fstack-check` are different mechanisms. |
| `_FORTIFY_SOURCE` with glibc | Add checks to supported C-library calls where the compiler can determine useful bounds. Verify the effective level and optimized build; defining a macro alone is insufficient. |
| `-ftrivial-auto-var-init=zero` or `pattern` | Reduce exposure from some automatic storage. Confirm support and effective mode, then fix missing source initialization and type invariants independently. |
| ELF executable: `-fPIE` and `-pie` | Produce position-independent executable code suitable for runtime address randomization. OS policy still determines randomization. |
| ELF shared library: `-fPIC` and `-shared` | Use the shared-library model where required; this is not an interchangeable executable recipe. |
| ELF `-Wl,-z,relro,-z,now` | Combine relocation protection with eager binding. Verify the final program headers and dynamic flags. |
| ELF `-Wl,-z,noexecstack` | Mark the stack non-executable. Check assembly and runtime requirements, and inspect the resulting stack program header. |

Fortification covers selected libc interfaces, not arbitrary C++ containers or
raw pointer accesses. Unknown object sizes can leave calls unchecked. Level 3
uses dynamic object-size information where supported and can introduce work at
runtime. Inspect glibc's effective `__USE_FORTIFY_LEVEL` and a representative call's
generated code; absence of a `_chk` call alone is inconclusive because optimization
can prove a call safe. See [glibc fortification](https://sourceware.org/glibc/manual/latest/html_node/Source-Fortification.html).

Automatic-variable initialization is a mitigation, not evidence that the source
establishes a meaningful value. Compiler treatment of uninitialized variables,
padding, exclusions, and defaults varies. Preserve explicit initialization and
uninitialized-read diagnostics. See [GCC initialization options](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
and the [Clang user manual](https://clang.llvm.org/docs/UsersManual.html).

Hardware control-flow protection and type-based CFI have different requirements.
On supported x86 GNU/Linux configurations, `-fcf-protection` participates in CET;
hardware, OS, linker, and participating objects determine effective protection.
Clang's `-fsanitize=cfi` checks selected control transfers and generally requires
LTO and suitable visibility. Review exported interfaces and shared-library
boundaries before changing visibility; cross-DSO support has separate limits.
See [Clang CFI](https://clang.llvm.org/docs/ControlFlowIntegrity.html) and
[GNU linker options](https://sourceware.org/binutils/docs/ld/Options.html).

For native Windows, evaluate MSVC mechanisms in their own terms: `/GS` stack
checks, `/guard:cf` compilation with `/GUARD:CF` linking, `/DYNAMICBASE` for ASLR
compatibility, `/NXCOMPAT` for DEP compatibility, and `/CETCOMPAT` where supported.
CFG requires compatible compiler/linker settings and `/DYNAMICBASE`; CET support
also depends on the target environment. These do not repair the underlying
invalid access. Consult [MSVC security features](https://learn.microsoft.com/en-us/cpp/code-quality/build-reliable-secure-programs?view=msvc-170),
[/GS](https://learn.microsoft.com/en-us/cpp/build/reference/gs-buffer-security-check?view=msvc-170),
and [CFG configuration and inspection](https://learn.microsoft.com/en-us/cpp/build/reference/guard-enable-control-flow-guard?view=msvc-170).

## Standard-Library Checks

Select the actual library's controls before its headers are processed, including
any precompiled headers or header units. A language-mode flag does not enable all
available checks, and a consumer macro does not rebuild a precompiled library.

For libc++, set `_LIBCPP_HARDENING_MODE` to a supported mode constant:

| Mode | Intended selection |
| --- | --- |
| `_LIBCPP_HARDENING_MODE_FAST` | Inexpensive checks focused on security-relevant preconditions. |
| `_LIBCPP_HARDENING_MODE_EXTENSIVE` | Additional precondition checks, including more logical errors; measure the workload. |
| `_LIBCPP_HARDENING_MODE_DEBUG` | The broadest checks, including expensive or heuristic checks, for diagnostic builds. |

Read the [libc++ hardening documentation](https://libcxx.llvm.org/Hardening.html)
for the installed version's categories and failure behavior. The mode alone does not enable every
bounded-iterator facility; some require vendor ABI configuration. Even bounded
vector or string iterators do not generally diagnose invalidation. Hardening
does not establish object lifetime, validate arbitrary pointer ranges, or make
continued execution after a violated precondition safe. The mode itself and
ABI-changing iterator options have different compatibility implications.

For libstdc++, `_GLIBCXX_ASSERTIONS` enables selected precondition checks;
`_GLIBCXX_DEBUG` adds broader container and iterator diagnostics. Debug-mode
containers can have different representations: keep their configuration
compatible wherever those objects cross translation-unit or library boundaries.
See [libstdc++ macros](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_macros.html)
and [debug-mode interoperability](https://gcc.gnu.org/onlinedocs/libstdc++/manual/debug_mode_using.html).

For supporting MSVC STL versions, `_MSVC_STL_HARDENING=1` enables the library's
selected checks. `_ITERATOR_DEBUG_LEVEL` is a separate ABI-sensitive diagnostic
setting; level 2 belongs to Debug builds, and level 1 is not a preferred
replacement for STL hardening. Verify defaults and compatibility for the
installed toolset. See [MSVC STL hardening](https://github.com/microsoft/STL/wiki/STL-Hardening)
and [_ITERATOR_DEBUG_LEVEL](https://learn.microsoft.com/en-us/cpp/standard-library/iterator-debug-level?view=msvc-170).

## Diagnostic Builds and CI

Choose a sanitizer for the suspected failure and supported target:

| Tool | Useful evidence and coverage limit |
| --- | --- |
| AddressSanitizer (ASan) | Many out-of-bounds and use-after-free accesses. Container spare-capacity checks depend on library annotations; ASan does not generally detect uninitialized values or data races. |
| UndefinedBehaviorSanitizer (UBSan) | Selected undefined operations such as signed overflow, invalid shifts, and alignment violations. It does not detect all undefined behavior. |
| ThreadSanitizer (TSan) | Data races on exercised paths. It does not prove a concurrent algorithm correct or test every possible interleaving. |
| MemorySanitizer (MSan) | Uses of uninitialized values. It needs a compatible instrumented environment, including relevant dependencies and usually an instrumented standard library. |

Use the compiler driver to compile and link with the selected sanitizer flags.
Retain debug information and suitable stack traces, often with `-g` and
`-fno-omit-frame-pointer`. Instrument relevant dependencies where feasible and
record coverage gaps; uninstrumented code, custom allocators, and assembly can
hide faults or complicate reports. Runtime availability is a separate check.

ASan plus UBSan is a useful supported combination for many GCC/Clang test builds.
Use separate configurations for ASan, TSan, and MSan; they are not a combined
profile. Inspect recovery policy: when a CI job must fail on a UBSan report,
select an appropriate non-recovering mode such as `-fno-sanitize-recover=undefined`.
Run the workload that reaches the suspected path, and preserve symbolized reports.

Native MSVC provides `/fsanitize=address` with its own architecture, runtime, and
option constraints. Do not infer native Windows TSan or MSan support from Linux
Clang documentation. Check [MSVC ASan](https://learn.microsoft.com/en-us/cpp/sanitizers/asan?view=msvc-170)
and its [known limitations](https://learn.microsoft.com/en-us/cpp/sanitizers/asan-known-issues?view=msvc-170).
For LLVM, use the individual [ASan](https://clang.llvm.org/docs/AddressSanitizer.html),
[UBSan](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html),
[TSan](https://clang.llvm.org/docs/ThreadSanitizer.html), and
[MSan](https://clang.llvm.org/docs/MemorySanitizer.html) manuals.

When CI work is authorized, add an explicit diagnostic configuration for the
relevant tests, with compile and final-link settings applied to the intended
targets. Start with one useful profile; add concurrency or uninitialized-memory
jobs when their coverage justifies the environment and dependency work. Preserve
the ordinary build and report which paths were exercised. Do not treat diagnostic
sanitizer runtimes as a general production hardening preset.

## A Small Linux/ELF Probe

In a disposable directory, this Bash example requires GCC with `-fhardened`, a
compatible glibc/ELF toolchain, sanitizer runtimes, and GNU `readelf`. C++20 is an
example baseline; retain the project's actual standard when adapting it.

```bash
cat > probe.cpp <<'CPP'
#include <cstdio>
int main() { return std::puts("configuration probe") < 0; }
CPP
g++ --help=hardened
g++ -std=c++20 -O2 -fhardened -Wl,-z,noexecstack probe.cpp -o probe-hardened
g++ -std=c++20 -O1 -g -fsanitize=address,undefined \
  -fno-omit-frame-pointer -fno-sanitize-recover=undefined probe.cpp -o probe-sanitized
./probe-hardened
./probe-sanitized
readelf -h -l -d -n probe-hardened
```

This checks configuration and startup only. Use a relevant isolated failing case
to verify a detector, then test the fix; this source does not exercise every
mitigation or guarantee a stack canary or fortified call will be generated.

## Production Sampling and Binary Evidence

[GWP-ASan](https://llvm.org/docs/GwpAsan.html) samples allocations through a
supporting allocator to detect some heap overflows and use-after-free faults in
deployed workloads. It requires allocator integration and usable crash reports;
it is not a universal compiler switch. Coverage is probabilistic, excludes stack
and global storage, and ends for a freed slot when that slot is reused. Select
sampling and rollout from the deployment's measurements; retain ordinary tests.

Inspect the actual shipped binary and relevant libraries. On ELF, use the
installed `checksec` tool's documented interface for a summary and `readelf` for
headers, segments, dynamic flags, and architecture properties; see the
[readelf manual](https://sourceware.org/binutils/docs/binutils/readelf.html). Confirm PIE,
`GNU_RELRO`, eager-binding flags, and `GNU_STACK` permissions as applicable.
On Windows, use `dumpbin /headers /loadconfig` for PE mitigation and CFG metadata.
Metadata proves properties of that artifact, not complete runtime enforcement
or absence of exploitable defects.

For an uncertain code-generation claim, inspect local assembly or disassembly
with the actual optimization, target, LTO, and dependency settings. Compiler
Explorer can illustrate a small reproducer when its configuration matches; it
does not establish the contents or performance of the deployed binary. Report
what was observed, what was exercised, and the remaining coverage limits.
