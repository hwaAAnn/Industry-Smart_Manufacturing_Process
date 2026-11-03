# Industry-Manufacturing Process MES

An industry–academia project with a manufacturer on a continuous process: building the data-driven layer of a smart-factory **MES (Manufacturing Execution System)** that turns shop-floor process data into process monitoring, operating KPIs, and recommended operating set-points.

> **Disclosure** — This repository is an overview only. It contains no source code, data, trained models, or real variable names. Variables are shown as generic placeholders (`decision_feat_N` for controllable set-points, `context_feat_N` for monitored variables).

## Background

- The process has many **decision variables** that operators set directly, and many **context variables** that are only monitored.
- Operating performance is judged by **KPIs** aggregated over a time window from several process indicators.
- Some key indicators are measured offline and only sparsely, so they cannot be tracked in real time without a model.
- Goal: give operators a single view of the current process state and KPIs, and a recommendation of set-points that **improve a primary KPI while keeping the other KPIs within their bounds**.

## System overview

<p align="center">
  <img src="docs/pipeline.svg" alt="Abstract MES pipeline: process data and monitoring, operating KPIs, set-point recommendation" width="900">
</p>

| Layer | Role in the MES |
|-------|-----------------|
| **1. Process data & monitoring** | Monitored variables are forecast over a short horizon, and ML indicator models turn (context + decision) variables into process indicators that are otherwise only available from offline measurement. Deviations from the expected state are flagged. |
| **2. Operating KPIs** | Observed history and the forecast horizon are combined and aggregated over the KPI window, giving KPIs that are available during operation rather than after the fact. |
| **3. Set-point recommendation** | A metaheuristic search over the decision variables proposes set-points that improve the primary KPI under KPI bounds; the candidates are re-evaluated through the indicator models before being returned to the operator. |

## My contributions

### Process modeling for monitoring and KPIs
- Built multivariate forecasting models for process indicators that are measured offline and therefore only sparsely labeled, while the process signals themselves are dense.
- Addressed label scarcity with a semi-supervised scheme: a confidence-filtered, current-state model supplies extra training targets for the longer-horizon forecaster.
- Compared several modeling approaches for the forecasting task.

### Decision support for operating set-points
- Formulated set-point selection as a constrained search on top of the ML models (objective, KPI constraints, decision-variable bounds).
- Chose **Particle Swarm Optimization**: all decision variables are continuous, which suits PSO's velocity-based update better than discrete-oriented methods such as GA.
- Designed a **KNN warm start**: the swarm is initialized with the best-performing set-points found among past process states most similar to the current one, so recommendations stay in a realistic, well-supported region.
- Handled constraints with a relative-violation penalty, velocity clamping, and a small mutation rate to avoid premature convergence.
- Also explored a **surrogate-assisted** alternative: distilling the indicator models into quadratic surrogates, conditioning them on the current process state, and solving the reduced problem with gradient-based optimization.

## Related

- Algorithm implementations and surrogate-assisted optimization (PSO / GA / Tabu Search / Simulated Annealing / quadratic-surrogate NLP) on a synthetic problem: see the `optbench` repository.
