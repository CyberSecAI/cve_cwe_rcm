
# Comparative Analysis of a Proposed Hierarchical Scoring Formula against Research Findings for CWE Classification Evaluation

## 1. Introduction

### 1.1. Context

The accurate assignment of Common Weakness Enumeration (CWE) identifiers to Common Vulnerabilities and Exposures (CVEs) is a critical task in cybersecurity. CWE provides a standardized language and categorization for software and hardware weaknesses, enabling better understanding, mitigation, and prevention of vulnerabilities. Effective CWE assignment supports vulnerability management, risk assessment, secure development practices, and the evaluation of security tools. Given the increasing volume of CVEs, automated tools, including machine learning models, are increasingly employed for this task. Evaluating the performance of these automated systems presents a significant challenge. The CWE structure is inherently hierarchical, typically represented as a Directed Acyclic Graph (DAG), where weaknesses are organized into different levels of abstraction (e.g., Pillar, Class, Base, Variant). Furthermore, a single CVE can often be associated with multiple CWEs, reflecting different facets of the underlying vulnerability. Consequently, the evaluation of automated CWE assignment tools falls under the domain of Hierarchical Multi-label Classification (HMC).

### 1.2. Problem Statement

Evaluating HMC systems necessitates metrics that go beyond traditional flat classification measures. Standard metrics like accuracy, precision, and recall, when applied naively, often fail to account for the crucial aspects of HMC: the hierarchical relationships between classes and the multi-label nature of predictions. Ignoring the hierarchy means treating all classification errors as equally severe, which contradicts intuition; for instance, misclassifying a specific type of 'Cross-Site Scripting' (a deep node) as a different specific type might be considered less erroneous than misclassifying it as 'Improper Access Control' (a node in a completely different branch, potentially shallower). Similarly, standard multi-label metrics might not adequately incorporate the hierarchical structure when assessing partial correctness. Therefore, specialized evaluation measures are required for meaningful assessment and comparison of HMC algorithms.

### 1.3. User Proposal Overview

This report analyzes a proposed scoring formula designed for evaluating automated CWE classification systems. The formula attempts to address the HMC nature of the task by incorporating a measure of distance within the CWE hierarchy to calculate a proximity score, which is then used to compute custom Precision, Recall, and F1-score metrics, aggregated across multiple labels and instances.

### 1.4. Report Objectives and Structure

The primary objective of this report is to provide an expert analysis of the proposed scoring formula, comparing its components and overall methodology against established research findings and best practices in HMC evaluation, particularly within the context of CWE classification. This analysis will identify the strengths and weaknesses of the proposal and offer concrete, research-backed recommendations for improvement. The report is structured as follows: Section 2 dissects the components of the user's formula. Section 3 compares the proposed hierarchical distance metric against alternatives discussed in the research. Section 4 evaluates the proposed aggregation method for Precision and Recall relative to standard multi-label and hierarchical practices. Section 5 assesses the partial credit mechanism. Sections 6 and 7 summarize the pros and cons of the proposal, respectively. Section 8 analyzes the potential impact of the user's suggested future work ("ToDo" items). Section 9 proposes specific improvements based on the research. Finally, Section 10 provides concluding remarks.

## 2. Analysis of the User's Proposed Scoring Formula

The proposed formula consists of three main components: a method for calculating hierarchical distance, a function to convert this distance into a proximity score, and an aggregation method to compute overall Precision, Recall, and F1-score.

### 2.1. Hierarchical Distance Calculation

The core of the hierarchical component relies on calculating the distance between a predicted CWE node and a true CWE node within the hierarchy graph.

* **Method:** The proposed distance metric is the **Shortest Path Length (SPL)** between the two nodes in the CWE hierarchy graph. The CWE hierarchy is generally structured as a Directed Acyclic Graph (DAG), allowing multiple parents for a node. The SPL calculation presumably traverses the graph edges, counting the number of steps in the shortest path connecting the two nodes, ignoring edge directionality for distance calculation purposes.
* **Sibling/Unrelated Handling:** For cases where nodes do not lie on a direct ancestor-descendant path, the proposal introduces specific, fixed distance values. If two nodes are siblings (sharing an immediate parent), a distance $d_{sibling}$ is assigned. If nodes are otherwise unrelated (no shared parent and not in an ancestor/descendant relationship, potentially only sharing the root or very distant ancestors), a distance $d_{unrelated}$ is assigned. These appear to be user-defined parameters.
* **Implicit Assumptions:** This method implicitly assumes that each edge in the CWE hierarchy represents a uniform semantic distance, typically a distance of 1. It does not differentiate between different types of relationships (e.g., ParentOf, ChildOf, MemberOf, PeerOf mentioned in CWE documentation) or the level of abstraction at which the relationship occurs.

### 2.2. Proximity Score Conversion

The calculated distance $d$ is then transformed into a proximity score $s$ using an inverse relationship.

* **Formula:** The conversion uses the formula $s = 1 / (1 + d)$.
* **Behavior:** This function maps distances to a score between 0 and 1. When the distance $d=0$ (i.e., the predicted node is the same as the true node), the score $s=1$. As the distance $d$ increases, the score $s$ decreases non-linearly, approaching 0 for very large distances. This provides an intuitive mapping where closer nodes in the hierarchy receive higher proximity scores.
* **Partial Credit Mechanism:** This score $s$ serves as the mechanism for awarding partial credit. A prediction that is incorrect ($d>0$) but hierarchically close to a true label still receives a score greater than 0, reflecting partial correctness. The amount of credit decreases as the hierarchical distance increases.

### 2.3. Aggregation for Precision/Recall/F1

The final step involves aggregating these proximity scores to calculate overall Precision (P), Recall (R), and F1-score (F1) for the classification task, considering the multi-label nature.

* **Method:** The proposal outlines a custom aggregation method. While the exact details require clarification, it appears to involve summing the proximity scores ($s$) calculated between pairs of predicted and true labels for each data instance (e.g., a CVE). These summed scores are then likely normalized in some way to produce example-level P, R, and F1 values.
* **Example-Level Calculation:** For a single CVE with a set of true CWEs $Y=\{y_1,...,y_N\}$ and a set of predicted CWEs $P=\{p_1,...,p_M\}$, the calculation likely involves computing the proximity score $s_{ij} = 1 / (1 + d(p_i, y_j))$ for relevant pairs $(p_i, y_j)$. How these pairwise scores are combined into example-level P and R is crucial but unspecified – it might involve averaging scores for each predicted label against all true labels (for P) or for each true label against all predicted labels (for R), potentially using maximums or sums. For instance, a possible interpretation for example-level Precision could be $\frac{1}{M} \sum_{i=1}^{M} \max_{j=1}^{N} s_{ij}$ and for Recall $\frac{1}{N} \sum_{j=1}^{N} \max_{i=1}^{M} s_{ij}$, although other summation or averaging schemes are possible.
* **Overall Aggregation:** The example-level P, R, and F1 scores are then aggregated across the entire dataset to produce the final metrics. The proposal suggests simple averaging of these example-level scores, which corresponds to a form of macro-averaging, but applied to custom, non-standard example-level metrics.

## 3. Comparison of the Proposed Hierarchical Distance Metric

The choice of a distance or similarity metric is fundamental to incorporating hierarchical information into the evaluation. The user's proposal utilizes Shortest Path Length (SPL), which needs to be compared against metrics discussed in the research literature.

### 3.1. Comparison with Path-Length Based Metrics

Path-length based metrics are a common starting point for measuring similarity in ontologies and hierarchies.

* **User Proposal (SPL):** The proposal uses the direct shortest path length between nodes.
* **Research Findings (Pros):** The primary advantage of SPL is its simplicity and intuitive appeal: nodes that are fewer steps apart in the graph are considered more similar. It is computationally straightforward to calculate using standard graph traversal algorithms like Breadth-First Search. SPL forms the basis for several established semantic similarity measures.
* **Research Findings (Cons):** Despite its simplicity, SPL suffers from significant drawbacks when used for semantic similarity in rich hierarchies:
    * **Ignoring Node Depth/Specificity:** SPL treats all edges as equal, regardless of their depth in the hierarchy. However, nodes deeper in a hierarchy typically represent more specific concepts. An edge connecting two general concepts near the root (e.g., 'Hardware Fault' to 'Software Fault') often represents a larger semantic distance than an edge connecting two specific concepts deep within a branch (e.g., 'Off-by-one Error' to 'Buffer Overflow'). SPL fails to capture this crucial aspect of semantic specificity tied to depth. Research emphasizes the importance of considering node depth and the depth of the Least Common Subsumer (LCS) – the deepest shared ancestor – for more meaningful similarity assessment.
    * **Uniform Edge Weights:** The assumption of uniform distance (cost=1) for every edge is often unrealistic. The semantic distance between a parent and child might vary across the hierarchy, or different relationship types (if present beyond parent/child) might imply different distances. The user's SPL metric inherently adopts this uniformity assumption.
    * **Arbitrary Sibling/Unrelated Values:** Assigning fixed, arbitrary values like $d_{sibling}$ and $d_{unrelated}$ lacks a principled foundation. Standard path-based measures often focus on ancestor-descendant relationships or paths through the LCS, implicitly treating unrelated nodes as infinitely distant or requiring specific handling based on the LCS. The user's approach makes the final score highly sensitive to these parameters, whose values are not derived from the hierarchy's structure or semantics.

### 3.2. Comparison with Other Semantic Similarity Metrics

Recognizing the limitations of simple path length, researchers have developed more sophisticated metrics:

* **Wu & Palmer Similarity:** This widely cited measure directly addresses the depth issue. Its formula,
    $$ Sim_{WP}(c_1, c_2) = \frac{2 \times depth(LCS(c_1, c_2))}{depth(c_1) + depth(c_2)} $$
    incorporates the depth of the two concepts ($c_1, c_2$) and their Least Common Subsumer (LCS). By considering the depth of the LCS relative to the depths of the concepts themselves, it provides a measure of similarity that accounts for specificity. Concepts sharing a deeper LCS are considered more similar. While popular, it can sometimes produce counter-intuitive results depending on the ontology structure.
* **LCS Depth:** A simpler approach related to Wu & Palmer is to use the depth of the LCS itself as a similarity indicator: the deeper the LCS, the more specific the shared information, and thus the higher the similarity.
* **Information Content (IC) Based Measures:** These methods quantify the specificity of a concept based on its probability of occurrence in a corpus or its structural properties within the ontology (e.g., number of descendants). Common IC-based similarity measures (e.g., Resnik, Lin, Jiang & Conrath) typically use the IC of the concepts and their LCS. For example, Resnik similarity is simply the IC of the LCS. These measures can capture data-driven or structure-driven semantics effectively but depend on the availability of a suitable corpus or a well-structured ontology.

The user's choice of SPL prioritizes topological proximity over the semantic nuances captured by depth, specificity, and information content. While simpler to compute, SPL may fail to accurately reflect the semantic relatedness between CWEs, potentially leading to misleading evaluations where errors between semantically distant but topologically close nodes are penalized less than errors between semantically closer but topologically more distant nodes. Metrics like Wu & Palmer or IC-based approaches are generally considered more semantically grounded for hierarchical evaluation.

### 3.3. Relevance to CWE Hierarchy

The specific characteristics of the CWE hierarchy further challenge the suitability of simple SPL.

* **Structure:** CWE is organized as a DAG, not strictly a tree, meaning nodes can have multiple parents. It features multiple levels of abstraction (Pillar, Class, Base, Variant) representing different granularities of weaknesses. The hierarchy is also actively maintained and evolves.
* **Challenges for SPL:** The DAG structure means multiple paths may exist between nodes; SPL typically uses only the shortest, potentially ignoring semantically meaningful connections. The defined abstraction levels strongly imply non-uniform semantic distances between parent-child pairs across different levels, which SPL ignores. For example, the semantic jump from a Pillar to a Class might be larger than from a Base to a Variant.
* **Mapping Practices:** The National Vulnerability Database (NVD) and others mapping CVEs to CWEs sometimes use higher-level CWEs or map to specific "views" like View-1003 (a simplified mapping) for consistency or due to ambiguity. This practice complicates evaluation: if the ground truth is a specific 'Base' level CWE (e.g., CWE-787 Out-of-bounds Write), is predicting its 'Class' level parent (e.g., CWE-119 Improper Restriction of Operations within the Bounds of a Memory Buffer) partially correct? The user's metric assigns partial credit via $1/(1+d)$, but the amount is based solely on path length, not the semantic relationship or abstraction level difference. Metrics sensitive to depth and structure, or methods like set-augmentation (discussed later), might handle these nuances more appropriately.

## 4. Evaluation of the Proposed Aggregation Method

The method used to aggregate scores across multiple labels and instances is critical for producing meaningful overall performance metrics. The user's proposal employs a custom aggregation based on summed proximity scores, which needs comparison with standard practices.

### 4.1. Contrast with Standard Multi-Label Metrics

Standard evaluation in multi-label classification employs several well-defined metrics:

* **User Proposal:** A custom P/R/F1 calculation based on summing $1/(1+d)$ proximity scores, likely averaged at the example level first.
* **Standard Metrics:** These are broadly categorized:
    * **Example-Based Metrics:** Evaluate performance on an instance-by-instance basis. Key examples include:
        * **Subset Accuracy (Exact Match Ratio):** The strictest metric; requires the predicted set of labels to exactly match the true set. Score is 1 if identical, 0 otherwise. It gives no partial credit.
        * **Hamming Loss:** The fraction of labels that are incorrectly predicted (misclassifications + missed labels), averaged over instances. Lower is better. It treats all labels and errors equally.
        * **Example-Based P/R/F1:** Calculates precision, recall, and F1 for each example based on the intersection and union of predicted and true label sets (e.g., using Jaccard similarity for F1: $|P \cap Y| / |P \cup Y|$). Then averages these scores across examples.
    * **Ranking Metrics:** Treat the output as a ranked list of labels. Examples include Ranking Loss, Average Precision, and One-Error. These are useful when the order or confidence of predictions matters.
    * **Label-Based (Averaging) Metrics:** Adapt binary metrics (P/R/F1) to the multi-label setting by averaging across labels. The two main approaches are Micro and Macro averaging.

### 4.2. Micro vs. Macro Averaging

Micro and Macro averaging are standard ways to aggregate P/R/F1 scores in multi-class and multi-label settings:

* **Definitions:**
    * **Micro-average:** Aggregates the counts of True Positives (TP), False Positives (FP), and False Negatives (FN) across all classes/labels first. Then computes the metric (e.g., Precision = $\sum TP / (\sum TP + \sum FP)$) from these global counts.
    * **Macro-average:** Calculates the metric (e.g., Precision) independently for each class/label. Then computes the simple arithmetic mean of these per-class metrics.
* **Interpretability:**
    * **Micro-average:** Gives equal weight to each classification decision (or each instance in multi-label settings where metrics are computed per instance). It reflects overall performance across all predictions. In multi-class (single-label per instance) settings, Micro-F1 equals Accuracy.
    * **Macro-average:** Gives equal weight to each class/label, regardless of its frequency. It reflects the average performance across all classes.
* **Sensitivity to Imbalance:** This difference in weighting is crucial for imbalanced datasets, which are common in HMC. Micro-average scores are dominated by the performance on the majority classes. Macro-average scores treat rare classes and frequent classes equally, making it a better indicator of performance across the board, especially for identifying weaknesses in classifying minority classes. Weighted macro-averaging, which weights each class's metric by its support (number of true instances), offers a compromise.

The user's proposed aggregation method, likely averaging custom example-level scores, resembles Macro-averaging in spirit but applies it to non-standard base metrics. This lack of grounding in the standard TP/FP/FN counts used by Micro/Macro averaging makes the resulting values difficult to interpret statistically, especially regarding their behavior with class imbalance. It is unclear whether the user's metric reflects overall decision accuracy (like Micro) or average per-class performance (like Macro).

### 4.3. Contrast with Hierarchical Aggregation (hP/hR/hF)

To explicitly incorporate hierarchy into P/R/F1, the set-augmentation approach leading to hierarchical Precision (hP), Hierarchical Recall (hR), and Hierarchical F-measure (hF) is a recognized standard.

* **Set-Augmentation (hP/hR/hF):** This approach modifies the sets of true labels (Y) and predicted labels (P) for a given instance before calculating standard P/R/F1. Both sets are augmented by adding all their respective ancestor nodes in the hierarchy.
    Let $An(c)$ be the set of ancestors of class $c$.
    Augmented true set:
    $$ Y_{aug} = Y \cup \bigcup_{y \in Y} An(y) $$
    Augmented predicted set:
    $$ P_{aug} = P \cup \bigcup_{p \in P} An(p) $$
    Hierarchical Precision:
    $$ hP = \frac{|P_{aug} \cap Y_{aug}|}{|P_{aug}|} $$
    Hierarchical Recall:
    $$ hR = \frac{|P_{aug} \cap Y_{aug}|}{|Y_{aug}|} $$
    Hierarchical F-measure:
    $$ hF = \frac{2 \times hP \times hR}{hP + hR} $$
* **Averaging hP/hR/hF:** These example-wise hierarchical metrics are then typically aggregated across the dataset using standard Micro or Macro averaging. This yields metrics like Micro-hF and Macro-hF, which combine hierarchical awareness (via set augmentation) with standard, interpretable aggregation methods.

**Comparison:** The hP/hR/hF approach integrates the hierarchy by redefining what constitutes a relevant prediction (considering ancestors), but then applies the standard, well-understood definitions of P/R based on set intersections and cardinalities. The user's method, conversely, keeps the notion of prediction sets standard but defines custom P/R based on summed, distance-derived proximity scores. The hP/hR/hF approach is arguably more integrated with the established evaluation framework.

### 4.4. Interpretability of User's P/R Metrics

A major drawback of the user's proposal is the lack of clear interpretability of the resulting P/R/F1 scores.

* **Lack of Standard Meaning:** Standard Precision represents the probability that a positive prediction is correct; standard Recall represents the probability that a true positive instance is correctly identified. It is unclear what probabilistic or set-theoretic meaning can be ascribed to the user's metrics derived from summing $1/(1+d)$ scores. A score of, say, 0.7 for the user's "Precision" cannot be directly interpreted in the same way as a standard Precision score of 0.7.
* **Sensitivity:** The scores are sensitive to the specific distance function ($d$) and the transformation ($s=1/(1+d)$). The choice of arbitrary distances $d_{sibling}$ and $d_{unrelated}$ can significantly impact the final score without a clear semantic justification. The non-linear decay of the proximity score also influences how errors at different distances contribute to the sum.
* **Comparability Issues:** The use of non-standard, custom metrics severely hinders comparability with other research and benchmark results. Established metrics like Micro-F1, Macro-F1, Micro-hF, and Macro-hF serve as common ground for evaluating HMC systems. Results based on the proposed custom metrics cannot be readily compared, making it difficult to assess relative performance or progress.

## 5. Assessment of the Partial Credit Mechanism

A key motivation for hierarchical evaluation is to award partial credit for predictions that are incorrect but "close" in the hierarchy.

### 5.1. User Proposal: Proximity Score $1/(1+d)$

* **Mechanism:** The formula $s=1/(1+d)$ directly implements a continuous partial credit scheme. The closer the prediction (smaller $d$), the higher the score $s$ (approaching 1), and vice versa. Credit diminishes non-linearly with distance.
* **Nature:** Explicit, continuous, distance-based partial credit.

### 5.2. Research Standard: Set-Augmentation (hP/hR/hF)

* **Mechanism:** The hP/hR/hF measures provide partial credit implicitly through the set augmentation process. When calculating the intersection $|P_{aug} \cap Y_{aug}|$, a predicted node $p$ contributes positively if it, or one of its ancestors, matches a true node $y$ or one of its ancestors. Predicting a parent, grandparent, or even a sibling (if they share a close common ancestor included in the augmented sets) of a true label can result in a non-zero intersection, thus contributing to the hP and hR scores.
* **Nature:** Implicit, set-overlap based partial credit, grounded in shared hierarchical context.

### 5.3. Comparison

* **Granularity:** The user's method provides a seemingly finer-grained partial credit score based on the exact distance value $d$. The hP/hR/hF mechanism is based on set membership overlap; the contribution to the intersection size is effectively binary for each element in the augmented sets, but the overall score reflects the degree of overlap.
* **Penalty for Large Errors:** How severely are very distant errors penalized? In the user's method, $s=1/(1+d)$ approaches zero as $d$ increases but never reaches it unless $d$ is infinite. This might assign small amounts of credit even to predictions that are semantically very far away, depending on the scale of $d$ and the values chosen for $d_{unrelated}$. In the hP/hR/hF approach, if a predicted node $p$ and a true node $y$ share no ancestors other than perhaps the root (which might be excluded from augmentation depending on the implementation), their augmented sets $An(p) \cup \{p\}$ and $An(y) \cup \{y\}$ might have minimal or no overlap. In such cases, the incorrect prediction $p$ contributes nothing towards matching $y$ in the intersection calculation, effectively receiving zero credit relative to that specific true label. This aligns with the intuition that grossly incorrect predictions should be heavily penalized. The $1/(1+d)$ function's decay might not be steep enough to adequately penalize such errors.
* **Interpretability:** The partial credit in hP/hR/hF arises naturally from applying the standard P/R definitions to hierarchically augmented sets. It fits within the well-understood framework of true positives, false positives, and false negatives, albeit defined over augmented sets. The user's $1/(1+d)$ score, while intuitive as a concept ("closer is better"), is an ad-hoc value whose contribution to the final non-standard P/R metrics lacks a clear, established interpretation within classification evaluation theory.

While the user's continuous score appears nuanced, the set-augmentation method integrates partial credit more coherently with the standard Precision/Recall framework, offering better interpretability and likely a more appropriate handling of severe errors.

## 6. Summary of Pros of the User's Proposal

Despite its limitations when compared to established research methods, the user's proposed formula exhibits several positive attributes:

* **Acknowledges Hierarchy:** The proposal explicitly recognizes the hierarchical nature of the CWE classification task and attempts to incorporate this structure into the evaluation, moving beyond inadequate flat classification metrics.
* **Handles Multi-Label:** The design considers the multi-label aspect, where a single CVE can map to multiple CWEs, aiming to evaluate predictions against potentially multiple true labels.
* **Intuitive Proximity Score:** The conversion of distance $d$ to a proximity score $s$ via $s=1/(1+d)$ is conceptually straightforward and aligns with the intuition that predictions closer in the hierarchy are better than those farther away.
* **Provides Partial Credit:** Unlike strict metrics like Subset Accuracy, the formula incorporates a mechanism for awarding partial credit, acknowledging that some incorrect predictions are "less wrong" than others based on hierarchical proximity.
* **Simplicity (Distance Metric):** The use of Shortest Path Length (SPL) as the distance metric is computationally simple and easy to implement compared to more complex semantic similarity measures that might require external corpora or intricate structural analysis.

## 7. Summary of Cons of the User's Proposal

The proposed formula suffers from several significant drawbacks when evaluated against established HMC evaluation principles and research findings:

* **Simplistic Distance Metric:** The core SPL distance metric ignores crucial semantic information embedded in the hierarchy, such as node depth (specificity) and the varying semantic distances between different levels of abstraction. Treating all hierarchical links as having uniform weight is a major oversimplification for complex taxonomies like CWE.
* **Arbitrary Parameters:** The reliance on user-defined, fixed distance values for sibling ($d_{sibling}$) and unrelated ($d_{unrelated}$) nodes introduces arbitrariness. The metric's behavior becomes sensitive to these parameters, which lack a clear theoretical or empirical justification.
* **Non-Standard Aggregation:** The custom method for calculating Precision, Recall, and F1-score based on summing proximity scores deviates significantly from standard, well-defined aggregation techniques like Micro and Macro averaging.
* **Unclear Interpretability:** The resulting P/R/F1 scores lack a clear statistical or probabilistic interpretation. It is difficult to understand what aspect of performance they truly measure or how they behave under conditions like class imbalance, making them hard to interpret meaningfully.
* **Comparability Issues:** The use of non-standard metrics makes it virtually impossible to compare results obtained using this formula with those from other studies or benchmarks that employ established HMC metrics like hF (Micro/Macro averaged).
* **Potentially Insufficient Penalty for Large Errors:** The gentle decay of the $1/(1+d)$ score might not sufficiently penalize predictions that are hierarchically very distant from the true labels, potentially inflating scores compared to methods like hP/hR/hF where such errors contribute minimally or not at all to the positive counts.
* **Ignores Edge Weights/Types:** The underlying SPL assumes uniform edge weights and doesn't account for potentially different relationship types within the CWE structure, further limiting its semantic accuracy.

## 8. Analysis of User's "ToDo" Items

The user identified two potential areas for future improvement: weighting relationships and incorporating a beta parameter in the F-score.

### 8.1. Weighting Relationships

* **User Intent:** This suggestion aims to address the limitation of assuming uniform distance for all hierarchical links by assigning different weights based on the relationship type (e.g., parent-child vs. sibling) or potentially the depth.
* **Research Context:** This aligns with the critique of basic path-based measures that treat all edges equivalently. More advanced graph-based semantic measures sometimes incorporate edge weighting schemes.
* **Evaluation:** Introducing weights is a step towards acknowledging non-uniform semantic distances. However, applying weights to a simple SPL calculation still does not fundamentally address the more critical issue of ignoring node depth and specificity, which metrics like Wu & Palmer or IC-based measures handle more directly. Furthermore, it introduces additional parameters (the weights themselves) that require careful justification and tuning, potentially adding complexity without guaranteeing a more semantically sound metric compared to established alternatives like Wu & Palmer or the set-augmentation approach. It refines the SPL approach but doesn't replace it with a more robust foundation.

### 8.2. Beta Parameter in F-Score

* **User Intent:** The proposal suggests incorporating a beta parameter, presumably to create an F-beta score analogous to the standard definition:
    $$ F_\beta = \frac{(1 + \beta^2) \times P \times R}{(\beta^2 \times P) + R} $$
    This allows tuning the F-score to prioritize either Precision ($\beta<1$) or Recall ($\beta>1$).
* **Research Context:** Using the F-beta score is a standard technique in classification evaluation when there is an asymmetric cost associated with false positives versus false negatives, or a specific need to emphasize either precision or recall.
* **Evaluation:** While incorporating a beta parameter is standard practice for balancing P and R, its utility depends entirely on the meaningfulness of the underlying P and R metrics. Applying it to the user's custom, non-standard P and R values does not fix the core problems associated with their calculation and interpretation. The fundamental issues lie in the distance metric and the aggregation method used to derive P and R, not in how they are combined into an F-score. Improving the P and R calculations themselves is far more critical than allowing for a beta-weighted combination of potentially flawed metrics.

### 8.3. Addressing Core Limitations?

The proposed "ToDo" items demonstrate an awareness of some of the formula's limitations, specifically the uniform edge weight assumption and the fixed balance of the F1 score. However, they represent incremental adjustments to the existing framework rather than a shift towards fundamentally different, research-backed methodologies. They fail to address the core weaknesses identified in Sections 3 and 4: the semantic inadequacy of the SPL distance metric (ignoring depth/specificity) and the non-standard, uninterpretable aggregation method for P/R/F1. Therefore, implementing these suggestions would likely yield only marginal improvements and would not resolve the fundamental issues of interpretability, comparability, and semantic relevance compared to established HMC evaluation practices.

## 9. Proposed Improvements Based on Research

Based on the analysis and comparison with established HMC evaluation research, the following improvements are recommended to create a more robust, interpretable, and comparable evaluation framework for the CWE classification task.

### 9.1. Adopt Standard Hierarchical Metrics (hP/hR/hF)

* **Recommendation:** Replace the custom P/R/F1 calculations based on proximity scores with the standard Hierarchical Precision (hP), Hierarchical Recall (hR), and Hierarchical F-measure (hF).
* **Rationale:** These metrics are specifically designed for HMC. They incorporate the hierarchy through set augmentation (adding ancestors to true and predicted sets) and then apply the standard, well-understood definitions of precision and recall based on the intersection and cardinality of these augmented sets. This approach provides inherent partial credit for predictions that share hierarchical context (ancestors) with the true labels. It directly addresses the need for hierarchy-aware evaluation within a standard P/R framework, overcoming the interpretability issues of the user's custom scores. These metrics are increasingly used in HMC literature, facilitating comparison.

### 9.2. Employ Standard Aggregation Techniques (Micro/Macro Averaging)

* **Recommendation:** Calculate the example-wise hP, hR, and hF scores derived from set augmentation. Then, aggregate these scores across the entire dataset using both Micro-averaging and Macro-averaging. Report both Micro-hF and Macro-hF.
* **Rationale:** Micro and Macro averages provide distinct and valuable perspectives on performance. Micro-hF reflects the overall performance weighted by individual predictions, while Macro-hF reflects the average performance per class, treating all classes equally. Reporting both is crucial for understanding performance on potentially imbalanced datasets like CWE, where performance on rare but critical CWEs might be obscured by Micro-averages but highlighted by Macro-averages. Using these standard aggregation methods ensures results are interpretable and comparable to other studies.

### 9.3. Consider More Robust Distance/Similarity Metrics (If Needed for Other Purposes)

* **Recommendation:** If a distance or similarity measure is needed for purposes beyond the primary P/R/F evaluation (e.g., as part of a loss function during model training, or for specific error analysis), replace SPL with a metric that better captures semantic similarity within the hierarchy.
* **Options:**
    * **Wu & Palmer Similarity:** Incorporates node depth and LCS depth, offering a good balance between structural information and computational feasibility. Requires access to the hierarchy structure and node depths.
    * **LCS-based Measures:** Focus on the properties (depth or Information Content) of the Least Common Subsumer, directly measuring shared specificity.
    * **Information Content (IC) Based Measures:** Quantify specificity based on corpus statistics or structural properties (e.g., descendant counts). Can provide strong semantic grounding but require additional data or assumptions about the ontology structure.
* **Rationale:** These metrics address the core limitations of SPL by incorporating information about node depth, specificity, and/or information content, leading to more semantically meaningful comparisons between CWE nodes. The choice among them depends on the specific application requirements and data availability. However, for the primary evaluation metrics, hP/hR/hF based on set augmentation is generally preferred over distance-based aggregation.

### 9.4. Comparative Summary

The following table summarizes the key differences between the user's proposal and the recommended approach based on hP/hR/hF with standard averaging:

| Feature         | User's Proposed Formula                                  | Recommended Approach (hP/hR/hF + Micro/Macro Avg)                  | Rationale / References                                                                     |
| :-------------- | :------------------------------------------------------- | :----------------------------------------------------------------- | :----------------------------------------------------------------------------------------- |
| Distance Metric | Shortest Path Length (SPL)                               | Implicit (via set overlap)                                         | SPL ignores depth/specificity. Set augmentation captures shared hierarchical context.         |
| Partial Credit  | Explicit, continuous via $s=1/(1+d)$                     | Implicit, via overlap of augmented sets ($P_{aug} \cap Y_{aug}$)   | Set overlap integrates better with P/R framework, potentially penalizes large errors more appropriately. |
| Aggregation (P/R) | Custom, sum/average of proximity scores ($s$)            | Standard P/R definitions applied to augmented sets ($P_{aug}, Y_{aug}$) | Standard definitions on augmented sets provide clear meaning. User's method lacks standard interpretation. |
| Overall Aggregation | Average of custom example-level scores (Macro-like)    | Standard Micro- and Macro-averaging of example-level hP/hR/hF      | Micro/Macro averages have well-defined interpretations, handle imbalance transparently, ensure comparability. |
| Interpretability| Low; scores lack standard meaning, sensitive to params   | High; based on standard P/R/F1 framework, Micro/Macro well-understood | Standard metrics allow clear interpretation and comparison with literature.             |
| Standardization | Non-standard                                             | Based on established HMC evaluation practices                      | Adherence to standards is crucial for scientific rigor and comparability.                |

## 10. Conclusion

This report conducted a comparative analysis of a user-proposed scoring formula for evaluating hierarchical multi-label CWE classification against established research findings and practices. The analysis reveals that while the proposal commendably attempts to incorporate both the hierarchical structure of CWE and the multi-label nature of CVE-to-CWE mapping, it relies on methods that deviate significantly from research-backed standards.

The strengths of the proposal lie in its explicit acknowledgment of the HMC problem, its intuitive mechanism for assigning proximity scores based on distance, and the inclusion of partial credit. However, these are overshadowed by significant weaknesses. The core distance metric, Shortest Path Length, is overly simplistic, ignoring crucial semantic information like node depth and specificity, which are vital in hierarchical contexts. Furthermore, the reliance on arbitrary parameters for sibling/unrelated distances and the custom aggregation method for Precision, Recall, and F1-score result in metrics that lack clear statistical interpretation and are difficult to compare with standard benchmarks. The proposed "ToDo" items, while showing awareness of some limitations, do not address these fundamental issues.

Based on the analysis of relevant research, the recommended approach is to adopt standard HMC evaluation metrics. Specifically, using Hierarchical Precision (hP), Hierarchical Recall (hR), and Hierarchical F-measure (hF) calculated via set augmentation provides a principled way to incorporate the hierarchy and award partial credit within the standard P/R framework. Aggregating these example-wise scores using both Micro- and Macro-averaging ensures interpretable results that account for potential class imbalance and allow for comparison with other studies. If distance-based measures are required for other analyses, metrics like Wu & Palmer similarity offer more semantic richness than simple path length.

Adopting these standard, research-backed evaluation practices will lead to more rigorous, interpretable, and comparable assessments of automated CWE classification systems. While developing novel metrics can be valuable, it is crucial to ensure they are well-grounded theoretically and validated against established methods, particularly in complex evaluation scenarios like HMC. Challenges remain in HMC evaluation, especially concerning the specific complexities of the CWE DAG structure and evolving mapping practices, but leveraging established metrics like hP/hR/hF provides a solid foundation.

## 11. References


* https://www.mdpi.com/2078-2489/12/8/298
* https://www.uh.edu/cbl/_files/publications/PR-2017-LZ.pdf
* https://graus.nu/blog/ontology-based-semantic-similarity-measurements-an-overview/
* https://www.kaggle.com/datasets/krooz0/cve-and-cwe-mapping-dataset
* https://content.iospress.com/articles/semantic-web/sw222919
* https://www.geeksforgeeks.org/precision-recall-and-f1-score-using-r/
* https://paperswithcode.com/task/hierarchical-multi-label-classification
* https://pmc.ncbi.nlm.nih.gov/articles/PMC3603583/
* https://cwe.mitre.org/documents/schema/schema_v5.0.html
* https://scikit-learn.org/stable/auto_examples/model_selection/plot_precision_recall.html
* https://www.researchgate.net/publication/243458423_Evaluation_Measures_for_Hierarchical_Classification_a_unified_view_and_novel_approaches
* https://cwe.mitre.org/about/faq.html
* https://phoenix.security/the-cve-nvd-crisis-a-wake-up-call-for-application-security/
* https://www.cs.kent.ac.uk/people/staff/aaf/pub_papers.dir/DMKD-J-2010-Silla.pdf
* https://arxiv.org/abs/1306.6802
* https://arxiv.org/pdf/1306.6802
* https://cwe.mitre.org/data/definitions/1276.html
* https://aclanthology.org/2022.acl-long.399.pdf
* https://proceedings.neurips.cc/paper/2020/file/6dd4e10e3296fa63738371ec0d5df818-Paper.pdf
* https://www.scitepress.org/Papers/2024/127704/127704.pdf
* https://cwe.mitre.org/data/index.html
* https://truefort.com/manage-cwe-vs-cve-vulnerabilities-weaknesses/
* https://cwe.mitre.org/data/definitions/1062.html
* https://www.researchgate.net/publication/270337594_A_Tutorial_on_Multi-Label_Learning
* https://sig-synopsys.my.site.com/community/s/article/What-is-the-difference-between-CWE-Type-and-Related-CWE-Types
* https://www.spinnakersupport.com/blog/2023/12/22/cve-vs-cwe/
* https://cwe.mitre.org/documents/schema/
* https://www.scirp.org/journal/paperinformation?paperid=74042
* https://cwe.mitre.org/top25/archive/2024/2024_methodology.html
* https://www.numberanalytics.com/blog/practical-guide-hamming-loss-multi-label-evaluation
* https://cwe.mitre.org/data/definitions/588.html
* https://www.kaggle.com/code/kmkarakaya/multi-label-model-evaluation/notebook
* https://hiclass.readthedocs.io/en/v4.13.1/algorithms/multi_label.html
* https://www.kiuwan.com/blog/cwe-common-weakness-enumeration/
* https://www.lacework.com/cloud-security-fundamentals/nvd-what-is-the-national-vulnerability-database
* https://www.balbix.com/insights/what-is-a-cve/
* https://cdn.aaai.org/Workshops/2007/WS-07-05/WS07-05-001.pdf
* https://xygeni.io/blog/cve-vs-cwe-key-differences-explained/
* https://arxiv.org/abs/1306.6802
* https://towardsdatascience.com/precision-and-recall-88a3776c8007/
* https://developers.google.com/machine-learning/crash-course/classification/accuracy-precision-recall
* https://cwe.mitre.org/data/definitions/1086.html
* https://www.codiga.io/blog/cve-vs-cwe/
* https://www.iriusrisk.com/resources-blog/understanding-cwes-and-cves-and-how-they-impact-your-threat-models
* https://danaepp.com/level-up-your-vulnerability-reports-with-cwe
* https://pmc.ncbi.nlm.nih.gov/articles/PMC3041307/
* https://zencoder.ai/blog/cve-vs-cwe
* https://cwe.mitre.org/documents/schema/
* https://nvd.nist.gov/vuln/data-feeds
* https://hiclass.readthedocs.io/en/v4.13.1/algorithms/multi_label.html
* https://docs.contrastsecurity.com/en/find-cves-from-cwes.html
* https://aclanthology.org/2021.emnlp-main.69.pdf
* https://academic.oup.com/bib/article/23/4/bbac216/6611916
* https://arxiv.org/pdf/1306.6802
* https://www.mdpi.com/2079-9292/13/7/1199
* https://users.iit.demokritos.gr/~paliourg/old/papers/DMKD2015.pdf
* https://datascience.stackexchange.com/questions/15989/micro-average-vs-macro-average-performance-in-a-multiclass-classification-settin
* https://datascience.stackexchange.com/questions/85981/micro-average-vs-macro-average-for-class-imbalance
* https://sklearn-evaluation.ploomber.io/en/latest/classification/micro_macro.html
* https://safjan.com/micro-and-macro-averages-in-multiclass-multilabel-problems/
* https://towardsdatascience.com/evaluating-multi-label-classifiers-a31be83da6ea/
* https://towardsdatascience.com/micro-macro-weighted-averages-of-f1-score-clearly-explained-b603420b292f/
* https://hiclass.readthedocs.io/en/v4.9.0/algorithms/multi_label.html
* https://www.amazon.science/blog/using-generative-ai-to-improve-extreme-multilabel-classification
* https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00675/122720/A-Closer-Look-at-Classification-Evaluation-Metrics
* https://stackoverflow.com/questions/67848456/when-do-micro-and-macro-averages-differ-a-lot
* https://arxiv.org/pdf/1211.4709#:~:text=from%20the%20root%20node%20and,*N%2F(N1%2BN2
* http://profs.etsmtl.ca/ctadj/papers/2-conf/2016/Djamel-ubicomm_2016_3_10_10130.pdf
* https://arxiv.org/pdf/1211.4709
* https://www.scirp.org/journal/paperinformation?paperid=74042
* https://www.researchgate.net/publication/242658963_A_New_Similarity_Measure_based_on_Edge_Counting
* https://pmc.ncbi.nlm.nih.gov/articles/PMC3603583/
* https://www.researchgate.net/publication/310572659_A_modification_of_Wu_and_Palmer_Semantic_Similarity_Measure
* https://www.mdpi.com/2076-3417/13/21/11959
* https://arxiv.org/pdf/1401.4603
* https://academic.oup.com/bioinformatics/article/32/9/1380/1743954
* https://arxiv.org/html/2502.11070v1
* https://par.nsf.gov/servlets/purl/10385264
* https://cwe.mitre.org/data/definitions/1003.html
* https://baolingfeng.github.io/papers/ICSE2023.pdf
