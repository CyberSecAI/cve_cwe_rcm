# CVE-CWE Benchmark Comparison Algorithm Requirements

> [!TIP]
> This document outlines the requirements for the algorithm that evaluates automated CWE assignments against the gold-standard benchmark dataset.

## Table of Contents

- [CVE-CWE Benchmark Comparison Algorithm Requirements](#cve-cwe-benchmark-comparison-algorithm-requirements)
  - [Table of Contents](#table-of-contents)
  - [Algorithm Core Requirements](#algorithm-core-requirements)
  - [CWE Matching Requirements](#cwe-matching-requirements)
  - [Multiple CWE Handling Requirements](#multiple-cwe-handling-requirements)
  - [Abstraction Level Requirements](#abstraction-level-requirements)
  - [Evaluation Metrics Requirements](#evaluation-metrics-requirements)
  - [Grounding and Low-Information Requirements](#grounding-and-low-information-requirements)
  - [Implementation Requirements](#implementation-requirements)
  - [References](#references)

## Algorithm Core Requirements

<a id="REQ_ALGO_MULTI_LABEL"></a>**REQ_ALGO_MULTI_LABEL**: The comparison algorithm MUST treat CWE assignment as a multi-label classification problem.

<a id="REQ_ALGO_HIERARCHICAL"></a>**REQ_ALGO_HIERARCHICAL**: The comparison algorithm MUST incorporate the hierarchical nature of CWEs in its matching logic.

<a id="REQ_ALGO_PARTIAL_CREDIT"></a>**REQ_ALGO_PARTIAL_CREDIT**: The comparison algorithm MUST support partial credit for related but non-exact CWE matches.

<a id="REQ_ALGO_CONFIGURABLE"></a>**REQ_ALGO_CONFIGURABLE**: The comparison algorithm MUST be configurable to allow different weighting schemes for different types of matches.

<a id="REQ_ALGO_EXPLAINABLE"></a>**REQ_ALGO_EXPLAINABLE**: The comparison algorithm MUST provide explanations for why specific matches were scored in a particular way.

## CWE Matching Requirements

<a id="REQ_MATCH_EXACT"></a>**REQ_MATCH_EXACT**: The algorithm MUST identify and give full credit (score = 1.0) for exact CWE ID matches.

<a id="REQ_MATCH_PARENT_CHILD"></a>**REQ_MATCH_PARENT_CHILD**: The algorithm MUST identify and score direct parent-child relationships in the CWE hierarchy (e.g., score = 0.8-0.9).

<a id="REQ_MATCH_ANCESTOR"></a>**REQ_MATCH_ANCESTOR**: The algorithm MUST identify and score indirect ancestor-descendant relationships (e.g., grandparent, score = 0.5-0.7).

<a id="REQ_MATCH_COUSIN"></a>**REQ_MATCH_COUSIN**: The algorithm MUST identify and score "cousin" relationships where CWEs share a common parent or grandparent (e.g., score = 0.4-0.6).

<a id="REQ_MATCH_UNRELATED"></a>**REQ_MATCH_UNRELATED**: The algorithm MUST identify unrelated CWEs (different branches with no close common ancestor) and give no credit (score = 0).

<a id="REQ_MATCH_CUSTOMIZABLE"></a>**REQ_MATCH_CUSTOMIZABLE**: The algorithm MUST allow customizable weights for each degree of match to compute similarity scores.

<a id="REQ_MATCH_CWE_GRAPH"></a>**REQ_MATCH_CWE_GRAPH**: The algorithm MUST use the official CWE graph or equivalent data structure to determine relationships between CWEs.

<a id="REQ_MATCH_RELATIONSHIP_TYPES"></a>**REQ_MATCH_RELATIONSHIP_TYPES**: The algorithm SHOULD consider CWE relationship types beyond ParentOf/ChildOf, such as PeerOf, CanPrecede/CanFollow, and RequiredBy/Requires when available.

## Multiple CWE Handling Requirements

<a id="REQ_MULTI_PAIRWISE"></a>**REQ_MULTI_PAIRWISE**: The algorithm MUST implement pairwise matching between elements of predicted and gold CWE sets.

<a id="REQ_MULTI_OPTIMAL"></a>**REQ_MULTI_OPTIMAL**: The algorithm MUST ensure each gold CWE is matched to at most one predicted CWE and vice versa to avoid double-counting.

<a id="REQ_MULTI_BEST_MATCH"></a>**REQ_MULTI_BEST_MATCH**: For cases where multiple matches are possible, the algorithm MUST select the match that yields the highest similarity score.

<a id="REQ_MULTI_SET_BASED"></a>**REQ_MULTI_SET_BASED**: The algorithm MUST support set-based comparison metrics for evaluating full sets of CWEs per CVE.

<a id="REQ_MULTI_LABEL_BASED"></a>**REQ_MULTI_LABEL_BASED**: The algorithm MUST support label-based metrics for evaluating individual CWE predictions across all CVEs.

## Abstraction Level Requirements

<a id="REQ_ABSTRACT_TRACKING"></a>**REQ_ABSTRACT_TRACKING**: The algorithm MUST track the abstraction level (Pillar, Class, Base, Variant) of each CWE.

<a id="REQ_ABSTRACT_FILTERING"></a>**REQ_ABSTRACT_FILTERING**: The algorithm MUST support filtering or grouping results by abstraction level.

<a id="REQ_ABSTRACT_SEPARATE_METRICS"></a>**REQ_ABSTRACT_SEPARATE_METRICS**: The algorithm MUST calculate metrics separately for different abstraction levels when requested.

<a id="REQ_ABSTRACT_MISMATCH"></a>**REQ_ABSTRACT_MISMATCH**: The algorithm MUST identify and report when predictions match at a different abstraction level than the gold standard.

<a id="REQ_ABSTRACT_PREFERENCE"></a>**REQ_ABSTRACT_PREFERENCE**: The algorithm SHOULD implement a configurable preference for Base/Variant level matches over Class/Pillar matches, in accordance with CWE mapping guidance.

## Evaluation Metrics Requirements

<a id="REQ_METRIC_EXACT_MATCH"></a>**REQ_METRIC_EXACT_MATCH**: The algorithm MUST calculate and report the Exact Match Rate (EMR) - the proportion of CVEs where predicted CWE sets exactly match gold CWE sets.

<a id="REQ_METRIC_AT_LEAST_ONE"></a>**REQ_METRIC_AT_LEAST_ONE**: The algorithm MUST calculate and report the At Least One Match Rate - the proportion of CVEs where at least one gold CWE was correctly predicted.

<a id="REQ_METRIC_PRECISION"></a>**REQ_METRIC_PRECISION**: The algorithm MUST calculate and report Precision (Positive Predictive Value) based on true and false positives.

<a id="REQ_METRIC_RECALL"></a>**REQ_METRIC_RECALL**: The algorithm MUST calculate and report Recall (Sensitivity) based on true positives and false negatives.

<a id="REQ_METRIC_F1"></a>**REQ_METRIC_F1**: The algorithm MUST calculate and report the F1 Score as the harmonic mean of precision and recall.

<a id="REQ_METRIC_BALANCED_ACCURACY"></a>**REQ_METRIC_BALANCED_ACCURACY**: The algorithm MUST calculate and report Balanced Accuracy, accounting for class imbalance by averaging recall across CWE classes.

<a id="REQ_METRIC_SOFT_VERSIONS"></a>**REQ_METRIC_SOFT_VERSIONS**: The algorithm MUST support "soft" versions of precision, recall, and F1 that incorporate partial match scores.

<a id="REQ_METRIC_BREAKDOWN"></a>**REQ_METRIC_BREAKDOWN**: The algorithm SHOULD provide breakdowns of metrics by CWE frequency, abstraction level, or other relevant criteria.

<a id="REQ_METRIC_MACRO_MICRO"></a>**REQ_METRIC_MACRO_MICRO**: The algorithm SHOULD calculate both micro and macro averages for precision, recall, and F1 when appropriate.

<a id="REQ_METRIC_CONFIDENCE"></a>**REQ_METRIC_CONFIDENCE**: The algorithm SHOULD incorporate confidence intervals or uncertainty measures for the reported metrics when sample sizes permit.

## Grounding and Low-Information Requirements

<a id="REQ_GROUND_HALLUCINATION"></a>**REQ_GROUND_HALLUCINATION**: The algorithm MUST implement detection for "hallucinated" CWE assignments that have no apparent support in the CVE description.

<a id="REQ_GROUND_KEYWORD"></a>**REQ_GROUND_KEYWORD**: The algorithm SHOULD use keyword/term matching to check if a CWE's characteristic terms appear in the CVE description.

<a id="REQ_GROUND_SEMANTIC"></a>**REQ_GROUND_SEMANTIC**: The algorithm SHOULD use semantic similarity techniques to assess if predictions are related to the CVE text content.

<a id="REQ_GROUND_SEPARATE_METRICS"></a>**REQ_GROUND_SEPARATE_METRICS**: The algorithm MUST report the percentage of predictions flagged as potential hallucinations.

<a id="REQ_LOW_INFO_EXCLUDE"></a>**REQ_LOW_INFO_EXCLUDE**: The algorithm MUST support excluding low-information CVEs from primary evaluation metrics.

<a id="REQ_LOW_INFO_SEPARATE"></a>**REQ_LOW_INFO_SEPARATE**: The algorithm MUST report metrics with and without low-information CVEs to show their impact on overall performance.

## Implementation Requirements

<a id="REQ_IMPL_PERFORMANCE"></a>**REQ_IMPL_PERFORMANCE**: The algorithm MUST be efficient enough to process large CVE datasets (tens of thousands of entries) in a reasonable time.

<a id="REQ_IMPL_OUTPUT_FORMAT"></a>**REQ_IMPL_OUTPUT_FORMAT**: The algorithm MUST produce structured output in a machine-readable format (e.g., JSON) with clear organization of metrics and results.

<a id="REQ_IMPL_DETAILED_RESULTS"></a>**REQ_IMPL_DETAILED_RESULTS**: The algorithm MUST provide detailed per-CVE results showing matches, scores, and classification decisions.

<a id="REQ_IMPL_SUMMARY_STATS"></a>**REQ_IMPL_SUMMARY_STATS**: The algorithm MUST generate summary statistics and aggregate metrics for the entire evaluation.

<a id="REQ_IMPL_VERSION_INFO"></a>REQ_IMPL_VERSION_INFO: The algorithm output report MUST include version information of: 1) the benchmark dataset used, 2) the comparison algorithm version, and 3) a timestamp indicating when the comparison was run.

<a id="REQ_IMPL_LOGGING"></a>REQ_IMPL_LOGGING: The algorithm MUST produce a detailed log file capturing the execution process, including configuration settings used, processing steps, any warnings or errors encountered, and summary of results. This log file MUST be separate from the main output report and should provide sufficient information for debugging and audit purposes.

<a id="REQ_IMPL_VISUALIZATION"></a>**REQ_IMPL_VISUALIZATION**: The algorithm SHOULD generate visualizations of results such as confusion matrices, precision-recall curves, or match distributions.

<a id="REQ_IMPL_DOCUMENTABILITY"></a>**REQ_IMPL_DOCUMENTABILITY**: The algorithm MUST be well-documented, with clear explanations of all metrics, weighting schemes, and decision processes.

<a id="REQ_IMPL_REPRODUCIBILITY"></a>**REQ_IMPL_REPRODUCIBILITY**: The algorithm MUST produce reproducible results given the same input data and configuration parameters.

<a id="REQ_IMPL_EXTENSIBILITY"></a>**REQ_IMPL_EXTENSIBILITY**: The algorithm SHOULD be designed to be extensible for future CWE versions, additional metrics, or enhanced matching techniques.

<a id="REQ_IMPL_SEMVER"></a>REQ_IMPL_SEMVER: The algorithm implementation and its scripts MUST use Semantic Versioning to clearly indicate compatibility and feature changes between releases.

## References

<a id="REF_CWE_GUIDANCE"></a>**REF_CWE_GUIDANCE**: [CWE - CVE → CWE Mapping "Root Cause Mapping" Guidance](https://cwe.mitre.org/documents/cwe_usage/guidance.html)

<a id="REF_RFC_2119"></a>**REF_RFC_2119**: [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) for definitions of "SHOULD", "MUST", etc.

<a id="REF_QUALITAGGER"></a>**REF_QUALITAGGER**: [QualiTagger: Automating software quality detection in issue trackers](https://arxiv.org/html/2504.11053v1)

<a id="REF_BALANCED_ACCURACY"></a>**REF_BALANCED_ACCURACY**: [balanced_accuracy_score — scikit-learn 1.6.1 documentation](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.balanced_accuracy_score.html)