# C++ Unit Test Style

## Scope

Apply when adding or editing C++ unit tests. The naming rules below apply when
the project uses GoogleTest; otherwise preserve the same behavioral principles
through the project's established test framework and naming convention.

Test explanation comments use zh-CN and follow the mechanical rules in
[comment-style.md](comment-style.md).

## Hard Checklist

Before editing and before finalizing a new or changed test:

1. Start from changed behavior or an acceptance signal. A declaration alone
   creates no test duty.
2. Make each case prove one distinct behavior, supported boundary, rejection,
   or failure form.
3. For GoogleTest, end the suite name with `Test` and begin the case name with
   `Test`.
4. Keep the case name to one behavior, or one behavior plus one essential
   qualifier. Do not narrate setup, action, and assertion steps in it.
5. Split independently failing concerns or move shared setup into a fixture or
   helper before extending a case name into a sentence.
6. Omit the first-line comment unless it adds a non-obvious boundary,
   evaluation rule, or assertion intent.
7. Do not use a first-line comment to translate the case name, restate obvious
   assertions, or narrate Arrange/Act/Assert.
8. Test the project's abstraction and contract, not raw standard-library,
   language, or third-party behavior such as copyability, movability,
   `noexcept`, or declared types.
9. Keep only cases that add an independent failure signal.
10. Cover normal supported behavior and each meaningful supported-domain
    boundary or runtime failure form introduced or changed by the task. Do not
    invent cases to fill a checklist.
11. Treat explicit programmer preconditions separately: release behavior
    outside them is not promised. Use a death test only when the assertion is
    itself a deliberate stable debug requirement.
12. For error paths, apply [error-handling.md](error-handling.md) and avoid
    binding tests to incidental prose or basic C++ mechanics.
13. Do not reduce coverage or delete a test without a stated reason and a
    replacement for any behavior signal that still matters.

When a test fails, first check whether it expresses the settled behavior. Fix
the implementation rather than weakening the assertion; change the test only
when the contract itself changed with the required authority.

## Test The Local Contract

Cover the categories that materially exist:

- normal success;
- a key boundary, variant, ordering rule, or ownership rule;
- a supported rejection, failure, or invalid-input form.

Absence of a category is not a gap when the contract does not contain it. Do not
broaden the supported domain, invent invalid inputs, or promise recovery merely
to make a symmetric test table.

Every case must provide a distinct diagnostic signal. Two tests are redundant
when they use different literals or setup but prove the same contract and would
lead to the same diagnosis. Remove one unless the second isolates a different
boundary or failure mode.

## Assert Observable Behavior

Prefer assertions about:

- observable results and stable output;
- caller-visible state changes;
- ordering, ownership, or lifetime that belongs to the contract;
- explicit boundary handling;
- promised error type, code, or structured context;
- project invariants owned by the subject under test.

Do not make tests primarily about private helper order, incidental intermediate
state, storage layout outside the contract, or an exact implementation strategy
when multiple strategies satisfy the same behavior. Arithmetic identities and
facts guaranteed solely by C++ or a third-party library are not independent
test targets.

Use the smallest representative input that still exercises the claim. Put the
main assertion first and keep related assertions together. Avoid noisy or large
inputs unless size itself is part of the behavior being tested.

## GoogleTest Naming

Suite names describe the subject or a stable scenario family:

```cpp
ModuleBindingTest
IncludeResolutionTest
StorageLimitsTest
```

Avoid outcome or catch-all suite names such as `WorksTest`,
`ShouldResolveReferencesCorrectlyTest`, and `EdgeCasesAndFailuresTest`. A fixture
takes the suite's name; a case name adds only the behavior and does not repeat
the suite's subject, such as `TestAsZoneKeepsInstant` in `DateTimeTest` rather
than `TestDateTimeAsZoneReturnsSameInstantInAnotherZone`.

Case names describe one behavior:

```cpp
TestBindsReferences
TestAllocatesSlotsInSourceOrder
TestRejectsDuplicateParameters
TestEmptyInputNoOp
```

Avoid sentence-shaped names such as:

```cpp
TestBindsReferencesAndAllocatesSlotsInSourceOrder
TestCreatesModuleThenResolvesReferencesThenChecksOrder
TestReturnsEmptyRangeWhenOnlyTopLevelObjectExists
```

Treat `And`, `Or`, or a long `When...Then...` form as a split signal. A single
case remains valid when its assertions are inseparable parts of one contract;
split when groups can fail independently and convey different diagnoses. Treat
`With` followed by another behavior clause the same way.

Names up to roughly 30 characters after the `Test` prefix are normally fine.
Beyond roughly 40, rewrite or split unless an unavoidable domain term needs
that space. Move shared setup into a fixture/helper and missing evaluation
context into a first-line comment; do not extend the name with more clauses.

## First-Line Comments

Write one only when the name and first assertions do not reveal the key
boundary or evaluation rule. Add one missing piece, such as a state distinction
or non-obvious assertion intent, rather than a second full test description:

```cpp
TEST(StorageLimitsTest, TestRejectsCountOverflow) {
    // 计数的上限由剩余字节预算决定, 与普通格式错误分支区分
}
```

Comments can explain a combined-state predicate or a precedence rule:

```cpp
TEST(ReferenceResolutionTest, TestFullyResolvedState) {
    // 未解析引用与错误记录的是不同状态, 是否全部完成由isFullyResolved统一判断
}

TEST(IncludeResolutionTest, TestUsesExplicitEntryBase) {
    // 显式入口基路径优先于模块源路径, 用于从外部重定向首次引用解析
}
```

Do not write:

```cpp
TEST(ModuleBindingTest, TestBindsReferences) {
    // 测试绑定模块引用
}
```

Nor should the first comment narrate the test procedure:

```cpp
TEST(FillHolesTest, TestFillsSingleHole) {
    // 先调用补洞函数, 再检查补洞结果
}
```

If deleting the comment loses no useful information, delete it. Shared setup
belongs in a fixture or local builder; repeated data-shape details belong in a
helper name rather than every case name.

Move context into a fixture when every case shares the same setup mode. Use a
helper or local builder whose name states the data shape when setup detail would
otherwise bloat several case names. Do not keep extending a case name merely
because all current assertions happen to share one body.

For example, `makeModuleWithExplicitEntryBase`,
`makeClosedMeshWithSingleHole`, or `bindModuleAndCollectResult` carries setup
information that would otherwise lengthen every case name.

An assertion such as `EXPECT_EQ(2 + 2, 4)` proves no project behavior; replacing
it with more arithmetic assertions does not create a useful test.

## Failure Tests

Exercise the owning API that produces the failure. Assert only what it promises:

- the most specific promised exception, status, or expected-like alternative;
- structured fields or error codes that callers consume;
- a small diagnostic fragment only when it proves useful context reached the
  boundary.

Do not lock the full `what()` string unless exact text is an explicit CLI,
protocol, or other external contract. Do not add constructibility,
inheritance-only, throw/catch, or repeated message-prefix tests. A low-level
error-construction helper may have focused tests for behavior it owns; callers
should not retest the language mechanism.

## Final Check

Read suite, fixture, case name, first comment, input, and assertions together.
A maintainer should be able to identify the promised behavior and why a failure
matters without reading a paragraph of setup or private implementation details.
