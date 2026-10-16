# C++ Comment Style

## Scope

Apply to comments in C++ source and headers. Test first-line comments follow
[test-style.md](test-style.md) for content and this reference for mechanical
format. Preserve existing maintenance comments that remain accurate and
relevant; move them with their logic during migration. Preserve user-authored
wording. Correct facts made false by the change, and remove comments whose
owning logic was removed. Other rewriting needs a concrete content defect or
the user's requested scope; stylistic uniformity, conversational wording, or
an agent's judgment that a comment is obvious is not sufficient. Report
uncertainty about a user-authored comment instead of silently rewriting it.

Source comments default to zh-CN and follow the project's permitted languages.
Runtime-visible text is not a source comment and remains English under the
main skill.

## Commenting Stance

Comments serve two readers with different needs.

- A caller treats a public API as a black box, using its declaration and
  applicable public documentation. Explain what it does, how to call it,
  its results and failures, and the conditions needed for correct use.
- A maintainer reads the implementation and needs the model behind it: the
  strategy, invariants, data flow, and reasons for non-obvious choices. Write
  maintenance comments proactively. Introduce a complex function at the top of
  its body, with a compact ASCII diagram when the structure is easier to see
  than to read, then explain the key points where they occur, so that comments
  and code guide the reader together.

Both readers know ordinary C++ and the basic subject of the API, but not the
module's conventions, internal model, task discussion, or design history.
A comment supplies what the signature, type, names, and applicable surrounding
context cannot; ordinary
language or smart-pointer semantics need no prose. Complete means enough to
understand, use, and maintain the code in context; it does not mean proving
that no misuse is possible. Mechanical rules apply exactly regardless of how
much explanation is needed.

Use readable, idiomatic prose in the chosen language. Establish the facts
and structure first, then write as a fluent speaker would explain them to
the intended reader. Use direct sentences and established terms. Preserve
every contract fact and mechanical rule while polishing. A phrase from the
signature may also be the
most natural description; use it when it contributes needed meaning.

## Hard Checklist

Apply items 1–7 to header and non-test source comments and item 8 to every
source comment, including test first-line comments. The sections below explain
and illustrate each item.

1. **Reader.** Decide whether the comment serves a caller of the public API or
   a maintainer of the implementation. Classify by intended use, not symbol
   export. A public comment relies on the declaration and applicable public
   documentation; an internal comment may also rely on nearby implementation.
2. **Public API content.** Write top-down: what the API does, how to use it,
   and what to watch for, in plain direct sentences. Cover observable behavior,
   inputs, result, ownership or lifetime, failure form, and important
   boundaries as the caller needs them. Show accepted inputs by example rather
   than by a grammar or a list of boundary cases. When a mechanism affects
   correct use, state the resulting public behavior or caller obligation.
   Private representations and execution steps belong in maintenance comments.
   Leave out design history and task-relative wording such as `这里`, `上面`,
   `下面`, `本次`, or `后续`. A non-obvious type opens with its role and use
   model.
3. **Public API form.** Document the public API with Doxygen (`///`).
   Internal declarations, including detail namespaces, private members, and
   internal headers, need not use Doxygen; concise `//` maintenance comments
   suffice. A documented public function has
   `@brief`, one blank comment line, then its tags:
   - `@brief` is one complete sentence naming what the function acts on and
     what it produces;
   - every parameter has an `@param` entry and every template parameter,
     including a deduced one, a `@tparam` entry; a short entry suffices when
     the name already says most of it;
   - a remark about one parameter or the result goes in that entry; `@details`
     and `@note` hold only what applies to the whole declaration;
   - each tag keeps the responsibility given in Doxygen Tag Responsibilities.

   A parameterless trivial accessor needs at most its brief. An operator needs
   only a brief and any `@exception`, shared across its family with `@copydoc`.
4. **Maintenance content.** For new, changed, or migrated implementation, give
   non-obvious strategy, state transitions, data flow, invariants, index
   relationships, and boundary-preserving reasons the explanation they need,
   introducing a complex function at the top of its body and explaining local
   conditions and reasons beside the operations they govern. During review,
   follow the implementation in order and check that the overview is backed
   by these local explanations. A clear API comment does not replace this.
   Do not narrate nearby statements, write validation steps in checklist voice,
   or describe recovery for states no supported path produces.
5. **New and revised comments.** Cover the required contract or model, then
   keep only what the context does not already carry; omit a new comment when
   nothing remains. Existing comments follow Scope's preservation rules.
6. **Wording.** Keep established domain terms and acronyms stable and prefer
   natural prose in the chosen language over copied low-value identifiers.
   Use noun-centered language for types, results, and fields, verbs for
   behavior, and predicates for boolean queries. Match the detail to the
   missing model: no filler sentences, and no compression into cryptic text.
7. **Content before layout.** Settle the content first. The formatter owns
   wrapping; never cut, reword, or move facts between tags to fit a line.
8. **Mechanical format.** Apply exactly:
    - Use ASCII punctuation. The enumeration mark `、` is the sole exception.
    - Follow an in-prose comma, semicolon, or colon with exactly one ASCII
      space.
    - Add no spaces at Chinese-English or Chinese-identifier boundaries.
    - In prose, put no spaces inside parentheses. In Chinese prose, put no
      spaces around them; in English prose, use normal word spacing around
      parenthetical phrases.
    - End each logical comment line without sentence-ending punctuation. When a
      formatter wraps one sentence, keep punctuation required inside it.
    - Running prose is plain text: no backticks, emphasis, heading markers, or
      HTML tags. Doxygen lists and `@code` blocks remain available when needed.

After editing, inspect every touched `//`, `///`, and `/** */` line against this
list. Nearby comments do not override the mechanical rules even when they are
the local majority.

## Explain The Model, Not The Signature

### Prepare The Model

Before drafting a non-obvious component model, answer in order:

1. Who is the intended reader, and what do the declaration and applicable
   context already tell them?
2. What role, relationship, or operating model does that reader still need?
3. What does it produce or change, why does that result exist, and who uses it?
4. Which invariant, precondition, transition, or interpretation makes the code
   understandable?

Explain a real gap directly, even when it needs several sentences. A reader's
ability to inspect implementation does not replace the model that gives those
details meaning. Put the distilled conclusion in the comment, not the questions
or deliberation.

### Draft Public Comments In Three Passes

1. State the caller-visible operation or the type's role. For a function, think
   input -> operation -> observable result before writing the brief.
2. Place each remaining fact in the tag that owns it before considering its
   removal. Observable algorithm edge cases remain part of the contract.
3. Read only the declaration and applicable public documentation as a caller
   unfamiliar with the module. Check that they suffice to choose inputs,
   interpret results, and handle promised failures. Each statement must
   contribute to correct use; verify the description against the implementation
   for accuracy and missing contract facts. The brief must not promise more
   than the detailed tags. Fix awkward prose by separating mixed tag
   responsibilities, then reread for natural phrasing in the chosen language.
   Preserve facts, tag responsibilities, and mechanical format while polishing.

For example, a resampling operation preserves endpoints and permits a shorter
final segment. This implementation-led brief makes the caller reconstruct it:

```cpp
/// @brief 按弧长参数化逐段遍历路径, 在累计弧长达到间距整数倍处线性插值生成采样点,
/// 间距不整除总弧长时末段按剩余弧长收尾, 单位为微米
ScanPath resample(ScanPath const& path, double spacing);
```

This shorter draft loses the endpoint and final-segment behavior:

```cpp
/// @brief 把路径上的点重新撒均匀
ScanPath resample(ScanPath const& path, double spacing);
```

Keep the operation, inputs, result, and failures in their respective places:

```cpp
/// @brief 按给定弧长间距重采样路径
///
/// @param path 待重采样的路径
/// @param spacing 目标采样间距, 单位为微米
/// @return 包含原路径起点和终点的采样路径; 除末段外, 相邻采样点间的路径弧长为spacing, 末段弧长不大于spacing
/// @exception std::invalid_argument spacing不是正有限值时抛出
[[nodiscard]] ScanPath resample(ScanPath const& path, double spacing);
```

The same method applies to a device setter. In this example, `Milliwatt` is the
unit type and the named exceptions are the API's public error types. This draft
describes private execution order:

```cpp
/// @brief 设置目标值; 调用时先检查ready状态, 再检查范围, 然后更新缓存并调用实现
void target(Milliwatt value);
```

State the operation, unit, and observable failure conditions instead:

```cpp
/// @brief 设置目标值
///
/// @param value 目标值, 单位为mW
/// @exception DeviceNotReadyError 设备未就绪时抛出
/// @exception DeviceControlError 值越界或设备拒绝写入时抛出
void target(Milliwatt value);
```

The caller needs these conditions; validation order, cache updates, and private
calls remain implementation details.

### Establish A Component Model

Do not restate input and output types or turn implementation order into a
contract. State the role, operating model, invariant, produced result, or
consumer relationship that a reader cannot recover cheaply.

For an internal scanline component, this comment only restates its inputs
and outputs:

```cpp
// 以已排序边表为输入, 输出配对后的填充线段
class ScanlineEngine;
```

Explain the algorithm model and how its result is consumed:

```cpp
// 扫描线填充的核心扫描循环
// 边按下边界排序后, 扫描线沿单调方向线性推进
// 每条扫描线对当前活动边做Even-Odd配对, 产出该行的填充线段, 逐行回调给消费者
class ScanlineEngine;
```

A buffered controller needs to explain when a caller's actions take effect:

- A reader may expect a setter to act on the device immediately.
- Here it only appends to a command list; a separate execute request triggers
  the deferred, batched actions.
- The signature already shows the setters and execute method. Explain their
  operating model instead of listing those methods again.

```cpp
/// @brief 基于命令列表的控制接口
///
/// @details 调用方先写入命令列表, 再请求设备执行; 写入本身不触发硬件动作
class BufferedController;
```

An internal component may rely on upstream guarantees and explain how its
result is consumed. Use a diagram when it makes a feedback relation clearer:

```cpp
/// @brief 根据已验证的依赖关系构造任务执行顺序
///
/// @details 会话加载阶段已经保证依赖图无环; 本组件维护每个任务的剩余前驱数,
/// 将新就绪任务持续加入队列, 最终产出供执行器顺序消费的任务列表
///
/// @code
/// 已就绪任务 -> 就绪队列 -> 输出顺序
///                  |
///                  +-> 解除后继依赖 -> 新就绪任务
/// @endcode
class ExecutionOrderBuilder;
```

For an obvious operation, the name may already carry the action. Add only the
missing unit, reference frame, side effect, lifetime, or failure form:

```cpp
/// @brief 平移整条扫描路径, 坐标单位为微米
///
/// @param dx x方向偏移
/// @param dy y方向偏移
/// @param dz z方向偏移
/// @exception std::invalid_argument 偏移量不是有限值时抛出
ScanPath& translate(double dx, double dy, double dz);
```

Do not add documentation to a trivial getter, setter, or predicate when the
name, signature, type, and surrounding type comment already make the behavior
complete.

## Information Placement

Keep repeated semantic explanations in one closest owner:

- public module or type comment: shared concepts and interactions needed to
  use the API, including what a component or result represents and how it is used;
- internal component comment: the implementation model shared by its operations;
- field or local variant comment: the local distinction that would otherwise
  confuse maintainers;
- function comment: responsibility, inputs, result, mutation, failure, and
  caller-relevant boundary;
- implementation comment: strategy, invariant, transition, or non-obvious
  reason.

A local result type or variant explains the local semantic distinction most
likely to confuse a maintainer. When a special rule has a practical purpose,
state that effect instead of saying only that the rule exists. Keep definition,
use, result, and diagnostic roles distinct.

Define a result object's categories at their owner and let returning functions
refer to that model. Explain a component through its purpose, behavior, and
conditions; include a negative boundary only when it prevents realistic misuse.

Simple field comments use trailing `///<` by default. Use a preceding block only
when a field genuinely requires multi-line API documentation. A field comment
explains the field, not the whole enclosing type.

Use a compact ASCII diagram only when a spatial relation, feedback path, memory
layout, or pipeline is materially easier to understand visually than in one
sentence. Put a diagram in a Doxygen block inside `@code`; an ordinary
implementation comment may place it directly below its orienting sentence. Do
not add decorative diagrams.
Label the model being shown and keep the diagram local to the code it explains.

## Doxygen Tag Responsibilities

- `@brief`: one natural, complete sentence stating the stable role, operation,
  transformation, or result in its declaration context. For an operation, name
  what it acts on and what it produces.
- `@details`: caller-visible scope, interpretation, evaluation, coverage, or
  public operating model that cannot fit naturally in the brief. For internal
  declarations, it may explain an algorithm or pipeline. Public declarations
  describe sequence only when it is caller-visible behavior. Keep default-behavior
  reminders, transient reminders, statement-by-statement narration, and failure
  lists out of this tag. A remark about one parameter or the result belongs in
  that `@param` or `@return`.
- `@tparam`: the kinds of types a template parameter accepts.
- `@param`: input meaning, plus any unit, accepted range, ownership, default
  semantics, or remark specific to that parameter. Show accepted text forms by
  example rather than by a full grammar.
- `@return`: result meaning, value categories, ownership, and observable
  boundaries.
- `@exception`: the stable direct failure type and trigger condition. Do not
  enumerate incidental allocator, container, or transitive implementation
  failures.
- `@note`: an important, non-obvious reminder that does not belong to another
  tag, such as a material thread-safety, lifetime, stability, performance, or
  common-assumption boundary. It is not overflow space.
- `@warning`: a condition whose omission is likely to cause dangerous or serious
  misuse. Use `@note` or ordinary prose for normal boundaries.

Beyond the required `@param` and `@tparam` entries, use only tags that carry
needed information.

For a stream operation, distinguish the result from failure and configuration:

```cpp
/// @return 已成功读取字节的哈希摘要; 若读取到文件末尾, 则为完整内容摘要
/// @exception std::ios_base::failure 由std::istream触发, 是否抛出取决于输入流设置
/// @note 不修改is.exceptions(), 也不清除已有状态位; eofbit、failbit、badbit的设置行为与流配置一致
/// @note 若流不抛异常, 则本函数也不因读取失败而抛出; 返回的是"成功读取部分"的摘要
```

## Natural Chinese

- Prefer direct verbs and predicates over `对...进行...` and stacked abstract
  nouns.
- Keep professional domain terminology, but do not invent formal-sounding terms
  when ordinary wording is precise.
- Interpret an ambiguous English identifier through its algorithmic or domain
  role, not a dictionary gloss. If the role is unclear, retain the established
  domain term or rewrite the sentence.
- Do not repeat a type name as its explanation, such as `表示ASTNode类型`.
- Use noun-centered wording for a result and direct verbs for an operation.
- For repository prose and source comments, use a soft target of roughly 80
  ASCII columns when clauses wrap naturally. Keep a sentence or clause together
  when splitting harms readability; code, paths, identifiers, tables, tags,
  and diagrams may run longer. Do not split a sentence merely to even out line
  lengths or move facts between Doxygen tags to fit a line budget.

### Short Types And Natural Terms

A simple type may need only its role and one essential unit:

```cpp
/// @brief 可注入的时间提供者, 返回当前UNIX时间戳(秒)
struct TimeProvider;
```

Use a predicate for a boolean query: `模块是否有效`. Prefer `汇总当前检查结果`
to `汇总当前阶段产物的语义一致性状态`, and `AST的基本节点类型` to
`表示ASTNode类型`.

Keep established domain acronyms while replacing fragmented English prose:

```cpp
/// @brief per-channel设置carrier frequency
void frequency(Channel channel, Megahertz value);
```

```cpp
/// @brief 按通道设置RF频率
void frequency(Channel channel, Megahertz value);
```

### Result Identity And Local Detail

This names storage categories without explaining the result:

```cpp
/// @brief 按内部分类列表汇总结果, 包含若干已解析项、失败项与待处理项
struct ResolveResult;
```

State its meaning and use:

```cpp
/// @brief 引用解析产生的可用目标和诊断
///
/// @details 调用方使用可用目标继续构建依赖关系, 并集中报告无法解析的引用
struct ResolveResult;
```

With those categories owned by the result, this function comment repeats them:

```cpp
/// @brief 解析引用并返回结果; 成功项写入resolved, 失败项写入errors, 待后续处理项写入unresolved
ResolveResult analyzeReferences(Input const& input);
```

The function's responsibility is sufficient:

```cpp
/// @brief 解析当前输入中的引用并返回结果
ResolveResult analyzeReferences(Input const& input);
```

For a special retention rule, `该情况按既定规则继续保留旧记录` only restates
the rule. `继续保留已有结果, 避免后续诊断被级联噪声淹没` explains its purpose.
