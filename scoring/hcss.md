# Hierarchical CWE Scoring System (HCSS)

Below is a proposal for a comprehensive scoring system for evaluating automated CVE-to-CWE assignments that addresses both the multi-label nature of the problem and the hierarchical structure of the CWE framework.

See [A survey of Hierarchical Classification Standards](./hierarchical_scoring_system.md) for background context.

## 1. Core Design Principles

The Hierarchical CWE Scoring System (HCSS) is designed with the following principles:

1. **Handle Multi-Label Classification**: Support evaluating cases where a CVE has multiple correct CWE labels
2. **Leverage Hierarchy**: Use the CWE hierarchy to award partial credit based on semantic proximity
3. **Balanced Evaluation**: Provide metrics that balance precision and recall
4. **Standardized Scoring**: Deliver scores in an interpretable range (0-1)
5. **Comparability**: Align with established evaluation practices in hierarchical classification

## 2. HCSS Methodology

### 2.1 Set Augmentation Approach

HCSS uses a set augmentation approach based on established hierarchical classification research:

1. For each CVE, we have:
   - **Y**: The set of true/benchmark CWE identifiers
   - **Ŷ**: The set of predicted CWE identifiers

2. We augment both sets with all ancestors in the CWE hierarchy:
   - **Y_aug** = Y ∪ {all ancestors of each CWE in Y}
   - **Ŷ_aug** = Ŷ ∪ {all ancestors of each CWE in Ŷ}

3. Calculate hierarchical precision (hP), recall (hR), and F-measure (hF) using set operations:
   - **hP** = |Ŷ_aug ∩ Y_aug| / |Ŷ_aug|
   - **hR** = |Ŷ_aug ∩ Y_aug| / |Y_aug|
   - **hF** = 2 × hP × hR / (hP + hR)

### 2.2 Aggregation Methods

For a dataset containing multiple CVEs, HCSS calculates two types of averages:

1. **Micro-average**: Aggregates the numerators and denominators across all CVEs before division
   - Gives more weight to CVEs with more complex mappings
   - Reflects overall performance

2. **Macro-average**: Calculates metrics for each CVE separately and then averages
   - Gives equal weight to each CVE
   - Shows performance across different types of vulnerabilities

The primary results to be reported are Micro-hF and Macro-hF, along with the constituent hP and hR values.

## 3. Implementation Details

### 3.1 Ancestor Calculation

To determine ancestors for a CWE node:
1. Parse the MITRE CWE XML data to build the hierarchy graph
2. For each node, recursively traverse up all "ChildOf" relationships to identify all ancestors
3. Store the ancestors for efficient lookup during scoring

### 3.2 Handling Special Cases

- **Empty sets**: If Y or Ŷ is empty, handle accordingly (0 for precision when Ŷ is empty, 0 for recall when Y is empty)
- **Root node**: The root node CWE-1000 is not included in the comparison sets. 
  - If it was, it would result in overly generous partial credit even for a complete mismatch, since there is always 1 element (the root) common in both sets
- **Different views**: The CWE can be viewed through different perspectives (e.g., CWE-1000 per https://riskbasedprioritization.github.io/cwe/cwe_views/) which should be used for evaluation.

## 4. Worked Examples

### Example 1: Single CWE, Exact Match

- **Benchmark**: CWE-79
- **Prediction**: CWE-79

1. Augment sets (actual hierarchy: CWE-79 → CWE-74 → CWE-707 → CWE-1000, but excluding CWE-1000):

   * Y\_aug = {CWE-79, CWE-74, CWE-707}
   * Ŷ\_aug = {CWE-79, CWE-74, CWE-707}

2. Calculate metrics:

   * hP = |{CWE-79, CWE-74, CWE-707}| / |{CWE-79, CWE-74, CWE-707}| = 3/3 = 1.0
   * hR = |{CWE-79, CWE-74, CWE-707}| / |{CWE-79, CWE-74, CWE-707}| = 3/3 = 1.0
   * hF = 2 × 1.0 × 1.0 / (1.0 + 1.0) = 1.0

**Result:** Perfect score (**1.0**) for exact match.

---

### Example 2: Single CWE, No Match

- **Benchmark**: CWE-79
- **Prediction**: CWE-352 (unrelated to CWE-79)

1. Augment sets (excluding CWE-1000):

   * Ancestors of CWE-79: CWE-79 → CWE-74 → CWE-707
   * Ancestors of CWE-352: CWE-352 → CWE-345 → CWE-693
   * Y\_aug = {CWE-79, CWE-74, CWE-707}
   * Ŷ\_aug = {CWE-352, CWE-345, CWE-693}

2. Calculate metrics:

   * hP = |{}| / |{CWE-352, CWE-345, CWE-693}| = 0/3 = 0.0
   * hR = |{}| / |{CWE-79, CWE-74, CWE-707}| = 0/3 = 0.0
   * hF = 0.0 (by definition when precision and recall are zero)

**Result:** Complete mismatch (**0.0**).

---

### Example 3: Single CWE, Parent Relationship

- **Benchmark**: CWE-79
- **Prediction**: CWE-74 (direct parent of CWE-79)

1. Augment sets (excluding CWE-1000):

   * Y\_aug = {CWE-79, CWE-74, CWE-707}
   * Ŷ\_aug = {CWE-74, CWE-707}

2. Calculate metrics:

   * hP = |{CWE-74, CWE-707}| / |{CWE-74, CWE-707}| = 2/2 = 1.0
   * hR = |{CWE-74, CWE-707}| / |{CWE-79, CWE-74, CWE-707}| = 2/3 ≈ 0.667
   * hF = 2 × 1.0 × 0.667 / (1.0 + 0.667) ≈ 1.334 / 1.667 ≈ 0.800

**Result:** High but not perfect score (**0.800**) for parent prediction.

---


### Example 4: Complex Multi-Label Case
- **Benchmark**: CWE-79, CWE-89
- **Prediction**: CWE-79, CWE-74, CWE-352

1. **Augment sets** (using the provided chains, excluding CWE-1000):

* Ancestors of **CWE-79**: CWE-79 → CWE-74 → CWE-707
* Ancestors of **CWE-89**: CWE-89 → CWE-943 → CWE-74 → CWE-707
* Ancestors of **CWE-352**: CWE-352 → CWE-345 → CWE-693

Thus:

* **Y\_aug** (Benchmark set) = {CWE-79, CWE-89, CWE-74, CWE-707, CWE-943}
* **Ŷ\_aug** (Prediction set) = {CWE-79, CWE-74, CWE-707, CWE-352, CWE-345, CWE-693}

2. **Calculate metrics**:

* Intersection: {CWE-79, CWE-74, CWE-707}
* Precision (**hP**):

$$
\frac{| \{CWE-79, CWE-74, CWE-707\} |}{| \{CWE-79, CWE-74, CWE-707, CWE-352, CWE-345, CWE-693\} |} = \frac{3}{6} = 0.500
$$

* Recall (**hR**):

$$
\frac{| \{CWE-79, CWE-74, CWE-707\} |}{| \{CWE-79, CWE-89, CWE-74, CWE-707, CWE-943\} |} = \frac{3}{5} = 0.600
$$

* F-score (**hF**):

$$
\frac{2 \times 0.500 \times 0.600}{(0.500 + 0.600)} = \frac{0.600}{1.100} \approx 0.545
$$

**Result:** Moderate score (**0.545**) due to correctly predicting CWE-79 and its ancestors but failing to predict CWE-89 and adding unrelated CWEs (CWE-352).

---


## 5. Advantages of HCSS

1. **Theoretical Grounding**: Based on established research in hierarchical multi-label classification
2. **Semantic Awareness**: Incorporates the CWE hierarchy structure to reward closeness
3. **Balanced Evaluation**: Captures both precision and recall aspects
4. **Interpretable**: Produces normalized scores in a meaningful range
5. **Flexibility**: Supports both micro and macro averaging for different analysis needs
6. **Standardized**: Aligns with scientific evaluation practices for comparable results

## 6. Implementation Recommendations

1. Use the MITRE CWE XML data to build the hierarchy graph
2. Consider implementing optional weights for edges based on abstraction levels
3. Provide both micro and macro-averaged scores for comprehensive evaluation
4. Include standard metrics alongside HCSS for comparative context

This HCSS framework provides a robust, theoretically sound approach to evaluating automated CVE-to-CWE assignments, directly addressing the multi-label and hierarchical nature of the problem.