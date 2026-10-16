---
name: ai4science
description: >
  Guide stage-aware Python scientific and AI4Science experiment work: clarify the
  experiment stage, evaluation protocol, representation, reproducibility,
  artifacts, and framework reuse while keeping the current workflow sufficient,
  clear, maintainable, and repo-context driven.
---

# AI4Science

## Purpose

Use this skill to land and iterate on Python-based scientific and AI4Science experiment
code while keeping small experiments clean, explicit, and easy to evolve. The
skill keeps the current repo and current scientific question in charge: inspect
the local context, identify the smallest useful model of the problem, then add
only the structure that carries clear meaning for the current question.

## Hard Rules

- Use the current experiment stage as the primary design input.
- Inspect relevant repo code, data flow, and existing conventions before
  proposing structure, fields, artifacts, or dependency changes.
- Prefer the simplest working research structure that is clear, explicit,
  rerunnable, and easy to maintain. Engineering formality, architectural purity,
  and document structure are secondary unless the current workflow benefits
  from them.
- Add modules, classes, configuration objects, artifact types, and workflow
  machinery only when they make the current experiment materially easier to
  understand, change, reuse, verify, or rerun.
- Model the common structure of the problem before naming fields. Treat a
  low-information flag as a signal to search for a better shared representation.
- Add core dimensions when they express stable, meaningful properties or
  relations of the problem. Keep values derived when they are cheaper and
  clearer to compute at the use site.
- Treat data loading, preprocessing, modeling, evaluation, and reporting as
  reasoning stages, not mandatory modules or classes. Split them only when the
  separation makes the current code or workflow clearer and easier to maintain;
  keeping related steps together is valid when the main path remains readable.
- Keep logging, checkpoints, metrics, and artifacts low entropy: record values
  that are needed to understand, compare, rerun, or debug the current workflow.
  Add richer provenance only when the experiment stage requires it.
- Let documentation and artifact layout follow actual research use. Persist
  information needed to interpret, reproduce, compare, or resume work; omit
  routine activity, transient attempts, and format-only ceremony.
- Confirm metric, split, and leakage assumptions before changing train/eval
  logic.
- Keep holdout test data for final reporting, not model selection or iterative
  feature tuning.
- Ask before installing packages, creating environments, changing dependency
  ownership, or relying on compatibility-sensitive framework behavior.

## Workflow

1. Establish the stage: early setup, local debugging, active iteration, mature
   comparison, or repeatable workflow.
2. Inspect the relevant local context: entry points, data paths, transforms,
   model/evaluation code, outputs, and dependency files.
3. Run the representation checkpoint when the task involves feature extraction,
   classification, preprocessing, schema design, or new fields.
4. Identify the main research path and choose the smallest organization that
   keeps it clear. Use direct code, local functions, or modules according to the
   current amount of logic and reuse; do not map each reasoning stage to a code
   entity.
5. Confirm scientific-integrity basics before train/eval changes: primary
   metric, validation protocol, split ownership, and leakage risks.
6. Reuse mature framework-native infrastructure before inventing local
   experiment machinery.
7. Add logging, checkpoints, and artifacts only to the depth needed for the
   current stage.
8. Surface only assumptions or structural choices that materially affect
   scientific validity, interpretation, or maintenance of the current task.

## Discuss Before Coding When Needed

Pause for a short, natural design discussion only when:

- the user asks for feature extraction, classification, analysis variables, or a
  new data schema but has not stated the modeling assumptions;
- the implementation would add several dimensions, flags, configuration keys,
  log fields, artifact files, checkpoint entries, metrics, or
  artifact types;
- a sample has unusual properties that could become fixture-only fields;
- the current implementation has become difficult to understand or change and
  several materially different organizations remain plausible;
- the metric, split, or leakage assumptions would change the implementation.

Explain only the relevant entities, representation, downstream use, recommended
smallest design, and unresolved choice. Use prose or a small list according to
the content; do not require a fixed format.

## Reference Routing

- `references/stage-principles.md`: read when the experiment stage controls how
  much structure to add.
- `references/representation-and-decomposition.md`: read before adding feature
  fields, schemas, stage modules, or pipeline structure.
- `references/environment-and-compatibility.md`: read when runtime, dependency,
  or portability choices matter.
- `references/ecosystem-and-reuse.md`: read before building local framework
  machinery.
- `references/structure-principles.md`: read when balancing clear decomposition
  against needless architecture.
- `references/python-code.md`: read for Python implementation, refactoring,
  review, Ruff setup, and lightweight verification.
- `references/reproducibility-and-artifacts.md`: read when outputs,
  checkpoints, logs, or reruns are part of the task.
- `references/boundaries.md`: read when the task may be PyTorch mechanics,
  scientific validity, or another specialized domain instead of experiment
  workflow.

## Self-check

Before finalizing a recommendation or patch, verify:

- the chosen structure matches the current experiment stage;
- every new field, dimension, log entry, artifact, checkpoint entry, or metric
  has a current purpose for understanding, comparing, rerunning, or debugging;
- low-information flags have been replaced by a common representation or kept
  outside the core model with a stated reason;
- IO, preprocessing, modeling/classification, and reporting boundaries are clear;
- for train/eval changes, metric, split, and leakage assumptions are stated
  or explicitly blocked;
- the design leaves repo-specific implementation details to the current code
  context instead of imposing a generic framework.
- every added module, class, configuration layer, artifact type, or document has
  a current research or maintenance payoff rather than only a conceptual label.

## Response Expectations

Explain why the chosen level of structure is sufficient now. Mention the
experiment stage only when it materially changes the recommendation. Include
representation or decomposition only when it affects the current task. Flag
dependency or environment changes for approval. When modeling assumptions are
missing, ask a small number of concrete questions or state the unresolved points
instead of inventing fields.
