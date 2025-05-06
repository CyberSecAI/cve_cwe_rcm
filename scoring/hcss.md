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

### Example 2: Single CWE, Parent Relationship

-   **Benchmark**: CWE-79
-   **Prediction**: CWE-74 (direct parent of CWE-79)

1.  **Augment sets** (using the hierarchy: CWE-79 → CWE-74 → CWE-707, excluding CWE-1000):

    * Ancestors of CWE-79 are CWE-74 and CWE-707.
    * **Y\_aug** (Benchmark set) = {CWE-79, CWE-74, CWE-707}
    * Ancestors of CWE-74 are CWE-707.
    * **Ŷ\_aug** (Prediction set) = {CWE-74, CWE-707}

2.  **Calculate metrics**:

    * Intersection: $Y_{aug} \cap \hat{Y}_{aug} = \{CWE-79, CWE-74, CWE-707\} \cap \{CWE-74, CWE-707\} = \{CWE-74, CWE-707\}$
    * Precision (**hP**):
        $$
        \frac{|\{CWE-74, CWE-707\}|}{|\{CWE-74, CWE-707\}|} = \frac{2}{2} = 1.0
        $$
    * Recall (**hR**):
        $$
        \frac{|\{CWE-74, CWE-707\}|}{|\{CWE-79, CWE-74, CWE-707\}|} = \frac{2}{3} \approx 0.667
        $$
    * F-score (**hF**):
        $$
        \frac{2 \times 1.0 \times 0.667}{(1.0 + 0.667)} = \frac{1.334}{1.667} \approx 0.800
        $$

**Result:** High but not perfect score (**0.800**) for predicting a direct parent.

---

### Example 3: Single CWE, No Match

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

### Example 4: Single CWE, Parent Relationship

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

### Example 5: Complex Multi-Label Case
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

### Example 6: Complex Multi-Label Case with Missing and Partial Matches

- **Benchmark**: CWE-79, CWE-89, CWE-352
- **Prediction**: CWE-79, CWE-74

1. **Augment sets** (using provided hierarchy, excluding CWE-1000):

* Ancestors of **CWE-79**: CWE-79 → CWE-74 → CWE-707
* Ancestors of **CWE-89**: CWE-89 → CWE-943 → CWE-74 → CWE-707
* Ancestors of **CWE-352**: CWE-352 → CWE-345 → CWE-693

Thus:

* **Y\_aug** (Benchmark set) = {CWE-79, CWE-89, CWE-352, CWE-74, CWE-707, CWE-943, CWE-345, CWE-693}
* **Ŷ\_aug** (Prediction set) = {CWE-79, CWE-74, CWE-707}

2. **Calculate metrics**:

* Intersection: {CWE-79, CWE-74, CWE-707}
* Precision (**hP**):

$$
\frac{| \{CWE-79, CWE-74, CWE-707\} |}{| \{CWE-79, CWE-74, CWE-707\} |} = \frac{3}{3} = 1.0
$$

* Recall (**hR**):

$$
\frac{| \{CWE-79, CWE-74, CWE-707\} |}{| \{CWE-79, CWE-89, CWE-352, CWE-74, CWE-707, CWE-943, CWE-345, CWE-693\} |} = \frac{3}{8} = 0.375
$$

* F-score (**hF**):

$$
\frac{2 \times 1.0 \times 0.375}{(1.0 + 0.375)} = \frac{0.75}{1.375} \approx 0.545
$$

**Result:** Moderate score (**0.545**) reflecting precise but incomplete prediction—correctly capturing CWE-79 but missing CWE-89, CWE-352..

---
Okay, let's create that example using the HCSS methodology and the CWE chains you provided.

---

### Example 7: Complex Multi-Label Case with Multiple Paths and Partial Matches

> [!NOTE]  
> CWE-798 appears in 3 different chains. 
> 
> This raises the question on which chain(s) do we use
> - the shortest chain?
> - all the chains?
> - a selected chain? 
>
> For one chain:
> - If a Predicted CWE is a ancestor/descendant of a Benchmark CWE then this uniquely identifies the chain.
> - If a Predicted CWE equals the Benchmark CWE then we still need to pick a chain
>
> For all chains:
> - the logic is the same always
> - the Recall and F1 will be lower if the CWE that appears in multiple chains is in the Predicted set


#### Using all 3 chains for CWE-798
-   **Benchmark**: CWE-912, CWE-798
-   **Prediction**: CWE-321, CWE-912

We will use the following CWE chains provided (excluding "CWE View 1000 Root"):

* CWE-912 Chain: `CWE-710 > CWE-684 > CWE-912`
* CWE-798 Chains:
    * Chain 1: `CWE-710 > CWE-657 > CWE-671 > CWE-798`
    * Chain 2: `CWE-284 > CWE-287 > CWE-1390 > CWE-1391 > CWE-798`
    * Chain 3: `CWE-693 > CWE-330 > CWE-344 > CWE-798`
* CWE-321 Chain: `CWE-284 > CWE-287 > CWE-1390 > CWE-1391 > CWE-798 > CWE-321`

1.  **Augment sets** (excluding CWE-1000 Root):

    * **Benchmark set (Y)** = {CWE-912, CWE-798}
        * Ancestors of CWE-912: {CWE-710, CWE-684}
        * Ancestors of CWE-798 (from all paths where it's a descendant): {CWE-710, CWE-657, CWE-671, CWE-284, CWE-287, CWE-1390, CWE-1391, CWE-693, CWE-330, CWE-344}
        * **Y\_aug** = Y $\cup$ Ancestors(Y) = {CWE-912, CWE-798} $\cup$ {CWE-710, CWE-684} $\cup$ {CWE-710, CWE-657, CWE-671} $\cup$ {CWE-284, CWE-287, CWE-1390, CWE-1391} $\cup$ {CWE-693, CWE-330, CWE-344}
        * **Y\_aug** = {CWE-912, CWE-798, CWE-710, CWE-684, CWE-657, CWE-671, CWE-284, CWE-287, CWE-1390, CWE-1391, CWE-693, CWE-330, CWE-344}
        * $|Y_{aug}| = 13$

    * **Prediction set (Ŷ)** = {CWE-321, CWE-912}
        * Ancestors of CWE-321: {CWE-798, CWE-1391, CWE-1390, CWE-287, CWE-284}
        * Ancestors of CWE-912: {CWE-710, CWE-684}
        * **Ŷ\_aug** = Ŷ $\cup$ Ancestors(Ŷ) = {CWE-321, CWE-912} $\cup$ {CWE-798, CWE-1391, CWE-1390, CWE-287, CWE-284} $\cup$ {CWE-710, CWE-684}
        * **Ŷ\_aug** = {CWE-321, CWE-912, CWE-798, CWE-1391, CWE-1390, CWE-287, CWE-284, CWE-710, CWE-684}
        * $|\hat{Y}_{aug}| = 9$

2.  **Calculate metrics**:

    * Intersection: $Y_{aug} \cap \hat{Y}_{aug}$ = {CWE-912, CWE-798, CWE-710, CWE-684, CWE-284, CWE-287, CWE-1390, CWE-1391}
    * $|Y_{aug} \cap \hat{Y}_{aug}| = 8$

    * Precision (**hP**):
        $$
        \frac{| Y_{aug} \cap \hat{Y}_{aug} |}{| \hat{Y}_{aug} |} = \frac{8}{9} \approx 0.889
        $$

    * Recall (**hR**):
        $$
        \frac{| Y_{aug} \cap \hat{Y}_{aug} |}{| Y_{aug} |} = \frac{8}{13} \approx 0.615
        $$

    * F-score (**hF**):
        $$
        \frac{2 \times hP \times hR}{(hP + hR)} = \frac{2 \times 0.889 \times 0.615}{(0.889 + 0.615)} = \frac{1.093}{1.504} \approx 0.727
        $$

**Result:** A score of approximately **0.727**. The prediction correctly identified CWE-912 and its ancestors, partially captured CWE-798 by predicting its child (CWE-321) and some shared ancestors, but missed other paths leading to CWE-798 and its specific ancestors on those paths.

#### Using 1 of 3 chains for CWE-798

Here we recalculate Example 7, but this time applying the constraint that for CWE-798 in the benchmark set, we **only** use the ancestors from the chain `CWE View 1000 Root > CWE-284 > CWE-287 > CWE-1390 > CWE-1391 > CWE-798`.

The scenario is:
-   **Benchmark**: CWE-912, CWE-798
-   **Prediction**: CWE-321, CWE-912

We use the following chains (excluding "CWE View 1000 Root"):

* CWE-912 Chain: `CWE-710 > CWE-684 > CWE-912`
* CWE-798 Chain **(for Benchmark ONLY, constrained)**: `CWE-284 > CWE-287 > CWE-1390 > CWE-1391 > CWE-798`
* CWE-321 Chain (relevant for Prediction): `CWE-284 > CWE-287 > CWE-1390 > CWE-1391 > CWE-798 > CWE-321`
* CWE-912 Chain (relevant for Prediction): `CWE-710 > CWE-684 > CWE-912`

1.  **Augment sets** (excluding CWE-1000 Root, with the constraint on Benchmark CWE-798):

    * **Benchmark set (Y)** = {CWE-912, CWE-798}
        * Ancestors of CWE-912: {CWE-710, CWE-684}
        * Ancestors of CWE-798 **(using only the specified chain)**: {CWE-284, CWE-287, CWE-1390, CWE-1391}
        * **Y\_aug** = Y $\cup$ Ancestors(CWE-912) $\cup$ Ancestors(CWE-798 constrained)
        * **Y\_aug** = {CWE-912, CWE-798} $\cup$ {CWE-710, CWE-684} $\cup$ {CWE-284, CWE-287, CWE-1390, CWE-1391}
        * **Y\_aug** = {CWE-912, CWE-798, CWE-710, CWE-684, CWE-284, CWE-287, CWE-1390, CWE-1391}
        * $|Y_{aug}| = 8$

    * **Prediction set (Ŷ)** = {CWE-321, CWE-912}
        * Ancestors of CWE-321: {CWE-798, CWE-1391, CWE-1390, CWE-287, CWE-284}
        * Ancestors of CWE-912: {CWE-710, CWE-684}
        * **Ŷ\_aug** = Ŷ $\cup$ Ancestors(Ŷ) = {CWE-321, CWE-912} $\cup$ {CWE-798, CWE-1391, CWE-1390, CWE-287, CWE-284} $\cup$ {CWE-710, CWE-684}
        * **Ŷ\_aug** = {CWE-321, CWE-912, CWE-798, CWE-1391, CWE-1390, CWE-287, CWE-284, CWE-710, CWE-684}
        * $|\hat{Y}_{aug}| = 9$

2.  **Calculate metrics**:

    * Intersection: $Y_{aug} \cap \hat{Y}_{aug}$ = {CWE-912, CWE-798, CWE-710, CWE-684, CWE-284, CWE-287, CWE-1390, CWE-1391}
    * $|Y_{aug} \cap \hat{Y}_{aug}| = 8$

    * Precision (**hP**):
        $$
        \frac{| Y_{aug} \cap \hat{Y}_{aug} |}{| \hat{Y}_{aug} |} = \frac{8}{9} \approx 0.889
        $$

    * Recall (**hR**):
        $$
        \frac{| Y_{aug} \cap \hat{Y}_{aug} |}{| Y_{aug} |} = \frac{8}{8} = 1.0
        $$

    * F-score (**hF**):
        $$
        \frac{2 \times hP \times hR}{(hP + hR)} = \frac{2 \times 0.889 \times 1.0}{(0.889 + 1.0)} = \frac{1.778}{1.889} \approx 0.941
        $$

**Result:** A score of approximately **0.941**. Restricting the ancestors of CWE-798 in the benchmark set to a single chain significantly increased the Recall and the overall F-score in this specific example, as the prediction happened to align well with the ancestors on that particular chain. This highlights how the choice of hierarchy representation (e.g., using a specific view or all paths) impacts the scoring.

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