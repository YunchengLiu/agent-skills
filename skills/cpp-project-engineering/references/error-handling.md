# C++ Error Handling

## Scope

Apply when adding or changing a rejection, assertion, error return, thrown
exception, catch boundary, diagnostic, public failure comment, or failure-path
test.

This reference controls error form and reporting. Design resource lifetime,
cleanup, rollback, and post-failure state separately from it, using the
operation's contract and C++ ownership rules. Do not make error reporting own
those guarantees by accident.

## Hard Rules

1. Derive handling from the failed operation, supported domain, owning
   boundary, caller response, and guarantees already established. Do not start
   from an exception type or error framework.
2. Use the lightest form that carries the control flow and information the
   immediate caller needs. A local component does not throw merely because an
   outer API throws.
3. Preserve an API family's settled error form unless the task explicitly
   changes it. Do not introduce a parallel error model for local convenience.
4. Never silently discard a failure possible during supported use. Handle,
   return, or propagate it at its owning boundary.
5. Prefer a standard exception when its meaning completely describes the
   caller-visible failure. Add a custom type only for a required domain category,
   stable public boundary, or structured context.
6. Propagate an accurate original failure by default. Do not catch only to add
   a prefix, restate the same reason, or hide call depth.
7. Before translating an exception, check for a non-throwing operation, stable
   probe, existing status or error-code path, or local expected-like result.
   Translate once only when those forms do not preserve the required boundary.
8. Do not expose a private dependency's exception type through a public API.
   Decouple it at the narrow boundary that owns the public abstraction without
   building a parallel wrapper architecture.
9. A broad catch owns every subtype it includes. Do not absorb resource,
   system, programmer, or unknown failures merely to deduplicate catch bodies.
10. Do not repeat validation, fallback, or recovery for a property an earlier
    boundary already guarantees.
11. Diagnostics must be clear, neutral, concise, natural English. Include only
    domain and runtime context that materially identifies or diagnoses the
    failure; omit internal terminology and sensitive or unbounded data.
12. Prefer a direct fixed message. Format only runtime values with real
    diagnostic value, and add structured fields only when callers consume them
    independently of prose.
13. Document stable direct failure behavior and test project-owned behavior.
    Do not document incidental transitive failures or test basic C++ exception
    mechanics.

## Choose The Error Form

- `bool`: a predicate or probe whose caller needs only yes or no and for which
  `false` is unambiguous.
- `std::optional` or equivalent: a value may legitimately be absent and absence
  needs no error detail.
- the API family's expected-like value, status, or error code: anticipated
  failure that callers inspect, retry, combine, or analyze locally.
- exception: a throwing API cannot complete its promised operation and callers
  are not expected to branch locally on every failure.
- assertion or established unreachable mechanism: an explicit programmer
  precondition or internal invariant, never a replacement for supported runtime
  failure handling.

A parser step, verifier probe, retry loop, or helper may use a non-throwing form
even when its outer operation throws. Convert once at the owning boundary.
Conversely, do not add an expected-like public API solely to avoid one local
catch, and do not make a cheap probe throw only to obtain a verbose message.
Paired APIs may deliberately offer a cheap non-throwing probe and a throwing
diagnostic operation when each has a current caller.

## Standard And Custom Exceptions

Use the most precise standard category whose normal meaning exactly matches the
public failure:

- `std::invalid_argument`: an argument is outside this API's stated input
  domain;
- `std::out_of_range`: exceptional access or lookup names a position, key, or
  identifier outside the available range or set; use ordinary absence when
  missing is expected;
- `std::domain_error`: a valid call shape supplies values for which the
  mathematical or semantic operation is undefined;
- `std::length_error`: a defined size or capacity limit is exceeded,
  independently of allocation success;
- `std::overflow_error` or `std::underflow_error`: the stated numeric result
  cannot be represented;
- `std::logic_error`: the operation conflicts with a public state or sequencing
  rule and no more precise logic category applies;
- `std::runtime_error`: a valid operation fails for a non-system runtime reason
  with no more precise standard or domain category;
- `std::system_error`, `std::filesystem::filesystem_error`, and other standard
  exceptions carrying accurate structured system information normally
  propagate unchanged.

Use a custom type only when at least one current need is real:

- callers distinguish a domain or pipeline-stage failure;
- callers need structured source, code, path, range, entity, or operation data;
- a committed public API promises a stable domain failure form;
- a private dependency must be translated to preserve the public abstraction.

The module containing a throw is not by itself a reason for a custom exception.

A verifier's domain violation is a verification failure when its responsibility
is to decide whether an artifact satisfies a model; the artifact being a
function argument does not automatically make it `invalid_argument`. Likewise,
for well-formed domain data that the selected backend, runtime, or operation
cannot support, report the existing capability or domain failure instead of
classifying the value as invalid input.

## Propagation And Translation

Catch only when the current boundary owns a concrete job:

- translating to the error form that boundary promises;
- adding structured context unavailable elsewhere;
- aggregating failures into a defined result;
- preventing an exception from crossing an explicitly non-throwing boundary.

Before a translating catch, check in order:

1. a non-throwing overload that performs the operation and returns status;
2. an existing stable probe when the caller needs only a branch decision;
3. a local expected-like form for inspect, retry, or aggregation;
4. moving translation to the single owning outer boundary;
5. catch and translate only when the required outer behavior cannot be
   expressed cleanly through the preceding forms.

Do not replace one accurate throwing operation with check-then-act when it
duplicates expensive work, races mutable external state, or loses diagnostic
information. When cleanup does not alter the reported failure, catch by const
reference and use `throw;`.

Identical catch bodies do not prove that `catch (std::exception const&)` is
correct. Broad standard catches include failures such as `std::bad_alloc`; own
them only when the public translation policy genuinely applies to all included
subtypes.

## Private Dependencies

A private build or implementation dependency must not force its exception types
into a public API. Prefer its non-throwing interface when it supplies enough
information to construct the project-owned result. Otherwise translate once at
the narrow public or domain boundary.

Expose the public operation and stable useful context, not the dependency's
type hierarchy. Preserve useful error codes and fields instead of flattening
them into text. Include an underlying message only when it is safe, accurate,
and useful at the outer boundary.

Do not add an adapter hierarchy, duplicate a dependency operation, discard an
error code, or otherwise distort the design solely to hide a type that never
crosses the public boundary.

## Diagnostics And Construction

Prefer one short English sentence that identifies the operation or subject and
states the immediate reason. Add an identifier, index, path, value, or limit
only when it materially helps locate or understand the failure.
Prefer a readable domain value over an opaque underlying representation.

Avoid:

- helper names, internal phases, storage layout, or speculative root causes;
- uniform prefixes added only for visual consistency;
- mechanical label lists or literal translation;
- secrets, credentials, large payloads, or unbounded dumps;
- a generic message so vague that the failed area is unknowable;
- a detailed message that requires private implementation knowledge.

Exception text is diagnostic prose, not a machine-readable interface unless a
real external protocol explicitly makes it one.
Similar failures should carry similar useful information.

Prefer a direct construction with a fixed message, such as
`throw std::invalid_argument("...");`. Format only when runtime
context distinguishes the subject or explains the reason. Use the project's
existing formatting or exception-construction facility instead of inventing a
second helper. When that facility already accepts a format and arguments, do
not preformat the same message at the call site. When a custom exception has
repeated formatted construction, give that type a formatting constructor;
structured public data needs an appropriate constructor and typed fields, or
the type's existing tag-based construction. Do not flatten it merely to fit a
generic helper. Do not add helper parameters or stored fields only to make
prose more verbose.

Keep construction local and bounded. Building an error must not perform file,
device, network, lock-heavy, or other
failure-prone work. Use [formatting-boundary.md](formatting-boundary.md) for the
choice and visibility of formatting libraries.

## Logging

Logging does not replace the API error form. Record one useful event at the
boundary with the best stable context rather than logging the same failure in
each helper and catch. Follow [logging.md](logging.md) for level, stacktrace,
message, and performance rules.

## Public Comments

Document the stable direct failure form and core trigger callers need. Do not
list every allocator, container, standard-library, or transitive failure, or
every private subcase covered by one public condition. Update a comment when the
promised form changes; leave an accurate readable comment unchanged.

Use `@exception` according to [comment-style.md](comment-style.md). Do not add an
implementation comment that merely repeats an obvious throw or message.

## Failure Tests

Exercise the owning API and assert only the promised boundary:

- the most specific promised exception or non-throwing alternative;
- exact structured fields, codes, ranges, paths, or identifiers that form
  public error data;
- a small diagnostic fragment when it proves required context reached the
  message.

Do not assert a complete message unless exact text is a CLI, protocol, or other
explicit external contract. Do not add inheritance-only, constructibility-only,
throw/catch, or repeated prefix tests. Each case must contribute one distinct
failure signal under [test-style.md](test-style.md).

Directly construct an error in a test only when its constructor performs
project-owned behavior that cannot be exercised more appropriately through its
owning API. A formatting-and-throwing helper may have focused tests proving that
formatted arguments reach the selected error type; do not repeat standard
exception-semantics tests at its call sites.

## Ablation Check

Compare each exception type, catch, wrapper, translation, context field,
message argument, recovery branch, comment, and test with its removal or a
lighter error form. Remove it when the required behavior and independent
evidence remain. This comparison does not permit changing a settled public
failure form or discarding structured information callers actually use.
