# Part 2 — KPI Framework, Business Experiment Analysis & Decision Recommendation

## Business Context

A subscription-based digital product company ran an A/B experiment on a new onboarding and activation campaign, aimed at improving user conversion and early engagement. Users were split into:
- **Control** — existing onboarding experience
- **Treatment** — new campaign experience

**Decision needed:** Whether to launch the Treatment experience to all users.
**Who it impacts:** All future signups (product/growth team owns the rollout decision; finance and support teams are affected by downstream revenue and ticket volume).
**Metric that should improve:** Paid conversion rate (signup → paying customer).
**Risks to monitor:** Refund rate, support ticket volume, and revenue quality per converted user — a campaign that converts more users at the cost of more refunds, more support load, or lower-value customers is not an unambiguous win.
**Evidence required before recommending:** A statistically valid comparison of Control vs Treatment on the primary metric, plus a check of guardrail metrics and segment-level consistency, before any launch recommendation is made.

This problem framing is summarized in `outputs/recommendation_memo.md`.

## Dataset Description

- **File:** `data/campaign_experiment_data.xlsx`, sheet `experiment_data`
- **1,408 raw user-level records** (1,400 after removing 8 exact duplicate rows), covering signup date, experiment group, region/device/traffic source/plan segments, funnel events (landing page visit, trial start, onboarding completion, paid conversion), revenue, support tickets, refunds, days to convert, and an engagement score.
- A second sheet, `business_context`, defines each field and explicitly flags the guardrail caution: *"A better conversion rate may still be risky if refunds, support tickets, or low-quality revenue increase."*

## North Star Metric Selected
**Paid conversion rate.** Full justification (why this over supporting metrics, connection to growth, and blind-optimization risk) is in `outputs/recommendation_memo.md`.

## KPI Tree Summary
`outputs/kpi_tree.png` (preview also at `screenshots/kpi_tree_preview.png`) shows:
- **North Star:** Paid conversion rate
- **3 primary drivers:** Landing page engagement, Trial activation, Onboarding completion — each with 2 sub-drivers
- **3 guardrail metrics:** Refund rate, Support ticket rate, Revenue quality (ARPU per converted user)

## Experiment Analysis Approach

Performed in `analysis/experiment_analysis.xlsx`:

| Check | Finding | Handling |
|---|---|---|
| Missing values | `device_type` (18), `traffic_source` (24) missing in raw data | Left as-is in clean_data; excluded from segment breakdowns where the field is the grouping key, since imputing a channel/device would fabricate data |
| Group counts | Control 693 / Treatment 715 (raw); 690 / 710 after dedup | Balanced, no action needed |
| Duplicate user IDs | 8 exact duplicate rows (full row re-exports) | Removed, kept first occurrence |
| Invalid binary values | All of `visited_landing_page`, `started_trial`, `completed_onboarding`, `converted_to_paid`, `refund_requested` contain only 0/1 | No invalid values found; verified, not corrected |
| Outliers in revenue | Revenue is 0 for non-converters (expected) and ranges $218–$8,610 for the 72 converters; IQR method flags a handful of high-value outliers | Retained — these are real high-value conversions, not data errors |
| Segment distribution across groups | Region, device type, traffic source, and plan type are balanced within ~5pp between Control and Treatment | Confirms valid randomization; no rebalancing needed |

## Experiment Summary
`outputs/experiment_summary.xlsx` compares Control vs Treatment on all 11 required metrics (Summary Metrics sheet) plus a Segment Breakdown sheet covering conversion rate and support ticket rate by region, device type, traffic source, and plan type. All values are live Excel formulas referencing the cleaned dataset (`raw_data` sheet within the same workbook).

Headline result: **Paid conversion rate roughly doubled** (3.2% → 7.0%), but **support ticket rate also rose sharply** (14.8% → 24.8%) and **revenue per converted user fell by half** ($1,630 → $770).

## Hypothesis Test Summary
Two-proportion z-test on paid conversion rate (one-tailed, α = 0.05): **Z = 3.26, p = 0.00055 → reject H₀.** The conversion lift is statistically significant. Full notes in `analysis/hypothesis_test_notes.md`; evidence in `screenshots/hypothesis_test_output.png` and the `Hypothesis Test` sheet of `outputs/experiment_summary.xlsx`.

## Guardrail Metrics Considered
1. **Support ticket rate** — +10.0pp (statistically significant, p < 0.0001, consistent across every segment). **Primary risk.**
2. **Refund rate** — 0.0% (Control) vs 0.4% (Treatment); only 3 refunds, all in Treatment (marginal, p ≈ 0.09). **Moderate risk, monitor.**
3. **Revenue quality (ARPU per converted user)** — down ~53% in Treatment (marginal, p ≈ 0.08, but consistent with median revenue). **Risk to revenue quality.**

Full guardrail analysis and segment-level detail in `outputs/recommendation_memo.md`.

## Final Recommendation
**Launch only for selected segments** — Free/Premium plan users via Organic, Paid Search, and Referral traffic, where conversion lift is strong and the guardrail trade-off is most favorable. Hold off on Social traffic (the only segment with negative lift) and Basic plan users (negligible lift) pending root-cause investigation of the support ticket increase. Full reasoning in `outputs/recommendation_memo.md`.

## Assumptions and Limitations
- Missing `device_type`/`traffic_source` values were left blank rather than imputed, to avoid fabricating segment membership.
- Revenue, days-to-convert, and refund/revenue-quality comparisons are based on a relatively small converter sample (72 total), so those specific findings are directionally strong but only marginally significant — treated as real risk signals, not settled fact.
- The analysis window is 30 days; longer-term retention/LTV is out of scope for this dataset.
- Root cause of the support ticket increase (UI confusion, pricing clarity, technical issues, etc.) is not diagnosable from this dataset alone.

## Screenshots Included
- `screenshots/summary_metrics.png` — Control vs Treatment summary table
- `screenshots/hypothesis_test_output.png` — full z-test inputs, calculation, and decision
- `screenshots/kpi_tree_preview.png` — KPI tree image
