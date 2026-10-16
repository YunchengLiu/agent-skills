---
name: first-principles
description: >
  Reason from required outcomes, facts, and constraints, and test each part
  through ablation. Use throughout analysis, planning, design, implementation,
  problem solving, verification, and review; scale depth to the decision.
---

# First Principles

Start with what the result must accomplish. Derive the approach from current
facts and constraints; use established practice to inform that reasoning.

## When To Apply It

Apply this method throughout task framing, planning, design, implementation,
diagnosis, verification, review, and delivery. At each substantive decision about responsibilities,
mechanisms, expression, or evidence, check what the required result still needs
and whether reuse or a simpler form supplies it. Keep routine judgments brief;
deepen the comparison when uncertainty or consequences warrant it. The reasoning
guides the work without requiring a separate report for every action.

## Establish The Problem

1. State the required outcome and the observable evidence of success.
2. Identify the relevant facts, constraints, scope, and established invariants.
   Separate confirmed facts from assumptions that still need evidence.
3. Trace the current process: what it already provides, where the necessary
   information is available, who uses the result, and what remains missing.
4. Read each part in context: its enclosing scope, adjacent explanation, and
   relationship to the whole determine what it means.

## Derive And Compare

Start with the simplest approach that delivers the complete outcome. Assign
each responsibility where the information needed to fulfill it is available,
and let subsequent steps use the established result. Carry this context into
implementation and review: each part supplies its remaining responsibility and
may rely on facts and guarantees established in its applicable scope. Follow
the task's complete path through its consumers; inspecting the whole project
is needed only when that impact requires it.

For a consequential choice, compare credible alternatives under the same
established requirements, including the simplest compliant approach. Identify
the facts that support the current choice and the evidence that would favor a
simpler one; check those premises before deciding. Base necessity on current
requirements and evidence. When a premise fails, revise the approach within the
agreed scope. Consult relevant evidence and mature approaches, and identify the
principle that makes an approach fit this problem. Resolve assumptions that
could change the choice with a focused question, inspection, or experiment.

Keep the detail needed for someone to understand the result and its reasons.
Completeness includes the connections, affected users, and evidence required
for the result to work.

## Ablation

When choosing an addition, and after each design, implementation, execution,
or validation stage, compare it with removal, reuse, or a simpler alternative.
At stage boundaries, inspect the actual result for omissions and drift:

1. Which requirement, invariant, useful information, or distinct acceptance
   signal would be lost?
2. Could an existing method or a more direct approach meet the same requirements?
   Consider replacing related parts together and judge the complete result;
   dependencies within the current approach alone do not establish necessity.
3. Does the retained part solve the stated problem at the required scope?

Remove a part when the simpler version preserves the required result. Retain
rules, explanations, examples, and checks whose removal would weaken a
requirement or create a likely misunderstanding. Brevity follows clarity.

For mechanical rules, compare each changed line with the simpler version,
then check every retained line against the applicable checklist.

Use a controlled experiment when reasoning alone cannot distinguish the
alternatives: hold other relevant conditions fixed, change one factor, and
compare the same acceptance signal. Distinguish measured results from reasoned
expectations. Finish when the required result and sufficient evidence are
established. Carry forward supported decisions and valid evidence; repeat a
comparison or check only when changed requirements, artifacts, new evidence,
or an unresolved concern could change its result. A stage check can reuse those
decisions while checking what changed.
