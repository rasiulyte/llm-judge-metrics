# Autograder Validation Toolkit

A practical notebook for validating LLM-as-judge systems against human evaluations.

## Overview

When deploying AI assistants at scale, organizations rely on autograders—LLMs that evaluate other LLM outputs—to maintain quality standards. This toolkit provides the statistical metrics and workflows needed to validate autograder reliability before production deployment.

## What's Included

| Section | Purpose |
|---------|---------|
| **Metrics Primer** | Cohen's Kappa, Kendall's Tau, Spearman's Rho, Pearson's R, Mean Bias |
| **Domain-Specific Thresholds** | Graduated thresholds for general chat, user-facing support, and safety-critical applications |
| **Rubric & Prompt Design** | Template rubric and evaluation prompt with guidance on rubric drift |
| **Stratified Analysis** | Slice-level validation by surface, risk level, or any categorical dimension |
| **Threshold Calibration** | Confusion matrix analysis at different pass/fail operating points |
| **Ranking Validation** | Best-of-N selection win-rate for model comparison tasks |
| **Safety Grading** | Separate safety dimension with false-negative-focused analysis |
| **Failure Mode Catalog** | Constant output, systematic leniency/harshness, random scoring |

## Quick Start

Open `autograder_validation.ipynb` and run the cells, or copy the core function:

```python
import numpy as np
from sklearn.metrics import cohen_kappa_score
from scipy.stats import kendalltau, spearmanr, pearsonr

human_scores = np.array([5, 4, 3, 5, 2, 4, 3, 5, 4, 3])
auto_scores  = np.array([5, 4, 3, 4, 2, 4, 3, 5, 4, 3])

# Run the evaluate_autograder() function defined in the notebook
results = evaluate_autograder(human_scores, auto_scores, "My Autograder")
```

## Key Metrics at a Glance

| Metric | What It Measures | When to Use |
|--------|------------------|-------------|
| **Quadratic Kappa** | Agreement beyond chance | Primary validation metric |
| **Kendall's Tau** | Rank correlation | A/B testing, model comparison |
| **Mean Bias** | Systematic over/under-scoring | Threshold calibration |
| **MAE** | Average scoring error magnitude | General accuracy |

## Recommended Thresholds

| Scenario | Kappa | \|Bias\| |
|----------|-------|---------|
| Internal search/chat | ≥ 0.70 | ≤ 0.30 |
| User-facing support | ≥ 0.80 | ≤ 0.20 |
| Safety/compliance gates | ≥ 0.85 | ≤ 0.10 |

These are starting points—tune empirically for your domain.

## Sample Size Guidelines

| Purpose | Minimum N |
|---------|-----------|
| Initial validation | 50 |
| Production approval | 100 |
| High-stakes deployment | 200+ |
| Per-slice validation | 30 per slice |

The notebook uses N=20 for illustration. At this size, confidence intervals are wide (±0.15–0.20 for Kappa/Tau).

## When to Run Validation

- **Launch readiness** — Before deploying a new autograder
- **Post-prompt change** — After modifying autograder prompts or rubrics
- **Model upgrade** — When the underlying LLM is updated
- **Safety red-team refresh** — After adversarial testing reveals new failure modes
- **Quarterly audit** — Detect drift over time

## File-Based Workflow

The notebook includes a `run_eval()` function for CSV-based workflows:

```python
# Defined in the notebook - loads CSVs and runs evaluation
results = run_eval(
    'human_scores.csv',
    'autograder_scores.csv',
    id_col='response_id',
    score_col='quality_score'
)
```

## Export for CI/CD

The notebook includes an `export_results()` function for logging:

```python
# Defined in the notebook - exports metrics as JSON
json_summary = export_results(results, 'eval_results.json')
```

Output:
```json
{
  "kappa_quadratic": 0.881,
  "kendall_tau": 0.831,
  "mean_bias": 0.15,
  "approved": true,
  "n": 100
}
```

## Interpreting Results

### If Approved
Document thresholds, archive the validation dataset, schedule periodic revalidation.

### If Failed on Kappa
1. Check human-human Kappa first (establishes ceiling)
2. Review autograder prompt for clarity
3. Check rubric for ambiguous criteria
4. Analyze disagreements by slice

### If Failed on Bias
- **Lenient:** Add examples of low-quality responses to prompt
- **Harsh:** Add examples of acceptable responses to prompt

### If Failed on Tau
1. Check for score compression (all 3s and 4s)
2. Verify ranking examples in prompt
3. Consider pairwise comparison approach

## Dependencies

```
numpy
scipy
scikit-learn
```

## License

MIT

## Contributing

Issues and PRs welcome. For major changes, please open an issue first to discuss.
