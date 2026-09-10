# The One Day Promise That Was Never One Day
### Diagnosing a Silent SLA Failure in Supply Chain Data

**Tools:** Tableau · **Dataset:** DataCo Smart Supply Chain (Kaggle) · **Author:** Dayanand P C

[Live interactive dashboard →](https://public.tableau.com/app/profile/dayanand.p.c/viz/Kaggle_DataCo_SCM_Dayanand_P_C/ScheduledvsActualFindingaSilentSLAFailureinSupplyChainData)

---

## Introduction

This project uses the DataCo Smart Supply Chain dataset, 180,519 real order records from a global retailer, spanning multiple regions, product categories, and four shipping tiers. Every order in this dataset carries two dates that matter more than any other field: a scheduled delivery window, promised the moment the order ships, and the actual delivery time that followed. Late delivery is common in retail, customers expect it occasionally and rarely think hard about why it happens. This project treats that gap between promise and delivery as a business problem worth diagnosing properly, not just describing.

## Why this question, and not a different one

Network wide, 57.29% of orders arrive late. That figure by itself is close to useless, it tells you a problem exists without telling you where it lives, why it's happening, or whether it's worth fixing. Most public analysis built on this dataset stops exactly here, a rate, a chart, a conclusion that late delivery is bad. This project asks three further questions instead: where specifically does lateness concentrate, what mechanism is actually causing it, and what does it cost in terms a business would recognize as real, not inflated for effect. Answering all three, in that order, is what turns a descriptive chart into an actual diagnosis.

## Finding the real signal

Cutting late delivery rate by region surfaces a modest pattern. Central Africa runs highest at 60.15%, against a network average of 57.29%, a gap of under three percentage points. Cutting by product category surfaces a slightly stronger pattern, Golf Bags & Carts sit at 68.85% against a 57.76% category average. Both are real, and neither is decisive, a gap this size could plausibly be explained by dozens of unrelated regional or seasonal factors, and neither cut points toward a specific, fixable cause.

Cutting late delivery rate by shipping mode changes the picture entirely. First Class runs at 100% late across 27,814 orders, every single one. Second Class runs at 79.83% across 35,216 orders. Same Day sits at 47.93%. Standard Class, the tier carrying the largest share of total order volume in the entire network, sits lowest at 39.77%, making it the network's most reliable tier. That's a sixty percentage point spread between the fastest tier and the most dependable one, roughly ten times larger than anything region or category produced. This is the finding worth chasing.

| Cut | Highest value | Network average | Spread |
|---|---|---|---|
| Region | Central Africa, 60.15% | 57.29% | +2.9 pts |
| Category | Golf Bags & Carts, 68.85% | 57.76% | +11.1 pts |
| Shipping Mode | First Class, 100.00% | 66.88% | +60.2 pts vs Standard Class |

## Why premium shipping is actually the problem, and why that's surprising

The intuitive read of that spread is that First Class shipping must be operationally inferior, slower fulfillment, worse handling, understaffed premium lanes. That intuition doesn't survive a second cut of the data. Comparing each tier's scheduled delivery days against its actual delivery days tells a different story entirely.

| Shipping Mode | Scheduled (days) | Actual (days) | Gap |
|---|---|---|---|
| First Class | 1.000 | 2.000 | 2x |
| Second Class | 2.000 | 3.991 | ~2x |
| Same Day | 0.000 | 0.478 | modest |
| Standard Class | 4.000 | 3.996 | ~exact |

First Class promises 1.0 day and delivers in 2.0 on average, exactly double its own promise. Second Class promises 2.0 days and delivers in 3.99, again nearly double. Standard Class promises 4.0 days and delivers in 3.996, essentially exact.

The failure isn't in execution, it's in the promise itself. Standard Class isn't succeeding because its operations are superior to First Class, it's succeeding because someone, at some point, set its delivery window honestly, and that number has stayed aligned with reality ever since. First Class and Second Class were given delivery windows the fulfillment process was never actually built to hit, and nobody has revisited those numbers since they were set. This is a scheduling calibration failure, not a shipping performance failure, and the distinction matters because it changes what the fix looks like entirely.

This finding also holds up against outside scrutiny. A separate, independently built analysis of this same dataset, using a different tool and reaching the conclusion without any connection to this project, reported the identical pattern: when First Class schedules 1 day, actual shipping is consistently 2 days, a 100% late rate. Two independent methods landing on the same number is a stronger basis for confidence than either one alone.

## What this is actually costing

Rather than assume a broken promise is expensive, this project quantified it directly. Comparing profit ratio for on time versus late orders within Second Class, the tier large enough to make a meaningful comparison, on time orders average 12.25% profit, late orders average 11.76%, a gap of 0.49 percentage points. Applied across $5.48 million in sales tied to Second Class's 26,987 late orders, that gap represents an **estimated $26,955 in profit quietly given up**.

This number is deliberately reported at its real size, modest rather than dramatic, because overstating it would undermine the credibility of everything else in this analysis. First Class shows a similarly reduced profit ratio on its late orders (12.73%), but with zero on-time orders in that tier to compare against, no equivalent gap could be isolated there.

One caveat worth stating plainly: this is a point estimate drawn from an observed correlation, not a controlled experiment. Late delivery and lower profit ratio move together within Second Class, but this analysis does not isolate lateness as the sole cause of the margin gap, other factors correlated with delivery delay, such as order complexity or discounting behavior, could contribute. The $26,955 figure should be read as a reasonable estimate of the scale of the issue, not a precise, audited cost.

## A data quality check that strengthened the finding rather than weakening it

The dataset ships with its own pre-built late delivery flag (`Late_delivery_risk`), reporting First Class at 95.32% late, differing from the 100% found through direct comparison of scheduled and actual days. Investigating that six point gap traced it to 1,301 cancelled First Class orders, which the pre-built flag excludes from its calculation, since a cancelled order was never technically delivered late or on time.

Rebuilding the late flag from scratch to properly exclude cancelled orders and rerunning the full analysis produced the same 100% figure. The finding survives the correction. The 1,301 excluded orders were never secretly on time, they were simply orders that should never have been counted either way.

## What should actually happen next

This is not a capacity problem requiring new investment, and it is not a carrier performance problem requiring a vendor renegotiation. It is a scheduling recalibration: resetting the promised delivery window on First Class and Second Class to reflect what the fulfillment process has actually delivered for years.

In a real engagement, this would move in three steps. First, operations leadership would need to confirm why the original 1-day and 2-day windows were set, whether it was a competitive positioning decision, a legacy commitment, or simply never revisited, since that changes how much resistance the change would face internally. Second, the new delivery windows should be piloted on a subset of First Class orders, one region or one fulfillment center, before a network-wide rollout, so the 2.0 and 3.99 day actuals used here are stress-tested against a live sample rather than trusted as a permanent baseline. Third, the fix should be measured on a 30/60/90 day cycle post-rollout, tracking late delivery rate and profit ratio for the recalibrated tiers specifically, to confirm the gap actually closes rather than just relabeling a problem that persists.

This would typically sit with whoever owns service level agreements, likely a logistics or fulfillment operations lead, in coordination with whoever set customer-facing delivery promises originally, often a product or customer experience function. It is low effort relative to almost any other fix available in this dataset, and it is the only one of the three findings in this analysis with a validation path that costs nothing to test before committing to it network-wide.

---

## Methodology notes

- Built entirely in Tableau, five linked views: late rate by region, by category, by shipping mode, scheduled vs actual days by mode, and profit ratio impact by mode
- Late delivery flag (`Is Late`) built from first principles, `Days for shipping (real) > Days for shipment (scheduled)`, rather than relying on the dataset's pre-built label, with cancelled orders explicitly excluded
- Network average reference line computed as a `FIXED` calculation to ensure a single consistent benchmark across all charts, rather than each chart re-averaging its own visible marks
- All figures in this write-up were independently re-verified in a separate pass directly against the raw CSV, not carried forward from intermediate calculations
- Dataset: [DataCo Smart Supply Chain Dataset, Kaggle](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)
