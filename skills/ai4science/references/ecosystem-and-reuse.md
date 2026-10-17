# Ecosystem and Reuse

## Find and Apply Relevant Experience

- When choosing a scientific method, setting up a domain tool, or implementing
  nontrivial calculations, actively look for how comparable tasks are already
  solved. Search official documentation and application examples, relevant
  papers and author implementations, technical blogs, and open-source projects.
- Use GitHub Issues and Discussions, maintainer explanations, and official
  communities to learn practical behavior and established solutions. Match
  version-sensitive advice to the active environment; check important claims
  against primary sources or the implementation that governs the behavior.
- Compare the method's assumptions, data needs, outputs, practical constraints,
  and integration costs with the current task. Adopt the useful method, script,
  or pattern and explain material adaptations; popularity alone is not a reason
  to select a model or introduce a dependency.
- For training loops, evaluation, metrics, logging, checkpoints, and experiment
  utilities, look for mature framework-native support and good open-source
  examples. Use them to improve local organization and implementation. Briefly
  document non-obvious borrowed techniques and preserve required attribution.
- Search to resolve the current method or implementation choice. Reuse reliable
  sources already available, broaden the search when a consequential gap remains,
  and proceed when the evidence supports a suitable approach. Read relevant
  sections of references rather than importing their full workflows as obligations.

## Domain and Tool Entry Points

Use these when the corresponding tools are involved; apply the same approach
in other domains.

- **FDTD / Lumerical:** [Ansys Optics](https://optics.ansys.com/hc/en-us) provides
  documentation, application examples, scripts, analysis tools, and links to
  courses and support discussions. Look for complete examples of the relevant
  modeling and postprocessing task, not only individual API commands. For
  unit-cell libraries and device assembly, the
  [small-scale metalens example](https://optics.ansys.com/hc/en-us/articles/360042097313-Small-Scale-Metalens-Field-Propagation)
  is one useful starting point for comparing workflows and reusing scripts.
- **PyTorch:** use the [official tutorials](https://docs.pytorch.org/tutorials/),
  documentation for the active version, [PyTorch Discuss](https://discuss.pytorch.org/),
  and relevant GitHub repositories, issues, and maintainer discussions. Consult
  specialized documentation for framework mechanics and advanced runtime behavior.

## Prefer Mature Infrastructure First

- If the user explicitly specifies a framework, follow that choice.
- Otherwise reuse the project's existing mature framework-native infrastructure
  first. If the framework is still open and the task matches a typical modern
  deep-learning workflow, PyTorch can be the default first choice together with
  other mature open-source libraries where appropriate.
- Build custom NN plumbing only when existing tools fail a concrete requirement.
- Prefer borrowing a proven pattern over inventing a local abstraction for common
  experiment tasks. Keep training logic directly readable; an early experiment
  does not need a local mini-framework.

## Recommend Dependencies Within the Task

Suggest libraries or helpers when they solve a real problem. Explain their
benefits and costs, and let the user decide whether the dependency is worth
adopting. Follow the project's dependency mechanism and obtain approval for
environment changes. Finding a useful tool or a larger research workflow does
not expand the authorized work.
