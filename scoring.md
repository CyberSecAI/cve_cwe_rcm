# Scoring_formula

## Data Points
A CVE can have one or more CWEs i.e. it's a multi-label classification problem.

MITRE CWE list is a rich document containing detailed information on CWEs (~2800 pages)
The RelatedNatureEnumerations form a (view-dependent e.g. CWE-1003) graph of 
- 1309 ChildOf/ParentOf
- 141 CanPrecede/CanFollow
- 13 Requires/RequiredBy

---

## Scoring System: Overview

The goal is to assign a numeric score that:

- Rewards exact matches highly.
- Rewards partial matches based on hierarchical proximity (parent-child relationships).
- Provides a standardized range (e.g. 0 to 1) for easy comparison.

---

## Definitions & Assumptions

- **Ground Truth (Benchmark)**: One or more known-good CWEs per CVE.
- **Prediction**: One or more CWEs assigned by the tested solution.
- **Hierarchy**: Relationships defined by MITRE’s CWE hierarchy (ChildOf, ParentOf, and sibling CWEs).
- **Proximity**: Measured as shortest hierarchical path (edge distance) between two CWEs.

---

## Step-by-Step Scoring Methodology

### Step 1: Calculate Hierarchical Distance
For each **predicted CWE**, calculate the shortest distance to each **benchmark CWE**:

- **Exact match**: Distance = **0**
- **Direct Parent or Child**: Distance = **1**
- **Grandparent, Grandchild, or Sibling**: Distance = **2**
- **Greater Distance (up/down the tree)**: Increment by 1 per hierarchical level traversed

If no hierarchical path exists between CWEs (unrelated), assign a large default penalty (e.g., distance = **10**).

### Step 2: Convert Distances to Proximity Scores
Convert hierarchical distances into proximity scores. For example:

$ProximityScore(d) = \frac{1}{(1 + d)}$


This formula produces intuitive scores:

| Distance | Proximity Score |
|----------|-----------------|
| 0        | 1.00            |
| 1        | 0.50            |
| 2        | 0.33            |
| 3        | 0.25            |
| 4        | 0.20            |
| ≥10      | ~0.09 or lower  |

---

## Step 3: Aggregate Proximity Scores (Multi-label Case)

Because a CVE can have multiple benchmark CWEs and multiple predictions, aggregate as follows:
- For each predicted CWE, take the average (or sum normalized by number of benchmarks) of proximity scores across all benchmark CWEs.


Given:

- Benchmark CWEs: 
  
$$
    \text{Benchmark CWEs B}= \{b_1, b_2, \dots, b_m\}   
$$

- Predicted CWEs: 
  
$$
        \text{Predicted CWEs P}= \{p_1, p_2, \dots, p_n\}
$$

Calculate scores as follows:


- **Recall**:  


$$
    \text{Recall}_{CVE} = \frac{1}{|\text{Benchmark CWEs}|}\sum_{\text{Benchmark CWEs}} \left(\frac{\sum_{\text{Predictions}} \text{ProximityScore}}{|\text{Predictions}|}\right)
$$

- **Precision**:  

$$
    \text{Precision}_{CVE} = \frac{1}{|\text{Predicted CWEs}|}\sum_{\text{Predicted CWEs}} \left(\frac{\sum_{\text{Benchmark CWEs}} \text{ProximityScore}}{|\text{Benchmark CWEs}|}\right)
$$

- **F1-Score** (Harmonic mean of precision and recall):  

$$
    \text{F1}_{CVE} = \frac{2 \times \text{Precision}_{CVE} \times \text{Recall}_{CVE}}{\text{Precision}_{CVE} + \text{Recall}_{CVE}}
$$



---

## Worked Example

**Benchmark CWEs**: CWE-79, CWE-89  
**Predicted CWEs**: CWE-79, CWE-74 (distance 2 from CWE-79, distance 3 from CWE-89), CWE-352 (unrelated)

| Predicted CWE | Benchmark CWE | Distance \( d \) | Proximity Score |
|---------------|---------------|------------------|-----------------|
| CWE-79        | CWE-79        | 0                | 1.00            |
| CWE-79        | CWE-89        | 3                | 0.25            |
| CWE-74        | CWE-79        | 2                | 0.33            |
| CWE-74        | CWE-89        | 3                | 0.25            |
| CWE-352       | CWE-79        | 10               | 0.09            |
| CWE-352       | CWE-89        | 10               | 0.09            |




### Recall Calculation:


  $$
    \text{Recall (CWE-79)} = \frac{1.00 + 0.33 + 0.09}{3} = 0.473
  $$

  $$
    \text{Recall (CWE-89)} = \frac{0.25 + 0.25 + 0.09}{3} = 0.197
  $$

  $$
    \text{Overall Recall} = \frac{0.473 + 0.197}{2} = 0.335
  $$

### Precision Calculation:

  $$
    \text{Precision (CWE-79)} = \frac{1.00 + 0.25}{2} = 0.625
  $$

  $$
    \text{Precision (CWE-74)} = \frac{0.33 + 0.25}{2} = 0.29
  $$


  $$
    \text{Precision (CWE-352)} = \frac{0.09 + 0.09}{2} = 0.09
  $$

  $$
      \text{Overall Precision} = \frac{0.625 + 0.29 + 0.09}{3} = 0.335
  $$

### F1-Score Calculation:

$$
\text{F1-score} = \frac{2 \times 0.335 \times 0.335}{0.335 + 0.335} = 0.335
$$



### Interpretation

- Scores near \(1.0\) indicate highly accurate assignments.
- Scores around \(0.3 - 0.5\) indicate moderate accuracy.
- Lower scores indicate poor predictions.


## ToDo
1. Assign different weights to different relationship types:

    - ChildOf/ParentOf: Weight 1.0
    - Requires/RequiredBy: Weight 0.8
    - CanPrecede/CanFollow: Weight 0.7
    - Sibling Of (same parent): Weight 0.6


2. Introduce a tunable parameter B to adjust the sensitivity of proximity scores:

$$
\text{ProximityScore}(d) = \frac{1}{1 + \beta \times d}
$$
