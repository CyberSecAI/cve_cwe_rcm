
# A Hierarchical Scoring System for Evaluating Automated CVE-to-CWE Assignments

## 1. Introduction

### 1.1. Context: The Imperative for Automated CVE-to-CWE Mapping

The landscape of cybersecurity is characterized by a continuously expanding volume of publicly disclosed vulnerabilities, cataloged as Common Vulnerabilities and Exposures (CVEs). Effectively managing these vulnerabilities necessitates not only identifying their existence but also understanding their fundamental nature and root causes, which are categorized by the Common Weakness Enumeration (CWE) system. Linking CVEs to their corresponding CWEs provides crucial context for risk assessment, prioritization of remediation efforts, and informing secure software development practices to prevent similar flaws in the future.

However, the sheer scale of vulnerability disclosure, potentially involving thousands of new CVEs each month, renders manual CVE-to-CWE mapping a resource-intensive, time-consuming, and potentially inconsistent process. This operational bottleneck underscores the critical need for automated solutions capable of accurately and efficiently assigning CWE identifiers to CVE entries. Furthermore, CWEs serve a vital role beyond reactive vulnerability management; they provide a common language and framework for developers, architects, and security professionals to proactively address the root causes of vulnerabilities during the software development lifecycle (SDLC), shifting focus from mere symptom management (fixing CVEs) to systemic prevention (eliminating CWEs).

### 1.2. Problem Statement: Limitations of Traditional Evaluation

Evaluating the performance of automated CVE-to-CWE assignment tools presents unique challenges that standard classification metrics fail to adequately address. The complexity arises primarily from two intrinsic characteristics of the CVE-CWE relationship:

* **Multi-Label Nature:** A single CVE does not necessarily map to just one CWE. A specific vulnerability instance (CVE) can arise from the confluence of multiple underlying weaknesses, meaning it may correctly map to a set of CWE identifiers. This multi-label classification scenario invalidates evaluation approaches designed for single-label problems.
* **Hierarchical Structure:** The MITRE CWE list is not a flat collection of independent categories but a structured hierarchy, often represented as a tree or Directed Acyclic Graph (DAG), with defined relationships like parent-child connections. Consequently, prediction errors are not uniform in severity. A predicted CWE that is hierarchically close to a true CWE (e.g., its direct parent or child) represents a more accurate understanding of the weakness than a prediction mapping to a completely unrelated part of the hierarchy. Traditional exact-match evaluation metrics penalize both types of errors equally, failing to capture the nuanced semantic distance between CWEs.

### 1.3. Objective: Designing the Hierarchical CWE Scoring System (HCSS)

The primary objective of this report is to design a Hierarchical CWE Scoring System (HCSS) capable of providing a fair, nuanced, and comprehensive evaluation framework for automated CVE-to-CWE assignment solutions. The HCSS aims to overcome the limitations of traditional metrics by explicitly addressing the multi-label and hierarchical nature of the problem.

The key design requirements for the HCSS are:

* **Multi-Label Assessment:** Accurately evaluate predictions where multiple CWEs can be assigned to a single CVE.
* **Exact Match Reward:** Provide maximum credit for predictions that exactly match the ground-truth CWEs.
* **Hierarchical Partial Credit:** Award partial credit for predicted CWEs that are semantically close to ground-truth CWEs within the MITRE hierarchy.
* **Penalty for Incorrect Predictions:** Appropriately penalize predictions that are unrelated to the true CWEs.
* **Justifiable Closeness Metric:** Incorporate a clearly defined and justifiable metric for quantifying the "closeness" or semantic similarity between CWE nodes.
* **Robust Aggregation:** Provide well-defined methods for aggregating scores, both for individual CVEs and across an entire benchmark dataset.
* **Grounded Methodology:** Base the system on established principles from multi-label classification evaluation, hierarchical classification, and ontology-based semantic similarity measurement.

### 1.4. Report Roadmap

This report systematically addresses the design of the HCSS. Section 2 examines the relationship between CVEs and CWEs, focusing on the mapping process and its multi-label nature. Section 3 delves into the structure and semantics of the MITRE CWE hierarchy. Section 4 discusses the requirements and methods for creating reliable benchmark datasets. Section 5 reviews standard multi-label and specialized hierarchical classification evaluation metrics. Section 6 explores various methods for quantifying semantic distance within the CWE hierarchy. Section 7 analyzes existing practices in evaluating CWE assignment tools and related scoring systems. Section 8 presents the detailed design of the proposed HCSS, integrating findings from previous sections. Finally, Section 9 provides concluding remarks and suggests directions for future work.

## 2. The CVE-CWE Ecosystem: Foundation and Mapping Dynamics

Understanding the interplay between CVEs and CWEs, and the processes governing their linkage, is fundamental to designing an appropriate evaluation system.

### 2.1. Defining CVE and CWE

* **CVE (Common Vulnerabilities and Exposures):** Maintained by the MITRE Corporation, CVE serves as a dictionary, providing unique identifiers (CVE IDs) for specific, publicly disclosed cybersecurity vulnerabilities found in software, hardware, or firmware products. Each CVE entry typically includes a standard identifier, a brief description, and references. CVE IDs are assigned by designated organizations known as CVE Numbering Authorities (CNAs), which include vendors, research organizations, and MITRE itself. CVE focuses on cataloging distinct instances of vulnerabilities as they manifest in real-world systems.
* **CWE (Common Weakness Enumeration):** Also managed by MITRE, CWE is a community-developed, formal list or dictionary that categorizes types of weaknesses in software and hardware architecture, design, code, or implementation. These weaknesses represent the underlying flaws or errors that can lead to exploitable vulnerabilities (CVEs). CWE aims to provide a common language for discussing, identifying, and mitigating these root causes, thereby supporting more secure development practices.

**Relationship:** The relationship between CWE and CVE is often analogized to that between a disease type and a specific patient case. A CWE describes a general category of weakness (e.g., CWE-89: SQL Injection), while a CVE describes a specific instance of that weakness found in a particular product or version (e.g., CVE-2023-12345 describing an SQL injection vulnerability in ExampleApp v1.2). Consequently, CVE records are frequently linked back to one or more CWE entries that represent the fundamental flaws exploited by the vulnerability.

### 2.2. The CVE-to-CWE Mapping Process

The primary public source for CVE-to-CWE mappings is the U.S. National Vulnerability Database (NVD), operated by the National Institute of Standards and Technology (NIST). NVD receives basic CVE information from MITRE and the CNAs and enriches it with additional analysis, including Common Vulnerability Scoring System (CVSS) severity scores, Common Platform Enumeration (CPE) applicability statements identifying affected products, and mappings to relevant CWE identifiers.

However, the mapping process employed by NVD and the broader ecosystem presents several challenges relevant to benchmarking:

* **Normalization and Specificity Loss:** For certain analyses, such as generating the annual CWE Top 25 Most Dangerous Software Weaknesses list, NVD employs a process of "normalization". This involves mapping CVEs primarily to View-1003: Weaknesses for Simplified Mapping of Published Vulnerabilities, a curated subset containing approximately 130 weakness types. If a CVE's initial mapping points to a CWE outside this view, the mapping is often adjusted upwards in the hierarchy to the nearest ancestor within View-1003. While this simplifies analysis for the Top 25 list, it can result in a loss of specificity. A vulnerability accurately identified at a detailed Base or Variant level might be recorded in NVD only at a more general Class or Pillar level. Furthermore, mappings that lack an ancestor within View-1003 may be categorized broadly as "NVD-CWE-Other" or "NVD-CWE-noinfo," or even removed from consideration for certain analyses.
* **Mapping Quality and Consistency:** The accuracy and consistency of mappings can vary. MITRE itself acknowledges that choosing overly high-level CWEs is a common mapping mistake and undertakes efforts to refine mappings, sometimes by consulting the relevant CNAs. CNAs, being closest to the affected products, are often best positioned to provide accurate mappings. However, obtaining feedback from CNAs can be challenging; in one reported re-mapping effort, feedback was received for only 27% of the CVEs submitted for review, leaving the majority with their existing, potentially unverified, mappings.
* **Ecosystem Stability and Timeliness:** The CVE/NVD ecosystem relies on complex workflows and funding structures. Reports have surfaced regarding potential funding expirations for MITRE's operation of the CVE program and backlogs or delays in NVD's enrichment process. Such issues can impact the timeliness, completeness, and potentially the quality of publicly available CVE data, including their associated CWE mappings.

These mapping dynamics carry significant implications for benchmarking. Relying solely on readily available NVD data as ground truth may introduce inaccuracies if the goal is to evaluate a tool's ability to identify the most specific correct CWE. The normalization process means that a tool correctly identifying a Base-level weakness (e.g., CWE-79: Cross-site Scripting) might be unfairly penalized if the benchmark only contains the normalized, higher-level Class mapping (e.g., CWE-74: Improper Neutralization of Special Elements used in an OS Command) derived from View-1003. This potential discrepancy necessitates either careful curation of benchmark datasets to preserve original specificity or designing the scoring system to account for such normalization effects.

Furthermore, the potential for delays or instability in the NVD pipeline highlights the value of static, curated benchmark datasets for reliable and reproducible evaluations. Dependence on live data feeds could introduce variability unrelated to the performance of the tool being assessed. Finally, the variable quality and confidence levels of mappings, stemming from factors like limited CNA review, suggest that benchmark datasets could benefit from documenting the provenance or confidence associated with each ground-truth mapping, potentially allowing for more nuanced evaluation strategies.

### 2.3. Multi-Label Nature of CVE-CWE Mappings

It is explicitly recognized that a single CVE can be associated with multiple CWEs. This reflects the reality that a vulnerability may exploit several distinct types of underlying weaknesses simultaneously. For instance, a vulnerability might involve improper input validation (e.g., CWE-20) leading to the exposure of sensitive information (e.g., CWE-200). The data structures used by NVD, specifically the JSON feed format (version 1.1), accommodate this by allowing an array of weakness descriptions (`problemtype_data`) for each CVE, where each entry can correspond to a different CWE ID. Descriptions of datasets used for CVE-to-CWE classification research also explicitly frame it as a multi-label classification problem. Therefore, any robust evaluation system must be designed to handle predictions and ground truth represented as sets of CWEs per CVE.

## 3. Navigating the MITRE CWE Hierarchy: Structure and Semantics

The hierarchical organization of the CWE list is central to developing a scoring system that awards partial credit based on semantic closeness. Understanding this structure is therefore essential.

### 3.1. Hierarchical Structure and Abstraction Levels

The CWE hierarchy organizes weakness types into a structure that can be conceptualized as a tree or, more accurately, a Directed Acyclic Graph (DAG), as some weaknesses may have multiple parents. This structure is defined by relationships between CWE nodes and is characterized by several distinct levels of abstraction, providing varying degrees of generality or specificity:

* **Pillar:** The highest level of abstraction, representing very broad categories or themes related to weaknesses (e.g., CWE-699: Software Development). Pillars serve as organizational anchors in views like the Research Concepts View (CWE-1000).
* **Class:** A more specific level than Pillars, describing weaknesses in an abstract manner, often independent of specific programming languages or technologies (e.g., CWE-310: Cryptographic Issues). Class-level weaknesses typically focus on one or two dimensions like behavior, property, or resource.
* **Base:** Represents a more concrete type of weakness, generally still independent of specific resources or technologies but providing sufficient detail to suggest specific detection and prevention methods (e.g., CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')). Base-level weaknesses often involve two or three dimensions. This level, along with Variant, is often considered the preferred level of abstraction for mapping the root causes of specific vulnerabilities (CVEs).
* **Variant:** The most specific level, describing a weakness linked to a particular type of product, environment, language, or technology (e.g., CWE-80: Basic XSS). Variant-level weaknesses typically incorporate three to five descriptive dimensions. It is also a preferred level for mapping specific vulnerability instances.
* **Compound:** Represents aggregations of multiple weaknesses, such as Chains (sequences of weaknesses) or Composites (combinations where multiple weaknesses must be present).

In addition to these abstraction levels, the CWE content is organized into various Views. These are specific perspectives or slices through the hierarchy designed for particular purposes, such as View-1003 (Weaknesses for Simplified Mapping), CWE-1000 (Research Concepts View), or views focused on specific programming languages (e.g., Software Written in Java) or development phases (e.g., Introduced During Design).

### 3.2. Relationships Between CWE Nodes

The structure of the hierarchy is explicitly defined by several types of relationships connecting CWE nodes (which can be Weaknesses, Categories, or Views):

* **ChildOf:** This is the primary hierarchical relationship, linking a CWE node to a more general parent node at a higher level of abstraction. A node can have multiple parents in a DAG structure.
* **ParentOf:** The inverse of ChildOf, linking a node to a more specific child node at a lower level of abstraction.
* **PeerOf:** Connects a node to another node at a similar level of abstraction that is related in nature but not in a direct parent-child lineage.
* **CanAlsoBe:** Suggests that a particular weakness might, in some contexts, be reasonably classified under a different CWE ID, indicating potential overlap or alternative perspectives.
* **MemberOf:** Links a specific weakness (Base, Variant, etc.) to the Categories or Views it belongs to, providing context within different organizational structures (e.g., CWE-79 is a MemberOf CWE-710: OWASP Top Ten 2017 Category A7 - Cross-Site Scripting (XSS)).

These relationships allow for systematic navigation of the hierarchy, enabling the identification of ancestors, descendants, parents, children, and peers for any given CWE node. This navigational capability is crucial for calculating hierarchical distances and implementing set-augmentation methods for evaluation.

### 3.3. Accessing and Interpreting Hierarchy Data

The official source for CWE definitions, relationships, and hierarchy structure is the MITRE CWE website. MITRE provides downloadable versions of the CWE list, primarily in XML format. Comprehensive XML downloads encapsulate the entire hierarchy, including weaknesses, categories, views, and their relationships. While CSV downloads are available for specific views, the XML format provides the most complete structural information.

The structure of the XML data is defined by the official CWE XML Schema (XSD), also available from MITRE. This schema details the elements (e.g., `<Weakness>`, `<Category>`, `<View>`), attributes (e.g., ID, Name, Abstraction), and crucially, the elements defining relationships. Hierarchical links like ParentOf and ChildOf are typically represented within a `<Related_Weakness>` element, which includes attributes specifying the `Nature` of the relationship (e.g., 'ChildOf', 'ParentOf', 'PeerOf') and the `CWE_ID` of the related node. Documentation explaining the schema elements is also provided. Various programming libraries and tools can be used to parse and query this XML data to extract the hierarchy structure needed for evaluation.

The explicit definition of abstraction levels offers a direct way to conceptualize hierarchical distance. Moving between adjacent levels (e.g., Variant to Base, Base to Class) represents a discrete step in generality. This structure can be directly leveraged by a closeness metric, perhaps assigning different weights or penalties based on the number of abstraction levels separating two CWEs.

Furthermore, the existence of multiple relationship types (ChildOf, ParentOf, PeerOf, CanAlsoBe) suggests that a sophisticated closeness metric could differentiate based on the type of link, not just its existence or the path length. For instance, ParentOf/ChildOf relationships might imply stronger semantic similarity than PeerOf relationships, reflecting the primary vertical structure of the hierarchy. This nuanced understanding could lead to more accurate partial credit assignments.

Crucially, the availability of the complete hierarchy, including abstraction levels and relationship types, in a machine-readable XML format governed by a defined schema confirms the technical feasibility of implementing an automated scoring system like HCSS. The necessary structural information can be programmatically extracted and utilized for calculating distances and relationships.

## 4. Benchmark Data for Evaluation: Establishing Ground Truth

The reliability and validity of any benchmark depend critically on the quality of the ground truth data used for comparison. For evaluating CVE-to-CWE mapping tools, this requires a "known-good" dataset of CVEs accurately mapped to their corresponding CWE sets.

### 4.1. Need for a "Known-Good" Benchmark

A benchmark dataset serves as the gold standard against which the predictions of automated tools are measured. Its purpose is to provide a consistent, reliable, and representative set of CVE-CWE mappings that reflect the true underlying weaknesses associated with the vulnerabilities. Without such a benchmark, evaluating and comparing the performance of different automated mapping solutions becomes subjective and unreliable.

### 4.2. Sources and Creation Methods

Several approaches can be employed to construct a benchmark dataset:

* **Leveraging Existing Databases:** The NVD is the most comprehensive public repository linking CVEs to CWEs. Datasets can be created by parsing the NVD data feeds, which are available in JSON format and contain CVE details along with associated CWE IDs within the `problemtype` structure. Publicly available datasets, such as one found on Kaggle, have been derived using this method. Open-source projects like OpenCVE also utilize these feeds to populate local databases.
* **Expert Curation and Verification:** As discussed in Section 2.2, raw NVD data may contain mappings that are normalized (losing specificity) or potentially inconsistent. Therefore, creating a high-fidelity benchmark often requires expert intervention. This can involve:
    * Manual review and verification of NVD mappings by cybersecurity experts.
    * Annotation of CVEs with the most specific applicable CWEs (aiming for Base or Variant level where possible).
    * Resolving ambiguities or disagreements in mappings.
    * Employing semi-supervised methods where initial mappings (perhaps generated by large language models) are rigorously reviewed and corrected by human experts.
* **Specialized Datasets:** Research efforts often generate datasets tailored to specific evaluation goals. Examples include:
    * **CASTLE:** A dataset of 250 small, manually crafted C programs, each designed to contain zero or one specific CWE, intended for evaluating static analysis tools, formal verification methods, and LLMs on code-level detection.
      * https://github.com/CASTLE-Benchmark
    * **CVE-Bench:** Focuses on evaluating the ability of LLM agents to repair CVEs, providing repository context and simulating real-world repair processes.
    * **Malvuln Dataset:** A specialized dataset focusing on vulnerabilities within malware samples.

While valuable, these specialized datasets may not be directly suitable for the general task of benchmarking tools that map CVE descriptions to CWE identifiers across a broad range of vulnerabilities.

The process of creating a truly "known-good" benchmark is inherently challenging. The limitations of authoritative sources like NVD (normalization, potential delays, reliance on sometimes unresponsive CNAs) mean that simply downloading and using NVD data might not yield a sufficiently accurate or specific ground truth for rigorous benchmarking. A robust methodology combining data extraction with expert validation, potentially involving multiple annotators and clear guidelines, appears necessary to ensure the benchmark's reliability and fitness for purpose.

Furthermore, the design of the benchmark influences its applicability. Datasets focusing on code snippets (like CASTLE) or repository context (like CVE-Bench) are suited for evaluating specific types of tools (e.g., SAST, code-aware LLMs). However, for the common task of mapping a CVE identifier and its textual description to one or more CWEs – a task often performed by analyzing the description text – the benchmark should primarily consist of tuples in the format: (CVE ID, CVE Description, {Set of Ground Truth CWE IDs}).

### 4.3. Benchmark Dataset Characteristics

An ideal benchmark dataset for HCSS should possess the following characteristics:

* **Scope and Representativeness:** It should include a large and diverse sample of CVEs, covering various types of weaknesses, different software and hardware domains, and spanning a significant time period to ensure the evaluation is not biased towards specific vulnerability trends. The size should be adequate for drawing statistically meaningful conclusions.
* **Mapping Quality and Specificity:** The ground-truth CVE-to-CWE mappings must be accurate and, critically, should aim for the appropriate level of specificity – ideally the Base or Variant level, as these are often the most actionable and represent the true root cause. Overly general mappings (e.g., at the Class or Pillar level) should be avoided unless they are genuinely the most appropriate fit. Documenting the provenance or confidence level of each mapping (e.g., NVD-assigned, CNA-verified, expert-annotated) would add significant value.
* **Handling Multi-Label Cases:** The dataset structure must explicitly support and represent the one-to-many relationship between CVEs and CWEs, storing the ground truth as a set of CWE IDs for each CVE.
* **Maintenance:** Given the dynamic nature of vulnerabilities and potential updates to CWE itself, the benchmark dataset may require periodic review, updates, or versioning to maintain its relevance and accuracy over time.

## 5. Evaluation Metrics for Complex Classification: Beyond Simple Accuracy

Evaluating multi-label, hierarchical classification tasks requires moving beyond traditional single-label accuracy metrics to capture the nuances of partially correct predictions and the significance of hierarchical relationships.

### 5.1. Standard Multi-Label Classification Metrics

Several metrics have been developed for standard multi-label classification problems, where the hierarchy is ignored, but the presence of multiple labels per instance is considered:

* **Subset Accuracy (Exact Match Ratio):** This is the most stringent metric, measuring the proportion of instances where the predicted set of labels is identical to the true set of labels. A single incorrect or missing label results in a score of zero for that instance. While intuitive, it often yields very low scores and fails to credit partially correct predictions.
* **Hamming Loss:** This metric calculates the average fraction of labels that are incorrectly predicted per instance. It considers both false positives (predicting a label that isn't true) and false negatives (failing to predict a label that is true), summed across all labels and averaged over all instances. Lower values indicate better performance. It is less strict than subset accuracy as it accounts for individual label errors.
* **Example-Based Metrics (Precision, Recall, F1-Score):** These metrics are calculated for each instance individually and then averaged across the dataset. For a single instance, precision is the fraction of predicted labels that are correct, recall is the fraction of true labels that were predicted, and F1 is their harmonic mean. Averaging these per-instance scores (sometimes referred to as 'samples' averaging) gives an overall measure of performance.
* **Label-Based Metrics (Micro/Macro/Weighted Averaging):** These metrics assess performance from the perspective of individual labels across the entire dataset.
    * **Micro-Averaging:** Aggregates the counts of true positives (TP), false positives (FP), and false negatives (FN) for all labels across all instances first. Then, calculates overall precision (`Total TP / (Total TP + Total FP)`), recall (`Total TP / (Total TP + Total FN)`), and F1 based on these aggregated counts. This approach gives equal weight to every individual label prediction, effectively weighting instances with more labels more heavily.
    * **Macro-Averaging:** Calculates precision, recall, and F1 for each label independently (treating it as a binary classification problem against all other labels). Then, computes the unweighted average of these per-label metrics. This gives equal weight to each label class, regardless of how frequently it appears in the dataset. It can be sensitive to performance on rare labels.
    * **Weighted-Averaging:** Similar to macro-averaging, but the average of per-label metrics is weighted by the number of true instances (support) for each label. This accounts for label imbalance, giving more importance to the performance on more frequent labels.

While these metrics effectively handle the multi-label aspect, they fundamentally treat the label space (CWEs) as flat and independent. They do not incorporate the hierarchical relationships between CWEs, meaning a prediction of a parent CWE is treated the same as a prediction of a completely unrelated CWE when evaluating against a true child CWE. This limitation necessitates the use of hierarchy-aware metrics.

### 5.2. Hierarchical Classification Metrics

Hierarchical classification metrics are specifically designed to address the limitation of flat metrics by incorporating the structure of the class hierarchy into the evaluation. The core idea is that classification errors are not equally severe; misclassifying an instance into a sibling or parent class is generally considered less erroneous than misclassifying it into a distant class in a different subtree.

A prominent family of hierarchical metrics suitable for multi-label problems involves augmenting the sets of predicted and true labels with their ancestors in the hierarchy. This approach naturally incorporates partial credit based on shared ancestry. The key metrics are Hierarchical Precision (hP), Hierarchical Recall (hR), and Hierarchical F-measure (hF):

**Concept:** For a given instance (CVE), let $Y$ be the set of true CWE labels and $\hat{Y}$ be the set of predicted CWE labels. Augment these sets by including all ancestors of each label within the hierarchy $H$.
$$ Y_{aug} = Y \cup \bigcup_{y \in Y} Ancestors(y, H) $$
$$ \hat{Y}_{aug} = \hat{Y} \cup \bigcup_{\hat{y} \in \hat{Y}} Ancestors(\hat{y}, H) $$
where $Ancestors(c, H)$ denotes the set of all ancestor nodes of class $c$ in hierarchy $H$.

**Calculation:**
$$ \text{Hierarchical Precision (hP)} = \frac{|\hat{Y}_{aug} \cap Y_{aug}|}{|\hat{Y}_{aug}|} $$
$$ \text{Hierarchical Recall (hR)} = \frac{|\hat{Y}_{aug} \cap Y_{aug}|}{|Y_{aug}|} $$
$$ \text{Hierarchical F-measure (hF)} = \frac{2 \times hP \times hR}{hP + hR} \quad (\text{if } hP+hR>0, \text{ else } 0) $$

**Interpretation:** These metrics measure the overlap between the augmented sets. If a predicted CWE $\hat{y} \in \hat{Y}_i$ is not an exact match for any true CWE $y \in Y_i$, but shares common ancestors, the intersection $|\hat{Y}_{aug,i} \cap Y_{aug,i}|$ will be non-empty due to these shared ancestors. This results in non-zero $hP_i$ and $hR_i$, effectively granting partial credit based on the degree of ancestral overlap. The closer the prediction is in the hierarchy (i.e., the more specific the shared ancestors), the larger the intersection and the higher the scores.

Similar to standard multi-label metrics, these hierarchical measures (hP, hR, hF) can be aggregated across a dataset using micro- or macro-averaging approaches.

* **Micro-averaging (hierarchical):** Calculate the augmented sets ($Y_{aug,i}, \hat{Y}_{aug,i}$) for each instance $i$. Sum the sizes of the intersections across all instances ($\sum_i |\hat{Y}_{aug,i} \cap Y_{aug,i}|$) and divide by the sum of the sizes of the augmented predicted sets ($\sum_i |\hat{Y}_{aug,i}|$) for micro-hP, and by the sum of the sizes of the augmented true sets ($\sum_i |Y_{aug,i}|$) for micro-hR. Calculate micro-hF from micro-hP and micro-hR. This gives equal weight to each node in the augmented sets across the dataset.
* **Macro-averaging (hierarchical):** Calculate $hP_i, hR_i, hF_i$ for each instance $i$. Compute the final Macro-hP, Macro-hR, Macro-hF by taking the unweighted average of these per-instance scores across all instances in the dataset. This gives equal weight to each instance (CVE).

The choice between micro and macro averaging depends on the evaluation focus. Micro averaging reflects overall performance on all hierarchical assignments, potentially favoring performance on CVEs with larger augmented label sets. Macro averaging reflects the average performance per CVE, treating each vulnerability instance equally regardless of the complexity of its CWE mapping. Given these different perspectives, reporting both provides a more comprehensive assessment.

The set-augmentation approach for hP/hR/hF stands out as particularly suitable for HCSS. It directly addresses the core requirements by handling multi-label sets and inherently incorporating partial credit through the mechanism of ancestor overlap, grounding the evaluation firmly in the structure of the CWE hierarchy itself. This makes it a strong candidate for the primary evaluation metric.

## 6. Measuring Closeness within the CWE Hierarchy: Quantifying Semantic Distance

To implement a partial credit mechanism, especially for alternative scoring approaches or deeper diagnostics beyond the set-based hP/hR/hF, a method for quantifying the semantic similarity or distance between any two CWE nodes in the hierarchy is required.

### 6.1. The Concept of Semantic Similarity/Distance in Ontologies

Hierarchies and ontologies, like CWE, implicitly encode semantic relationships between their constituent concepts (nodes). Nodes that are closer together in the structure, or share more specific common ancestors, are generally considered more semantically similar. Semantic similarity measures aim to quantify this relatedness, typically producing a score where higher values indicate greater similarity. These are often inversely related to distance metrics, where lower values indicate greater similarity (shorter distance). Since CWE provides a structured vocabulary and taxonomy for weaknesses, methods developed for measuring semantic similarity in ontologies are applicable.

### 6.2. Potential Closeness/Distance Metrics for the CWE Hierarchy

Several families of metrics exist for quantifying similarity or distance in hierarchical structures. Key candidates applicable to the CWE DAG include:

* **Path-Based Measures:** These rely on the paths connecting nodes in the hierarchy graph.
    * **Shortest Path Length:** The simplest measure counts the minimum number of edges connecting two nodes. A shorter path implies greater similarity. For CWE, this requires navigating the DAG considering ParentOf/ChildOf links. A potential refinement could involve penalizing changes in direction (moving up then down). A significant limitation is that it treats all hierarchical levels and edges as equivalent.
    * **Leacock & Chodorow Similarity:** Normalizes the shortest path length by the maximum depth of the hierarchy: $sim_{LC}(c_1, c_2) = -\log\left(\frac{shortest\_path(c_1, c_2)}{2 \times D}\right)$, where $D$ is the maximum depth. Requires knowing the overall hierarchy depth.
* **Node-Based (Information Content - IC) Measures:** These measures leverage the specificity or information content of nodes, often estimated from their frequency in a corpus.
    * **Resnik Similarity:** Defines similarity based solely on the Information Content (IC) of the Lowest Common Subsumer (LCS) or Most Specific Common Ancestor of the two nodes: $sim_{Res}(c_1, c_2) = IC(LCS(c_1, c_2))$. $IC(c)$ is typically calculated as $-\log P(c)$, where $P(c)$ is the probability of encountering class $c$ or any of its descendants. Requires a corpus annotated with CWEs to estimate probabilities.
    * **Lin Similarity:** Normalizes Resnik similarity using the IC of the individual nodes: $sim_{Lin}(c_1, c_2) = \frac{2 \times IC(LCS(c_1, c_2))}{IC(c_1) + IC(c_2)}$. Ranges between 0 and 1.
    * **Jiang & Conrath Distance:** A distance measure combining IC: $dist_{JC}(c_1, c_2) = IC(c_1) + IC(c_2) - 2 \times IC(LCS(c_1, c_2))$. Lower distance means higher similarity.
* **Hierarchy Structure Measures (Edge/Node Combination):** These measures directly use structural properties like node depth and LCS position without requiring external corpus statistics.
    * **Wu & Palmer Similarity:** Considers the depth of the two nodes ($c_1, c_2$) and the depth of their LCS in the hierarchy: $sim_{WP}(c_1, c_2) = \frac{2 \times depth(LCS(c_1, c_2))}{depth(c_1) + depth(c_2)}$. This measure captures similarity based on the relative position within the hierarchy, where depth reflects specificity. Ranges between 0 and 1 (for single inheritance trees, slightly different for DAGs).
    * **Lowest Common Subsumer (LCS) Depth:** Simply using the depth of the LCS as a measure of similarity. A deeper LCS indicates that the nodes share a more specific common ancestor, implying higher similarity.
* **Feature-Based Measures:** Similarity can also be assessed by comparing the features or descriptions of the CWE nodes themselves. For instance, applying Natural Language Processing techniques like Term Frequency-Inverse Document Frequency (TF-IDF) followed by Cosine Similarity to the textual descriptions of CWEs has been proposed for mapping CVEs. While useful for mapping, using it as the hierarchical closeness metric might not directly reflect the curated structure.

### 6.3. Evaluating and Selecting a Closeness Metric for HCSS

The selection of a closeness metric for HCSS should be guided by several criteria:

* **Computational Feasibility:** The metric must be efficiently computable using the available CWE hierarchy data (XML).
* **Interpretability:** The resulting similarity score should be relatively easy to understand and explain.
* **Semantic Alignment:** The metric should align with the intended semantics of the CWE hierarchy, particularly the concept of abstraction levels and parent-child relationships.
* **Sensitivity:** It should be sensitive to meaningful differences in hierarchical position (e.g., parent vs. distant cousin).

Comparing the candidates:

* **Simple Path Length:** Easy to compute but semantically weak, as it ignores node specificity (depth) and treats all links equally. A one-step path from Variant to Base might not represent the same semantic distance as a one-step path from Class to Pillar.
* **IC-Based Measures (Resnik, Lin, JC):** Theoretically appealing as they capture node specificity based on information content. However, they depend heavily on the availability and quality of a large, representative corpus annotated with CWEs for estimating probabilities ($P(c)$). Such a standard corpus may not exist for CWE, making these metrics less practical and potentially unstable compared to purely structural measures.
* **Structural Measures (Wu & Palmer, LCS Depth):** These metrics rely only on the inherent structure of the CWE hierarchy (node depths, parent-child links), which is readily available from the MITRE XML data. They capture aspects of specificity (depth) and shared ancestry (LCS) directly. Wu & Palmer provides a normalized score (0-1), while LCS Depth provides an absolute depth value.

Given the practicality constraints and the desire to leverage the explicit hierarchy structure, structural metrics like Wu & Palmer similarity or LCS Depth appear most suitable for HCSS. They avoid the dependency on external corpora required by IC-based methods and provide a more nuanced measure of closeness than simple path length by incorporating node depth and shared ancestry. The Wu & Palmer metric, providing a normalized score, might be particularly convenient for integration into a partial credit function.

The following table summarizes the comparison:

**Table 1: Comparative Analysis of Potential CWE Closeness Metrics**

| Metric Name             | Definition/Formula Sketch                                                    | Input Required                                    | Pros                                                                 | Cons                                                                                                | Computational Cost | Relevance to CWE Hierarchy                          |
| :---------------------- | :--------------------------------------------------------------------------- | :------------------------------------------------ | :------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------- | :----------------- | :-------------------------------------------------- |
| Shortest Path Length    | Min number of edges between nodes                                            | Hierarchy Structure                               | Simple, easy to compute                              | Ignores node depth/specificity, treats all edges equally                                            | Low                | Basic structural measure                            |
| Leacock & Chodorow      | $-\log\left(\frac{path\_length}{2 \times max\_depth}\right)$                  | Hierarchy Structure, Max Depth                    | Normalizes path length by depth                      | Sensitive to max depth calculation, still primarily path-based                                      | Low                | Incorporates depth                                  |
| Resnik Similarity       | $IC(LCS(c_1, c_2))$                                                          | Hierarchy Structure, IC values                  | Captures specificity of shared ancestor              | Requires corpus for IC calculation, IC definition can vary                                        | Medium (IC calc)   | Models semantic specificity                         |
| Lin Similarity          | $\frac{2 \times IC(LCS)}{IC(c_1) + IC(c_2)}$                                   | Hierarchy Structure, IC values                  | Normalized (0-1), balances LCS and node IC           | Requires corpus for IC calculation                                                                  | Medium (IC calc)   | Models semantic specificity, normalized             |
| Jiang & Conrath Distance| $IC(c_1) + IC(c_2) - 2 \times IC(LCS(c_1, c_2))$                             | Hierarchy Structure, IC values                  | Combines IC of nodes and LCS                         | Requires corpus for IC calculation                                                                  | Medium (IC calc)   | Models semantic distance based on specificity       |
| Wu & Palmer Similarity  | $\frac{2 \times depth(LCS(c_1, c_2))}{depth(c_1) + depth(c_2)}$               | Hierarchy Structure, Node Depths                  | Uses only structure, normalized (0-1 approx.), captures relative depth | May not fully capture nuances of DAGs vs. trees                                                     | Low-Medium         | Good balance of structure, depth, and shared ancestry |
| LCS Depth               | $depth(LCS(c_1, c_2))$                                                       | Hierarchy Structure, Node Depths                  | Uses only structure, directly reflects specificity of common ancestor | Not normalized, absolute depth may be less informative than relative                              | Low-Medium         | Directly measures specificity of shared lineage     |

The choice of closeness metric profoundly influences how partial credit is distributed. A metric like Wu & Palmer or LCS Depth, emphasizing shared ancestry and specificity, aligns conceptually well with the set-augmentation approach of hP/hR/hF, where overlap is determined by shared ancestors. While hP/hR/hF implicitly grants credit based on any shared ancestor, using a metric like Wu & Palmer could allow for a more fine-grained analysis or an alternative scoring mechanism where the amount of partial credit is proportional to the calculated similarity score.

## 7. State-of-the-Art in CWE Assignment Evaluation: Learning from Existing Practices

Examining how CWE assignment and detection tools are currently evaluated in academia and industry provides context for the HCSS design and highlights existing gaps.

### 7.1. Academic Benchmarking and Evaluation Approaches

Recent research efforts evaluating tools related to CWE identification often employ various methodologies and metrics:

* **Code-Level Detection Benchmarks (e.g., CASTLE):** Frameworks like CASTLE focus on evaluating the ability of Static Application Security Testing (SAST) tools, Large Language Models (LLMs), and Formal Verification (FV) techniques to detect specific CWEs within curated, compilable code snippets (primarily in C). Evaluation often involves metrics like detection rates, false positive rates, and sometimes custom scores like the "CASTLE Score" designed for fairer comparison across tool types. These benchmarks are valuable for assessing code analysis capabilities but typically deal with single, predefined CWEs per snippet and may not directly address the multi-label, text-based CVE description-to-CWE mapping task. They highlight common issues like high false positives in SAST and the limitations of FV for certain weakness types (e.g., cryptographic flaws).
* **LLM-Based Vulnerability Analysis:** Studies evaluating LLMs for vulnerability detection or CWE classification often rely on standard machine learning metrics like Accuracy, Precision, Recall, and F1-score, typically measured after fine-tuning the models on specific security datasets. Some approaches might involve generating synthetic data for training or using LLMs within iterative feedback loops. The focus is often on the model's classification performance on datasets that may or may not fully account for the CWE hierarchy or multi-label nature in their evaluation metrics.
* **Similarity-Based Mapping:** Approaches that map CVE descriptions to CWEs using NLP techniques (e.g., TF-IDF and Cosine Similarity) are typically evaluated using standard information retrieval or classification metrics, assessing how well the predicted CWE matches the ground truth based on textual similarity.
* **Use of Hierarchical Metrics:** While the problem of CWE assignment is inherently hierarchical, the explicit use of hierarchical evaluation metrics (like hP/hR/hF) in recent benchmarking papers for CWE detection tools seems less common than the use of standard flat metrics. However, research specifically on hierarchical classification as a field extensively discusses and utilizes such metrics, and libraries like HiClass provide implementations.

A noticeable gap appears in publicly documented, standardized evaluation frameworks specifically designed for the multi-label, hierarchical task of mapping textual CVE descriptions to sets of CWE identifiers, incorporating partial credit based on the hierarchy. Benchmarks like CASTLE serve a different, though related, purpose (code analysis evaluation). The HCSS proposed here aims to address this specific evaluation need.

### 7.2. Industry Scoring Systems (CVSS, CWSS, Vendor Scores)

Industry standards and vendor tools employ scoring systems related to vulnerabilities and weaknesses, but their purpose differs from evaluating classification accuracy:

* **CVSS (Common Vulnerability Scoring System):** This is the industry standard for rating the severity of specific vulnerabilities (CVEs) on a scale of 0-10. It considers factors like attack vector, complexity, privileges required, user interaction, and impact on confidentiality, integrity, and availability. CVSS helps organizations prioritize remediation based on potential impact. While NVD uses CWE information as part of its analysis process which feeds into CVSS scoring, CVSS itself does not measure the correctness of a CVE-to-CWE mapping.
* **CWSS (Common Weakness Scoring System):** Developed by MITRE, CWSS aims to score the relative severity or risk associated with different CWEs (weakness types). It uses metric groups like Base Finding (inherent risk, control effectiveness), Attack Surface (barriers to exploitation), and Environmental (contextual factors) to prioritize weaknesses. Like CVSS, CWSS focuses on risk assessment, not on evaluating the accuracy of classifying a CVE under a specific CWE.
* **Vendor Scores (e.g., Veracode):** Commercial security tool vendors often use proprietary scoring systems. For example, Veracode's Security Quality Score (0-100) aggregates the severity of detected flaws (based on CWE and CVSS concepts) within an application to provide an overall security posture rating. Such scores measure the security state of an application based on the tool's findings, rather than benchmarking the accuracy of the tool's internal CWE classification engine against an external ground truth.

These industry scoring systems (CVSS, CWSS) are crucial for risk management and prioritization but are fundamentally different from the goal of HCSS. They assess the severity or risk of the vulnerability/weakness, whereas HCSS assesses the accuracy of the classification linking a vulnerability (CVE) to a weakness type (CWE). Confusing these two distinct evaluation purposes is important. While HCSS focuses on classification correctness, the concepts from CVSS/CWSS regarding severity and impact could potentially inform future extensions, such as weighting classification errors differently based on the severity of the involved CWEs.

The relative scarcity of widely adopted hierarchical evaluation measures in practical tool benchmarking, despite their theoretical development, further underscores the potential value of a well-defined, standardized system like HCSS for the specific task of evaluating CVE-to-CWE mapping tools.

## 8. Design of the Hierarchical CWE Scoring System (HCSS)

Based on the preceding analysis of the problem domain, the CWE hierarchy, benchmark requirements, and evaluation metrics, this section details the proposed design for the Hierarchical CWE Scoring System (HCSS).

### 8.1. Core Philosophy

The HCSS is designed to provide a robust and fair evaluation of automated CVE-to-CWE mapping tools. Its core philosophy rests on accurately reflecting the multi-label nature of the task while explicitly rewarding predictions based on their proximity within the CWE hierarchy. It aims to move beyond simple exact-match accuracy to a more nuanced assessment that credits partial understanding demonstrated by hierarchically close predictions.

### 8.2. System Components

* **Input:** For each CVE $i$ in the benchmark dataset:
    * $Y_i$: The ground truth set of CWE identifiers.
    * $\hat{Y}_i$: The set of CWE identifiers predicted by the tool being evaluated.
    * $H$: Access to the MITRE CWE hierarchy structure (e.g., parsed from the official XML data), enabling navigation and identification of ancestor relationships.
* **Output:**
    * Per-CVE scores: $hP_i, hR_i, hF_i$.
    * Dataset-aggregated scores: Macro-averaged hP, hR, hF and Micro-averaged hP, hR, hF.
* **Key Functions (Conceptual):**
    * `get_ancestors(cwe_id, H)`: Returns the set of all ancestor CWE IDs for a given `cwe_id` based on the hierarchy $H$.
    * `augment_set(cwe_set, H)`: Applies `get_ancestors` to each element in `cwe_set` and returns the union of the original set and all ancestor sets.
    * `calculate_hP_hR(Y_i, \hat{Y}_i, H)`: Calculates the hierarchical precision and recall for a single CVE $i$ using the set-augmentation method.

### 8.3. Partial Credit Mechanism

The primary mechanism for incorporating partial credit in HCSS is the inherent property of the chosen core metrics: Hierarchical Precision (hP) and Hierarchical Recall (hR) based on set augmentation.

**Method:** As defined in Section 5.2, these metrics compare the augmented sets $Y_{aug}$ and $\hat{Y}_{aug}$, which include the original labels plus all their ancestors.

**Partial Credit:** If a predicted CWE $\hat{y} \in \hat{Y}_i$ is not an exact match for any $y \in Y_i$, but shares one or more ancestors with some $y \in Y_i$, these shared ancestors will contribute to the intersection $|\hat{Y}_{aug,i} \cap Y_{aug,i}|$. This directly results in non-zero $hP_i$ and $hR_i$, reflecting partial correctness. The more specific the shared ancestors (i.e., the closer $\hat{y}$ is to $y$ in the hierarchy), the larger the contribution to the intersection relative to the sizes of the augmented sets.

While this set-based approach is the core of HCSS, for diagnostic purposes or alternative scoring explorations, an explicit Closeness Metric can be defined based on the analysis in Section 6.

**Chosen Closeness Metric (for diagnostics/alternative scoring):** Based on the analysis favouring structural metrics, the Wu & Palmer Similarity ($sim_{WP}$) is a suitable candidate. It is calculated as $sim_{WP}(c_1, c_2) = \frac{2 \times depth(LCS(c_1, c_2))}{depth(c_1) + depth(c_2)}$, providing a normalized score (approx. 0-1) based on node depths and their lowest common subsumer's depth, computable directly from the hierarchy structure.

**Potential Use:** This $sim_{WP}$ score could be used to analyze the distribution of similarities for incorrect predictions, providing insight into how close the errors typically are. It could also form the basis of an alternative example-based scoring function, although the set-based hP/hR/hF is preferred for its elegance and direct integration of hierarchical credit.

### 8.4. Scoring per CVE (Handling Multi-Label)

The HCSS uses the set-based hierarchical metrics, calculated per CVE, as the primary measure of performance.

**Core Metrics per CVE ($i$):**
1.  Compute augmented sets: $Y_{aug,i} = augment\_set(Y_i, H)$ and $\hat{Y}_{aug,i} = augment\_set(\hat{Y}_i, H)$.
2.  Calculate intersection size: $Intersection_i = |\hat{Y}_{aug,i} \cap Y_{aug,i}|$.
3.  Calculate Hierarchical Precision: $hP_i = \frac{Intersection_i}{|\hat{Y}_{aug,i}|}$ (if $|\hat{Y}_{aug,i}|>0$, else 0).
4.  Calculate Hierarchical Recall: $hR_i = \frac{Intersection_i}{|Y_{aug,i}|}$ (if $|Y_{aug,i}|>0$, else 0).
5.  Calculate Hierarchical F-measure: $hF_i = \frac{2 \times hP_i \times hR_i}{hP_i + hR_i}$ (if $hP_i+hR_i>0$, else 0).

This approach directly handles the multi-label nature by operating on sets ($Y_i, \hat{Y}_i$) and incorporates partial credit via the augmentation step, as described previously.

### 8.5. Score Aggregation Across Dataset

To obtain overall performance scores across the entire benchmark dataset (containing $N$ CVEs), HCSS employs both Micro and Macro averaging of the per-CVE hierarchical metrics.

**Macro-Averaged Scores:**
$$ \text{Macro-hP} = \frac{1}{N} \sum_{i=1}^{N} hP_i $$
$$ \text{Macro-hR} = \frac{1}{N} \sum_{i=1}^{N} hR_i $$
$$ \text{Macro-hF} = \frac{2 \times \text{Macro-hP} \times \text{Macro-hR}}{\text{Macro-hP} + \text{Macro-hR}} \quad (\text{or average per-CVE } hF_i) $$
**Interpretation:** Represents the average performance per CVE, giving equal weight to each vulnerability instance.

**Micro-Averaged Scores:**
1.  Calculate total intersection size: $TotalIntersection = \sum_{i=1}^{N} |\hat{Y}_{aug,i} \cap Y_{aug,i}|$
2.  Calculate total augmented predicted size: $TotalPredAugSize = \sum_{i=1}^{N} |\hat{Y}_{aug,i}|$
3.  Calculate total augmented true size: $TotalTrueAugSize = \sum_{i=1}^{N} |Y_{aug,i}|$

$$ \text{Micro-hP} = \frac{TotalIntersection}{TotalPredAugSize} \quad (\text{if } TotalPredAugSize>0, \text{ else } 0) $$
$$ \text{Micro-hR} = \frac{TotalIntersection}{TotalTrueAugSize} \quad (\text{if } TotalTrueAugSize>0, \text{ else } 0) $$
$$ \text{Micro-hF} = \frac{2 \times \text{Micro-hP} \times \text{Micro-hR}}{\text{Micro-hP} + \text{Micro-hR}} \quad (\text{if Micro-hP + Micro-hR > 0, else 0}) $$
**Interpretation:** Represents overall performance aggregated across all individual nodes in the augmented sets, effectively giving more weight to CVEs with more complex hierarchical mappings (larger augmented sets).

**Primary HCSS Result:** The primary results reported should be the Micro-hF and Macro-hF scores, as the F-measure provides a balanced view of precision and recall. Reporting the corresponding precision and recall values (Micro/Macro hP and hR) is also recommended. Optionally, standard multi-label metrics like Hamming Loss or Subset Accuracy can be reported for comparative context, acknowledging their limitations regarding hierarchy.

The necessity of reporting both micro and macro averages stems from the potential variability in the number of true CWEs per CVE and the size of their corresponding augmented sets. A tool might excel at simple CVEs (few CWEs, shallow hierarchy) but struggle with complex ones, or vice versa. Micro and macro averages capture these different performance aspects: micro reflects aggregate performance weighted by label complexity, while macro reflects average performance per CVE instance. Presenting both provides a more complete and robust assessment of a tool's capabilities.

### 8.6. Formal HCSS Definition

The components and calculation flow of HCSS are summarized below:

**Table 2: HCSS Formal Definition**

| Component         | Symbol / Notation      | Definition / Calculation                                                                                                                               | Notes                                                                    |
| :---------------- | :--------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------- |
| **Inputs (per CVE i)** |                        |                                                                                                                                                        |                                                                          |
| True CWE Set      | $Y_i$                  | Set of ground truth CWE IDs for CVE i.                                                                                                                 | e.g., {CWE-79, CWE-89}                                                   |
| Predicted CWE Set | $\hat{Y}_i$            | Set of predicted CWE IDs for CVE i.                                                                                                                    | e.g., {CWE-80, CWE-89}                                                   |
| CWE Hierarchy     | $H$                    | Data structure representing CWE nodes and relationships (ParentOf/ChildOf).                                                                            | Parsed from MITRE XML data.                                              |
| **Core Functions**|                        |                                                                                                                                                        |                                                                          |
| Ancestor Set      | $Ancestors(c, H)$      | Set of all ancestor CWE IDs of node c in hierarchy H.                                                                                                  | Includes transitive ancestors up to the root(s).                         |
| Augmented Set     | $Y_{aug,i}, \hat{Y}_{aug,i}$ | $Y_{aug,i} = Y_i \cup \bigcup_{y \in Y_i} Ancestors(y, H)$ <br> $\hat{Y}_{aug,i} = \hat{Y}_i \cup \bigcup_{\hat{y} \in \hat{Y}_i} Ancestors(\hat{y}, H)$ | Includes original nodes plus all their ancestors.                      |
| **Per-CVE Metrics**|                        | Calculated for each CVE $i=1...N$.                                                                                                                     |                                                                          |
| Intersection Size | $Intersection_i$       | $|\hat{Y}_{aug,i} \cap Y_{aug,i}|$                                                                                                                     | Size of the overlap between augmented sets.                              |
| Hierarchical Precision | $hP_i$               | $\frac{Intersection_i}{\hat{Y}_{aug,i}}$ (handle division by zero)                                                                                 | Proportion of augmented predictions that are in the augmented true set.  |
| Hierarchical Recall | $hR_i$                 | $\frac{Intersection_i}{Y_{aug,i}}$ (handle division by zero)                                                                                        | Proportion of augmented true labels that are in the augmented predicted set.|
| Hierarchical F-measure | $hF_i$               | $\frac{2 \times hP_i \times hR_i}{hP_i + hR_i}$ (handle division by zero)                                                                            | Harmonic mean of $hP_i$ and $hR_i$.                                      |
| **Aggregated Metrics**|                        | Calculated over the entire dataset of N CVEs.                                                                                                        |                                                                          |
| Macro-hP          | $hP_{macro}$           | $\frac{1}{N} \sum_{i=1}^{N}hP_i$                                                                                                                      | Average per-CVE hierarchical precision.                                  |
| Macro-hR          | $hR_{macro}$           | $\frac{1}{N} \sum_{i=1}^{N} hR_i$                                                                                                                      | Average per-CVE hierarchical recall.                                     |
| Macro-hF          | $hF_{macro}$           | $\frac{2 \times hP_{macro} \times hR_{macro}}{hP_{macro} + hR_{macro}}$ (handle division by zero)                                                        | Overall F-measure based on macro-averaged P & R.                       |
| Micro-hP          | $hP_{micro}$           | $\frac{\sum_{i=1}^{N} Intersection_i}{\sum_{i=1}^{N}}$ |${\hat{Y}_{aug,i}}$ (handle division by zero)                                                     | Overall precision aggregated across all augmented labels.                |
| Micro-hR          | $hR_{micro}$           | $\frac{\sum_{i=1}^{N} Intersection_i}{\sum_{i=1}^{N}}$ |${Y_{aug,i}}$ (handle division by zero)                                                          | Overall recall aggregated across all augmented labels.                   |
| Micro-hF          | $hF_{micro}$           | $\frac{2 \times hP_{micro} \times hR_{micro}}{hP_{micro} + hR_{micro}}$ (handle division by zero)                                                        | Overall F-measure based on micro-averaged P & R.                       |
| **Optional Diagnostic**|                        |                                                                                                                                                        |                                                                          |
| Closeness Function| $sim(c_1, c_2)$        | e.g., $sim_{WP}(c_1, c_2) = \frac{2 \times depth(LCS(c_1, c_2))}{depth(c_1) + depth(c_2)}$                                                             | Quantifies similarity (0-1) between two CWE nodes. Used for analysis. |

## 9. Conclusion and Future Directions

### 9.1. Summary of HCSS

This report has detailed the design of the Hierarchical CWE Scoring System (HCSS), a novel framework for evaluating automated tools that map CVEs to CWEs. Addressing the inherent multi-label nature of CVE-CWE assignments and the crucial hierarchical structure of the CWE list, HCSS utilizes set-based hierarchical precision (hP), recall (hR), and F-measure (hF) as its core metrics. By augmenting the predicted and true CWE sets with their ancestors, HCSS naturally incorporates partial credit for predictions that are hierarchically close to the ground truth, rewarding semantic understanding beyond exact matches. The system specifies both Micro and Macro averaging techniques for aggregating scores across a benchmark dataset, providing a comprehensive view of tool performance. HCSS offers a significant improvement over traditional flat classification metrics, which fail to account for the severity of hierarchical errors, and provides a standardized, justifiable methodology tailored to the specific challenges of CVE-to-CWE mapping evaluation.

### 9.2. Recommendations for Implementation and Use

For practical implementation and application of HCSS:

* **Data Sources:** Utilize the official MITRE CWE XML data downloads to construct the hierarchy graph $H$. Standard graph traversal algorithms (e.g., Breadth-First Search or Depth-First Search) can be used to find ancestors.
* **Benchmark Dataset:** Employ a high-quality, curated benchmark dataset as described in Section 4. This dataset should contain representative CVEs with accurate, specific (ideally Base/Variant level) multi-label CWE ground truth mappings. The quality of the benchmark is paramount for meaningful evaluation.
* **Metric Calculation:** Implement the calculation of augmented sets ($Y_{aug}, \hat{Y}_{aug}$) and the subsequent computation of per-CVE $hP_i, hR_i, hF_i$. Aggregate these scores using both Micro and Macro averaging as defined in Section 8.5.
* **Reporting:** Report Micro-hF and Macro-hF as the primary HCSS scores. Include the corresponding Micro/Macro hP and hR values for completeness. Consider reporting standard metrics (e.g., Hamming Loss) alongside HCSS scores for broader context, while clearly stating the limitations of those flat metrics.

### 9.3. Future Work and Potential Extensions

The HCSS framework provides a solid foundation, but several avenues exist for future refinement and extension:

* **Severity Weighting:** Incorporate the severity or potential impact of CWEs into the scoring. Errors involving critical weaknesses (e.g., those highly ranked in the CWE Top 25 or having high CWSS scores) could be penalized more heavily than errors involving less severe weaknesses. This would require integrating CWSS scores or similar risk assessments into the calculation.
* **Refined Closeness Metrics:** Empirically investigate the performance and semantic validity of different structural closeness metrics (e.g., Wu & Palmer vs. LCS Depth vs. path-based variants) within the CWE context to potentially refine diagnostic analyses or alternative scoring functions.
* **Confidence Weighting:** If benchmark datasets include confidence scores for ground-truth mappings (reflecting NVD assignment vs. CNA verification vs. expert annotation), HCSS could potentially weight comparisons based on this confidence.
* **Temporal Analysis:** Extend the framework to evaluate how well tools map CVEs considering the evolution of the CWE hierarchy itself over different versions.
* **Standardization and Tooling:** Develop and release standardized, open-source implementations of the HCSS calculation logic and curated benchmark datasets to promote adoption and ensure consistent evaluation practices across the community.

By providing a methodologically sound approach to evaluating CVE-to-CWE mapping tools, HCSS can contribute to the development of more accurate and reliable automated solutions, ultimately enhancing vulnerability management and software security practices.



## 10 Research

### 10.1 Hierarchical Classifiers
1. [Evaluation Measures for Hierarchical Classification: a unified view and novel approaches](https://arxiv.org/abs/1306.6802v2), June 2013
2. [A Review of Performance Evaluation Measures for Hierarchical Classifiers](https://cdn.aaai.org/Workshops/2007/WS-07-05/WS07-05-001.pdf)
3. [A Survey of Hierarchical Classification Across Different Application Domains](https://www.cs.kent.ac.uk/people/staff/aaf/pub_papers.dir/DMKD-J-2010-Silla.pdf)
4. [Ontology-based semantic similarity measurements: an overview](https://graus.nu/blog/ontology-based-semantic-similarity-measurements-an-overview/)
5. [A modification of Wu and Palmer Semantic Similarity Measure](https://profs.etsmtl.ca/ctadj/papers/2-conf/2016/Djamel-ubicomm_2016_3_10_10130.pdf), 2016
6. [A NEW SIMILARITY MEASURE FOR TAXONOMY BASED ON EDGE COUNTING](https://arxiv.org/pdf/1211.4709)

