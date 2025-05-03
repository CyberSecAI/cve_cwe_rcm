# Benchmark Scoring Algorithm

> [!TIP]
> This document is an overview of different CWE benchmarking scoring approaches.


## Data Points
A CVE can have one or more CWEs i.e. it's a multi-label classification problem.

MITRE CWE list is a rich document containing detailed information on CWEs (~2800 pages)
The RelatedNatureEnumerations form a (view-dependent e.g. CWE-1003) graph of 
- 1309 ChildOf/ParentOf
- 141 CanPrecede/CanFollow
- 13 Requires/RequiredBy

MITRE CWE list contains different CWE Views
Ref "The CWE “View” Methods" https://cwe.mitre.org/documents/cwe_usage/guidance.html


CWE Software Assurance Trends View (View-1400) view contains every CWE. CWEs are organized into 22 high-level categories of interest to large-scale software assurance research to support the elimination of weaknesses using tactics such as secure language development. This view is structured with categories at the top level, with a second level of only weaknesses. 

---

## Scoring System: Overview

The goal is to assign a numeric score that:

- Rewards exact matches highly.
- Rewards partial matches based on hierarchical proximity (parent-child relationships).
- Provides a standardized range (e.g. 0 to 1) for easy comparison.
- should be based on the CWE graph, and moreover reinforce good mapping behavior.

---

## Definitions & Assumptions

- **Ground Truth (Benchmark)**: One or more known-good CWEs per CVE.
- **Prediction**: One or more CWEs assigned by the tested solution.
- **Hierarchy**: Relationships defined by MITRE’s CWE hierarchy (ChildOf, ParentOf, and sibling CWEs).
- **Proximity**: Measured as shortest hierarchical path (edge distance) between two CWEs.

---