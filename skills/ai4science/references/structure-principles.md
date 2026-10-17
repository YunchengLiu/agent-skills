# Structure Principles

## Keep the main path readable

- Make it easy to find data loading, model creation, training, evaluation, inference, logging, and output handling.
- Keep the control flow understandable after metrics, logging, and checkpoint support are added.
- Prefer direct code paths over clever indirection for ordinary experiment logic.

## Decomposition Method

Reason about common research stages such as:

- IO and dataset discovery;
- deterministic preprocessing or feature extraction;
- model inference, classification, or scoring;
- evaluation, reporting, and artifact writing.

These stages describe the flow; they do not require separate modules, classes,
configuration objects, or files. Keep related stages together when they share
state and iteration and the path remains readable. Split only when the resulting
boundary materially improves understanding, independent reuse or verification,
or maintenance of the current workflow.

Evaluate the whole structure. A new entity should remove more confusion or
duplication than it adds through naming, construction, wiring, navigation, and
state transfer. A direct script or a single file with local functions remains a
valid maintainable design whenever it is easy to inspect and change.

## Allow simple centralized configuration early

- A dedicated Python file with low-frequency, mostly read-only globals can be acceptable in early phases if it keeps the experiment easy to adjust.
- Do not introduce serialization layers, complex config systems, or argument surfaces before the task really needs them.
- Keep the structure organized enough that the current configuration style can later be consolidated without a rewrite.

## Add structure only when it pays off

- Add a function, module, class, configuration layer, or artifact type only when
  it materially improves the current workflow's clarity, reuse, verification,
  or maintenance.
- Add argument parsing when local and remote execution or repeated reruns need a more stable interface.
- Add stronger organization when the current shape starts to slow down iteration or make mistakes likely.
- Notebook-based exploration can be acceptable early, but once logic is rerun or shared repeatedly, move stable parts into scripts or modules and let notebooks stay thin.

## Simplest is best

- Prefer the simplest structure that keeps the experiment understandable, portable, and recoverable.
- Do not confuse roughness with failure if the current form is clear and easy to evolve.
- Prefer natural files and notes whose content supports research use; do not add
  document layers or fixed formats for appearance alone.
