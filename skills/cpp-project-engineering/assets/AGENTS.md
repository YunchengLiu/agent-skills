# Project Agent Instructions

## Working Principles

Work as the user's design-and-coding copilot. Keep decisions and results easy
to follow and review. The user and settled design own behavior, API shape,
failure forms, compatibility, security, and scale requirements; implement them
with sound ownership, lifetime, and resource handling.

First-principles reasoning and ablation apply to every task and subsequent
workflow. Use `first-principles` when available. Establish the required outcome,
facts, constraints, current path, consumers, and acceptance signal; choose the
smallest complete change. Carry the owning path and established guarantees
into local decisions; each part supplies the responsibility still missing in
its context. Compare each addition with removal, reuse, or a simpler form as
decisions arise and after each design, implementation, execution, and validation
stage. Keep routine judgments brief and reuse supported decisions unless new
facts or an unresolved concern could change them. Preserve required detail,
guarantees, and rule strength. For mechanical rules, make this comparison per
changed line and check every retained line against the applicable checklist.

## Expression

State the required behavior, conditions, and result directly. Use prohibitions
for concrete constraints and essential boundaries. Write instructions,
explanations, and collaboration in fluent, idiomatic language. Establish facts
and structure first, then polish wording while preserving meaning and rule
strength. Discussion corrections and temporary drafting notes must stay out of
lasting documentation; express effective requirements in their own right.

## Engineering Rules

Use `cpp-project-engineering` as the detailed engineering baseline. Read the
references for the touched surfaces before editing and check the diff against
them before delivery. Project-specific configuration and contracts take
precedence.

| Work | Entry point |
| --- | --- |
| Unsettled requirements or design | `planning-clarification` |
| C++ language and library choices | `modern-cpp` |
| CMake features and build-system choices | `modern-cmake` |
| Structure, comments, tests, errors, logging, and install conventions | `cpp-project-engineering` references |
| Verification, implementation closeout, and review | `cpp-project-engineering/references/validation-and-review.md` |
| Useful independent work or review | `use-subagents` |
| Task continuation | `task-handoff` |

## Workflow And Fallback

When a skill is unavailable, report that limitation and apply the principles
above and project instructions. Use relevant existing modules, utilities,
tests, and build configuration; keep their established style and usage.
Written rules remain authoritative.
Keep these basic requirements in either case:

- Production code, tests, and build changes are non-trivial by default. Scale
  design and closeout to the change; only the user may designate a specific
  typo, rename, or one-line comment/format task as trivial.
- Clarify unsettled behavior and material boundaries from evidence. Implement
  settled designs faithfully; reopen only a concrete conflict and choose local
  mechanics directly. Identify applicable declaration, implementation, test,
  and build locations. Obtain confirmation before expanding scope beyond the
  request and its necessary ripple; a report alone grants no authorization.
- Use the declared standard, configured dependencies, build tools, and
  formatter. Follow established file ownership and naming. Source comments
  use zh-CN; runtime-visible strings use English.
- Test changed behavior, supported boundaries, and failure forms. Preserve
  meaningful coverage. Before delivery, check necessity, contracts, affected
  callers, comments, tests, and build wiring; run change-matched validation.
- Complete an agreed spec phase and report decisions, deviations, validation,
  and risks before the next phase, unless several were requested together.

## Task Handoff

Use `task-handoff` for long tasks and before likely compaction, pauses, or
handoff. Default to `.agent-context/handoff/<task>/session.md`. Keep task-specific
probes, experiments, and temporary files in that task directory; project builds
and maintained deliverables retain their configured or agreed locations. If the
skill is unavailable, reuse the note matching the task's goal and scope; name a
new task directory by its stable objective. Keep only effective decisions,
current state, actual validation, and the next action in the note; replace stale
state and verify it on resume.

## Delivery

Report changes and reasons, verification that actually ran, deviations, and
material risks. State missing evidence and the next concrete check. For review,
report supported findings first and say when there are no substantive findings.
Commit only when requested, following the project's convention and deriving
scope from the owning target or delivered component.
