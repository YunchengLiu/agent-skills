# Stage Principles

Treat project stage as the main input for engineering structure, recordkeeping,
and recovery support. Choose scientific evidence for the question and claims
being evaluated; use the stage to set the depth appropriate to the current work.

## Early setup or local debugging

- Prioritize getting a runnable baseline with clear inputs and outputs.
- Keep the structure simple.
- Allow lightweight centralized configuration if it keeps the experiment easy to inspect and change.
- Save only what is needed to recover, inspect, or avoid losing work.
- Minimum expectations:
  - a clear entry point
  - obvious inputs and outputs
  - simple backup or recovery artifacts when needed

## Active experiment iteration

- Expect repeated runs, parameter changes, and comparisons.
- Tighten logging, output naming, and run parameters enough to stay organized.
- Start separating repeated logic out of the main script before the code becomes hard to follow.
- Minimum expectations:
  - rerunnable entry path
  - clearer run parameters
  - lightweight logging or metric visibility
  - outputs that can be matched back to a run
- Escalation triggers:
  - repeated parameter sweeps
  - shared server runs
  - repeated copy-paste between scripts
  - growing difficulty understanding the main training path

## Mature comparison or repeatable workflow

- Stabilize run interfaces, saved metadata, naming, artifact layout, and resume behavior.
- Make experiment outputs easier to compare across runs.
- Only now does stronger consistency around checkpoint and artifact formats usually pay off.
- Minimum expectations:
  - stable run interface
  - clearer saved metadata
  - comparison-friendly output layout
  - more deliberate checkpoint or artifact rules

## General rule

- Do not design for the final stage while the project is still in the first one.
- Keep the current stage clean enough that the next stage is easy to reach.
- Infer the stage from the user's request and recent workflow first. Use confirmation only when the stage is genuinely unclear and the recommendation would change.
