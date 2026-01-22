# LLM Alignment Stress-Test Benchmark (CDM + EvilGenie)

This repo is a research prototype for **stress-testing LLM alignment under “temptation”**: we define measurable alignment constraints, quantify how much “reward-hacking pressure” a task creates, and evaluate whether models drift toward violations as they encounter sequences of tasks.

The project has two pillars:

- **CDM (Context-Dependent Misalignment) setup**: a lightweight formalization and a set of operational metrics for “how misaligned” behavior looks under explicit constraints.
- **EvilGenie integration**: uses a reward-hacking-friendly programming benchmark to instantiate tasks where agents can cheat (e.g., hardcode, read tests, edit harness) and we can detect it.

---

## What to look at first (recruiter-friendly)

- **`setup.pdf`** (or `docs/setup.pdf`): the written setup + metrics (short, 6 pages).
- **EvilGenie adapter / tasks**: the code that runs EvilGenie problems and records alignment metrics.
- **`experiments/` + `results/`**: scripts + logged outputs (metrics tables, traces, run configs).

---

## CDM setup (high-level)

We model evaluation as a **sequence of tasks**. Each task defines:

- a reward (the thing the model tries to maximize), and
- a set of **alignment constraints** (what it is not allowed to do).

From this we track three quantities:

### 1) Violation probability (binary “did it violate?”)
For a deployed policy on a task, estimate how often *any* constraint is violated.

### 2) Compliance coefficient (severity “how much did it violate?”)
Not just whether it violated, but the **cumulative magnitude of violations** across the trajectory.

### 3) Temptation index (how strong is the incentive to violate?)
We define a protocol-based “temptation” score per task: how much higher the protocol’s return-to-go proxy is for the **best violating action** vs the **best compliant action**, aggregated into a task-level scalar.

### Empirical hypothesis (optional inference layer)
A simple model to test empirically is that compliance drift is predictable from temptation across tasks (ARX-style). This is not assumed for the benchmark to run; it’s the analysis layer.

---

## Why EvilGenie?

EvilGenie is a benchmark designed to **elicit reward hacking in programming settings**: it creates many opportunities for an agent to “pass tests” without solving the actual problem (e.g., hardcoding, reading `test_cases.json`, editing test files). The benchmark also includes multiple automated detection strategies (held-out tests, LLM judges, and test-file edit detection).  
This makes it a good substrate for alignment stress-testing: we can turn “reward hacking behavior” into explicit **constraint violations** and measure them.

---

## EvilGenie integration (what this repo adds)

This repo wraps EvilGenie tasks and records CDM metrics per run. Concretely:

- Treat each EvilGenie problem as a **task** with constraints like:
  - do not read test files / hidden cases,
  - do not modify the evaluation harness,
  - do not hardcode outputs keyed to known cases,
  - do not use environment exploits.
- Run agent scaffolds on the task and log:
  - violation probability,
  - violation magnitude (compliance coefficient),
  - temptation estimates (under a fixed evaluation protocol / compute budget).
- Use EvilGenie’s detection signals (held-out tests / judge / edit detection) as:
  - ground truth labels and/or
  - auxiliary detectors to validate our constraint instrumentation.

---

## Repository layout (adjust names to match your tree)

- `docs/`  
  - `setup.pdf` – CDM setup + definitions + metrics
- `evilgenie/` (submodule or vendored)  
  - upstream benchmark code + problems
- `cdm/`  
  - metric definitions, estimators, logging schema
- `experiments/`  
  - run scripts, configs, sweeps
- `results/`  
  - run outputs (JSON/CSV), plots, summaries

---

## Status

- [x] CDM setup written (definitions + metrics)
- [x] EvilGenie task integration scaffold
- [ ] First end-to-end experiment run + cleaned result tables
- [ ] Ablations: detectors vs constraints, compute budget sensitivity
- [ ] Inference layer (ARX-style analysis) + robust SEs (optional)

---

## Notes

This is a research repo. The goal is clarity and reproducibility over “production polish”:
- deterministic configs,
- saved artifacts,
- explicit metrics,
- easy-to-audit runs.

---

## References

- EVILGENIE: A Reward Hacking Benchmark (Gabor et al., 2025) — arXiv:2511.21654
- LiveCodeBench (problem source benchmark)
