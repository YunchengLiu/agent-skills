# Scientific Reasoning

- [Design with available information](#design-with-available-information)
- [Judge results and tradeoffs by their use](#judge-results-and-tradeoffs-by-their-use)
- [Choose useful further work](#choose-useful-further-work)
- [Read for the current decision](#read-for-the-current-decision)

## Design with Available Information

- Identify the calculation or decision needed and the theories, formulas,
  solvers, empirical models, and project results available to support it.
  Retain the sources, settings, units, and conventions needed to use them.
  Revisit established support when changed conditions, an implementation
  discrepancy, or new evidence could affect the current result.
- Treat scarce data, uneven coverage, and costly acquisition as design inputs.
  Inspect how observations were obtained, what ranges or conditions they cover,
  and which observations share an experiment or simulation. Use this information
  to choose the representation, model capacity, learning target, and evaluation.
- Use meaningful variables, relations, and applicable domain knowledge to
  express the problem before choosing a model. When an existing model captures
  the main behavior, parameter calibration or learning a residual may be useful;
  choose such combinations for their fit to the task, not as a required recipe.
- Relate uneven coverage to the intended use: important input regions, operating
  conditions, and continuous response ranges can matter as well as class counts.
  Choose weighting, sampling, or regional evaluation when it improves that use;
  interpret aggregate metrics alongside behavior in the regions that matter.
  Match the evaluation to the question rather than assuming a uniform target
  distribution. Preserve the agreed split and holdout rules when adapting it.
- Check a reused model's data demands, assumptions, inputs, objective, and
  evaluation against the actual problem. Reuse reliable implementations while
  adapting the parts that need to reflect the available information. Combine
  physical, empirical, learned, and calibration components when their roles help;
  judge task performance and explain the dominant behavior and key choices.
  Complete interpretability of every component is not a prerequisite for
  establishing task performance.
- In simulation-supported experimental work, let controllable parameters,
  realizable scales, and observable quantities shape both the design and its
  evaluation. Use simulation for the questions it can answer and experimental
  observations to calibrate or assess relevant differences. Use the process
  knowledge already available; add measurements to resolve specific decisions
  rather than requiring a complete calibration campaign for every task.
- Complete the analysis and recommendation supported by current information.
  Localize unresolved parts to the quantities or choices they affect. When more
  data would help, identify the question, useful acquisition region, and expected
  benefit relative to cost; keep acquisition within the authorized scope.

## Judge Results and Tradeoffs by Their Use

### Requirements and design preferences

- Separate fixed requirements, optimization preferences, and ideal references.
  Derive acceptance limits from the user's requirements, project agreements,
  or applicable method conditions. Treat "high," "uniform," or "close to ideal"
  as goals to balance; a metric or ideal value alone is not a pass/fail rule.
- Compare available feasible choices by the intended outcome, main features,
  relevant metrics, practical realization, and cost. Make a reasoned recommendation
  when several choices are acceptable. Qualitative evidence, domain experience,
  and meaningful scale comparisons can support it without a unique optimum or
  a numerical threshold for every preference.
- Match evidence to the claim: relevant observations for a feature or trend,
  comparable outcomes for an improvement, and discriminating evidence for a
  mechanism. Reserve proof obligations for theoretical results actually asserted.
  For one specified material or device, state its conditions and report the
  supported result. Study transfer when it is part of the question; untested
  transfer does not establish that the method is restricted to that one case.

For example, when asked to select eight phase levels from an existing unit-cell
scan, use the agreed normalization across the candidate set and compare high,
reasonably uniform amplitude with reasonably even phase coverage over one cycle.
Select a suitable combination on the available realizable parameter grid and
report its actual responses and tradeoffs. Imperfect coverage is a design
compromise, not by itself a reason to reject the library using a newly invented
percentage cutoff. Propose additional scans separately when they are worthwhile.

### Numerical correctness and useful accuracy

- Implement the chosen formulas and numerical method correctly. For numerical
  method or solver development, verify the relevant stability, convergence, and
  accuracy expected of that method. For candidate selection or applied analysis,
  evaluate the fidelity needed for the requested features, trends, rankings, or
  predictions. Choose depth by the task, not by an exemption for a named domain.
- Judge discrepancies by their effect on the result of interest. Where useful,
  convert them to physical scales or dimensionless quantities and compare with
  relevant resolution, uncertainty, or decision margins. Use existing results,
  domain experience, and scale reasoning to assess significance; establish exact
  sensitivity only when the task needs it. Intermediate last-digit differences
  do not on their own justify a repair.
- Distinguish arithmetic and discretization error from model, solver-method,
  and input limitations. Use established settings and reliable conventional
  arithmetic; prioritize errors that matter to the result. Local refinement
  cannot by itself remove model discrepancy or uncertain inputs. A tighter
  tolerance or special numerical treatment needs a credible basis and benefit,
  not an arbitrary decimal target or an assumed exact physical truth.
- Use known fabrication, experimental, or deployment limits when choosing
  parameters and judging nominal improvements. Keep realizable settings separate
  from the precision used internally to calculate their responses. Prefer gains
  that can survive practical realization; extra parameter digits below the
  meaningful control scale are not useful design progress by themselves.
- Choose normalization for the research target and keep its meaning consistent
  across comparisons. Preserve relative amplitudes when comparing strength;
  use suitable normalized quantities for patterns or trends. Absolute calibration
  is needed when the task depends on absolute values, not for every comparison.
- For an accuracy question that matters to the task, first use existing references,
  validation, and error information. If these leave a consequential question
  unresolved, choose a representative comparison or stability/convergence check
  that can answer it. Missing ground truth alone does not require a new test.
  Finish when the available evidence supports the intended use; reopen the
  relevant check for changed conditions or observed instability that affects it.

## Choose Useful Further Work

- Start from the current deliverable, the evidence already available, and any
  question still affecting it. Use inspection and reasoning to compare removal,
  reuse, or simpler alternatives before scheduling new runs. Choose additional
  work for a necessary check, unresolved decision, or authorized improvement
  whose likely benefit warrants compute, data, and implementation costs.
- For an improvement, compare with a suitable baseline under comparable
  conditions. For a key design choice or mechanism, choose a small set of
  informative controls that distinguish the relevant contribution or a credible
  competing explanation. Briefly explain what each planned comparison resolves;
  possible combinations do not constitute a required experiment matrix.
- Evaluate interacting modules through meaningful replacements or system-level
  comparisons when isolated removal would break the method or change the
  question. Validate the combined contribution at the level claimed; independent
  usefulness of every component is not required.
- For a correction step, compare the baseline and corrected target observations;
  add a targeted control when an important competing explanation remains.
  Established solvers and preprocessing can remain the common basis of that
  comparison. Extend checks for meaningful unresolved results, then report the
  outcome when the current question is answered.

## Read for the Current Decision

Use these as starting points when the corresponding choice needs support. Read
relevant sections and seek suitable implementations or follow-up work. Apply
the useful ideas in the current context; the sources' full workflows, benchmark
targets, and model choices are not additional project requirements.

- **Limited data with useful scientific knowledge:** Karpatne et al.,
  [Theory-guided Data Science](https://arxiv.org/abs/1612.08544) (2017).
  Consider how known relationships can shape representation and learning.
- **Small-data function regression:** Rasmussen and Williams,
  [Gaussian Processes for Machine Learning](https://gaussianprocess.org/gpml/chapters/)
  (2006), especially regression, covariance functions, and model selection.
  Consider priors, similarity, and uncertainty when evaluating a regression method.
- **Uneven continuous targets:** Yang et al.,
  [Delving into Deep Imbalanced Regression](https://proceedings.mlr.press/v139/yang21m.html)
  (ICML 2021). Consider target neighborhoods and label/feature distribution
  smoothing; choose the evaluation distribution for the actual research use.
- **Simulation-to-experiment calibration:** Kennedy and O'Hagan,
  [Bayesian Calibration of Computer Models](https://doi.org/10.1111/1467-9868.00294)
  (2001). Distinguish parameter calibration from model discrepancy when deciding
  what the available observations can correct.
- **Practical neural-network training:** Karpathy,
  [A Recipe for Training Neural Networks](https://karpathy.github.io/2019/04/25/recipe/)
  (2019). Use its data inspection and simple-baseline practices at the depth
  needed for the current training task.
