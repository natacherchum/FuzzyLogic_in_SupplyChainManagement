# Fuzzy Inference for Allocating Scarce Inventory

An educational Python example of a **Mamdani fuzzy inference system (FIS)** for assigning store priorities when warehouse inventory cannot satisfy all requests.

The implementation uses **scikit-fuzzy** (`skfuzzy`) and can be run in **Google Colab** or a local Python environment.

## The allocation problem

A central warehouse has **300 units** available, while three stores request **500 units** in total. The next warehouse replenishment is expected in one week.

Fulfilling 60% of every request would ignore differences in the stores’ inventory coverage and demand trends. This example formalises a priority-setting policy through linguistic variables, membership functions and fuzzy rules.

The workflow has two distinct stages:

1. **Fuzzy inference:** calculate a priority for each store.
2. **Constrained allocation:** use these priorities as weights to distribute available inventory without exceeding store requests.

The FIS output is an **allocation weight**, not a fulfilment percentage or a stockout probability.

## Fuzzy inference system

### Variables

| Variable | Role | Universe of discourse | Linguistic terms |
|---|---|---|---|
| Inventory coverage | Input | 0–14 days of demand | Critical, limited, sufficient |
| Change in demand | Input | −50% to +50% | Decreasing, stable, increasing |
| Request fulfilment priority | Output | 0–1 | Low, medium, high |

Inventory coverage is available store inventory divided by average daily demand. Change in demand is measured against the preceding comparable period: enter `20` for a 20% increase, not `0.20`.

### Membership functions

Parameters below follow the `skfuzzy` conventions: `trimf([a, b, c])` for triangles and `trapmf([a, b, c, d])` for trapezoids.

| Variable | Term | Function | Parameters |
|---|---|---|---|
| Coverage | Critical | `trapmf` | `[0, 0, 2, 5]` |
| Coverage | Limited | `trimf` | `[2, 5, 8]` |
| Coverage | Sufficient | `trapmf` | `[5, 8, 14, 14]` |
| Demand change | Decreasing | `trimf` | `[-50, -50, 0]` |
| Demand change | Stable | `trimf` | `[-50, 0, 50]` |
| Demand change | Increasing | `trimf` | `[0, 50, 50]` |
| Priority | Low | `trimf` | `[0, 0, 0.5]` |
| Priority | Medium | `trimf` | `[0, 0.5, 1]` |
| Priority | High | `trimf` | `[0.5, 1, 1]` |

Overlapping functions allow an input to partially belong to neighbouring fuzzy sets. Membership degrees are not probabilities.

### Rule base

| Inventory coverage / Demand change | Decreasing | Stable | Increasing |
|---|---|---|---|
| Critical | High | High | High |
| Limited | Low | Medium | High |
| Sufficient | Low | Low | Medium |

Example: **IF coverage is limited AND demand is increasing, THEN priority is high.**

### Inference procedure

- **Fuzzification:** evaluate input membership degrees.
- **AND:** use the minimum membership degree as the rule’s firing strength.
- **Implication:** clip each output membership function at its rule’s firing strength.
- **Aggregation:** take the pointwise maximum of the clipped outputs.
- **Defuzzification:** calculate the centroid of the aggregated fuzzy set.
