---
name: statistical-analysis
description: Hypothesis testing, confidence intervals, and A/B testing basics.
---
# statistical-analysis.md

## Purpose

Define conventions for statistical analysis — covering hypothesis testing,
confidence intervals, A/B testing, significance interpretation, effect size
measurement, and common pitfalls to avoid in statistical reasoning.

---

## Conventions

### Statistical Analysis Workflow

```
1. Define the Question     → What are we trying to learn?
2. State Hypotheses        → H₀ (null) and H₁ (alternative)
3. Choose the Test         → Based on data type and assumptions
4. Check Assumptions       → Normality, independence, sample size
5. Run the Test            → Calculate test statistic and p-value
6. Interpret Results       → Effect size + confidence interval + p-value
7. Document Conclusion     → Plain-language finding with caveats
```

### Test Selection Guide

| Question | Data Type | Recommended Test |
|---|---|---|
| Is the mean different from a value? | Continuous, 1 group | One-sample t-test |
| Are two group means different? | Continuous, 2 groups | Independent t-test (or Mann-Whitney) |
| Paired before/after comparison? | Continuous, paired | Paired t-test (or Wilcoxon) |
| Are 3+ group means different? | Continuous, 3+ groups | ANOVA (or Kruskal-Wallis) |
| Are two categories related? | Categorical, 2 vars | Chi-square test |
| Is there a linear relationship? | Continuous, 2 vars | Pearson correlation |
| Is there a monotonic relationship? | Ordinal/non-linear | Spearman correlation |

### Hypothesis Testing Pattern

```python
from scipy import stats

def run_hypothesis_test(
    group_a: pd.Series,
    group_b: pd.Series,
    alpha: float = 0.05,
    test: str = "t_test",
) -> dict:
    """Run hypothesis test with full result context."""
    # Check assumptions
    _, normality_a = stats.shapiro(group_a.sample(min(len(group_a), 5000)))
    _, normality_b = stats.shapiro(group_b.sample(min(len(group_b), 5000)))
    is_normal = normality_a > 0.05 and normality_b > 0.05

    # Select test based on assumptions
    if is_normal or len(group_a) > 30:
        stat, p_value = stats.ttest_ind(group_a, group_b, equal_var=False)
        test_used = "Welch's t-test"
    else:
        stat, p_value = stats.mannwhitneyu(group_a, group_b, alternative="two-sided")
        test_used = "Mann-Whitney U"

    # Effect size (Cohen's d)
    pooled_std = ((group_a.std()**2 + group_b.std()**2) / 2) ** 0.5
    cohens_d = (group_a.mean() - group_b.mean()) / pooled_std

    return {
        "test_used": test_used,
        "statistic": round(stat, 4),
        "p_value": round(p_value, 4),
        "significant": p_value < alpha,
        "alpha": alpha,
        "effect_size_d": round(cohens_d, 3),
        "group_a_mean": round(group_a.mean(), 4),
        "group_b_mean": round(group_b.mean(), 4),
        "group_a_n": len(group_a),
        "group_b_n": len(group_b),
    }
```

### Confidence Intervals

Always report confidence intervals alongside p-values:

```python
def confidence_interval(data: pd.Series, confidence: float = 0.95) -> tuple:
    """Calculate confidence interval for the mean."""
    n = len(data)
    mean = data.mean()
    se = stats.sem(data)
    ci = stats.t.interval(confidence, df=n-1, loc=mean, scale=se)
    return {"mean": round(mean, 4), "ci_lower": round(ci[0], 4), "ci_upper": round(ci[1], 4), "confidence": confidence}
```

### A/B Testing Framework

```python
def ab_test_analysis(
    control: pd.Series,
    treatment: pd.Series,
    metric_name: str,
    alpha: float = 0.05,
) -> dict:
    """Complete A/B test analysis with guardrails."""
    # Sample size check
    min_sample = 100  # Minimum per group
    if len(control) < min_sample or len(treatment) < min_sample:
        return {"error": f"Insufficient sample size. Need ≥{min_sample} per group."}

    # Core analysis
    result = run_hypothesis_test(control, treatment, alpha=alpha)

    # Practical significance (minimum detectable effect)
    lift_pct = ((treatment.mean() - control.mean()) / control.mean()) * 100

    result.update({
        "metric": metric_name,
        "lift_pct": round(lift_pct, 2),
        "practically_significant": abs(lift_pct) > 2.0,  # 2% minimum lift
        "recommendation": _ab_recommendation(result, lift_pct),
    })
    return result


def _ab_recommendation(result: dict, lift_pct: float) -> str:
    if not result["significant"]:
        return "No statistically significant difference. Keep control."
    if abs(lift_pct) < 2.0:
        return "Statistically significant but lift too small (<2%). Keep control."
    if lift_pct > 0:
        return f"Ship treatment. +{lift_pct:.1f}% lift is significant and practical."
    return f"Revert treatment. {lift_pct:.1f}% negative impact detected."
```

### Effect Size Interpretation

| Cohen's d | Interpretation | Practical Impact |
|---|---|---|
| < 0.2 | Negligible | Likely not worth acting on |
| 0.2–0.5 | Small | May matter at scale |
| 0.5–0.8 | Medium | Likely meaningful |
| > 0.8 | Large | Clearly significant |

### Result Reporting Template

```markdown
### A/B Test Result: [Experiment Name]

**Metric:** Average Order Value
**Period:** Jan 1–31, 2026
**Groups:** Control (n=5,234) vs Treatment (n=5,198)

| Metric | Control | Treatment | Lift | p-value |
|---|---|---|---|---|
| AOV | €98.50 | €104.20 | +5.8% | 0.003 |

**Effect Size:** Cohen's d = 0.41 (small–medium)
**95% CI for lift:** [+2.1%, +9.5%]

**Conclusion:** Treatment shows a statistically significant and practically
meaningful increase in AOV (+5.8%, p=0.003). Recommend shipping.

**Caveats:**
- Test ran during a promotional period — may not generalize
- Mobile segment not yet analyzed separately
```

---

## Anti-Patterns

- Never report p-values without effect size — statistical significance ≠ practical significance
- Never run A/B tests with fewer than 100 samples per group — results are unreliable
- Never peek at results mid-experiment — early stopping inflates false positive rate
- Never ignore multiple comparison correction — testing 10 metrics requires Bonferroni
- Never assume normality without testing — use non-parametric alternatives when violated
- Never report "the test failed" — there is no failure, only "no significant difference found"
- Never use p=0.05 as a magic threshold — report exact p-values and let context decide

---

## Cross-References

- **data-analytics/data-exploration** → EDA before formal hypothesis testing
- **data-analytics/metrics-and-kpis** → metrics being tested in A/B experiments
- **data-analytics/reporting** → statistical results in report findings
- **data-analytics/visualization** → charts for distributions and confidence intervals

---

## Ready-to-Use Prompt

```
Task: Perform [statistical analysis type] on [dataset/experiment]
Skill: data-analytics/statistical-analysis, data-analytics/data-exploration

REQUIREMENTS:
- Question: [what are we trying to learn]
- Data: [describe groups, variables, sample sizes]
- Test type: [t-test / chi-square / ANOVA / A/B test]

CONSTRAINTS:
- Follow 7-step workflow
- Check assumptions before choosing test
- Report effect size alongside p-value
- Include confidence intervals
- Use result reporting template for output
- Plain-language conclusion with caveats

DONE WHEN:
- Hypotheses stated clearly
- Assumptions checked and documented
- Test run with full result context
- Effect size and CI reported
- Plain-language conclusion written
- Code review skill applied before commit
```

