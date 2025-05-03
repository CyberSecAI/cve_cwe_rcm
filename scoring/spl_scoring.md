

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


# Evaluation Metrics

## Recall

Evaluate how well benchmark CWEs are covered by predictions:

$$
\text{Recall}_{CVE} = \frac{1}{|B|} \sum_{b \in B} \left( \frac{\sum_{p \in P} \text{ProximityScore}(b, p)}{|P|} \right)
$$

## Precision

Evaluate how accurately predictions align with benchmark CWEs:

$$
\text{Precision}_{CVE} = \frac{1}{|P|} \sum_{p \in P} \left( \frac{\sum_{b \in B} \text{ProximityScore}(p, b)}{|B|} \right)
$$

## F1-Score

Harmonic mean of precision and recall:

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
