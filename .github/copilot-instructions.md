# LLM Judge Metrics — Copilot Guide

## What This Project Is
- Evaluate LLM-as-judge reliability via three signals: Quadratic Kappa (agreement), Kendall's Tau (ranking), Mean Bias (calibration). Notebook-first; no CLI, package entrypoint, or tests.
- Decisions hinge on Kappa ≥ 0.7 and |Bias| ≤ 0.3; Tau ≥ 0.6 is used for ranking confidence.

## Key Artifacts
- [demo.ipynb](../demo.ipynb) shows happy-path and failure cases on a 20-sample 1–5 score array; keep outputs lightweight/clearable.
- [README.md](../README.md) captures the production readiness rule and brief metric explanations.
- Planned: [metrics.py](../metrics.py) for reusable stateless metric helpers; [sample_scores.csv](../sample_scores.csv) for multi-rater data.

## Data & Metric Conventions
- Inputs are aligned 1D NumPy arrays of integer scores on a 1–5 scale; clean/align before computing.
- Missing ratings should be `np.nan` (especially for future Krippendorff alpha work).
- Bias is computed as `np.mean(autograder - human)`; positive = lenient, negative = harsh.
- Use `cohens_kappa_score(..., weights="quadratic")` and `kendalltau(human, autograder)`; Tau returns `(tau, p_value)`.

## Dependencies & Setup
- Minimal stack: numpy, scikit-learn, scipy; avoid pulling pandas or heavier deps unless justified.
- Quick start:
	```bash
	pip install numpy scikit-learn scipy
	jupyter notebook .
	# Run demo.ipynb
	```

## Coding Patterns
- Favor pure, stateless functions; separate I/O/reporting from metric math.
- Keep notebook examples concise; prefer adding example cells over new scripts unless reuse demands it.
- If adding metrics to metrics.py, organize sections Agreement → Correlation → Bias/Error → Reporting; return simple floats or `(value, p_value)` tuples.
- CSV inputs should use headers `human_score, autograder_a, autograder_b, ...` for compatibility.

## Working Style
- Notebook outputs are transient; clear before committing if added.
- No packaging or CLI scaffolding expected; keep structure small and exploratory.
- When extending, show “good” vs “bad” autograder scenarios to illustrate metric behavior.
