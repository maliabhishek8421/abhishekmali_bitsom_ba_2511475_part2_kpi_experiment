# Hypothesis Test Notes

## Metric Being Tested
**Paid conversion rate** (`converted_to_paid`) — the North Star metric for this experiment.

### Reason for choosing this metric
Paid conversion is the metric closest to the actual business decision: whether the new onboarding/activation campaign drives more users to become paying customers. Funnel metrics earlier in the journey (landing page visits, trial starts, onboarding completion) are useful diagnostics, but they are *supporting* metrics — a campaign could lift visits or trial starts without ever lifting revenue-generating conversions. Testing conversion directly avoids drawing a launch conclusion from a metric that doesn't pay the bills.

## Hypotheses

- **Null hypothesis (H₀):** There is no difference in paid conversion rate between the Control group (existing onboarding) and the Treatment group (new campaign). `p_treatment = p_control`.
- **Alternate hypothesis (H₁):** The Treatment group has a higher paid conversion rate than the Control group. `p_treatment > p_control`.

## Test Type
- **One-tailed test.** The business question is specifically "does the new campaign *improve* conversion," not "does it differ in either direction." A campaign launch decision only cares about improvement; a one-tailed test is the more precise framing of that exact question and gives more power to detect the improvement we actually care about.
- **Test used:** Two-proportion z-test (large-sample, independent groups, binary outcome).
- **Significance level (α):** 0.05.

## Test Inputs (post-cleaning, deduplicated dataset, n = 1,400)

| Group | Users (n) | Converted (x) | Conversion rate |
|---|---|---|---|
| Control | 690 | 22 | 3.19% |
| Treatment | 710 | 50 | 7.04% |

- Pooled proportion: 5.14%
- Standard error: 0.0118
- **Z-statistic: 3.26**
- **P-value (one-tailed): 0.00055**
- 95% CI for the difference in proportions: [1.56pp, 6.15pp]

## Decision Rule
Reject H₀ if p-value < α (0.05).

## Result
P-value (0.00055) is far below α (0.05) → **reject the null hypothesis.**

The Treatment group's paid conversion rate (7.04%) is statistically significantly higher than Control (3.19%), an absolute lift of +3.85 percentage points and a relative lift of approximately +121%. This is not a result attributable to random chance at any conventional significance level.

## Business Interpretation
On the primary success metric alone, the new onboarding and activation campaign clearly outperforms the existing experience — it more than doubles the rate at which signups become paying customers, with high statistical confidence. **However, this result by itself is not sufficient to recommend a full launch.** The conversion lift must be weighed against the guardrail metrics (refund rate, support ticket rate, and revenue quality per converted user), which show concerning movement in the same Treatment group. See `outputs/recommendation_memo.md` for the full guardrail evaluation and final recommendation, which is connected directly to this test result: the conversion question is answered "yes, it improves," but the *launch* question depends on whether the guardrail risk is acceptable.

## Evidence
Test inputs, formulas, and output are captured in `screenshots/hypothesis_test_output.png` and computed live in `outputs/experiment_summary.xlsx` (Summary Metrics sheet, "Paid conversion rate" row).
