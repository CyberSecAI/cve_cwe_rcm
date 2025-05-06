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

## Scoring Approaches

The following Scoring Approaches were considerered
1. Path Based using CWE graph distance based on ChildOf
   1. [Path based](./spl_scoring.md) Proposal
   2. [See Critique]((./spl_scoring_critiique.md))
2. Hierarchical Classification Standards
   1. [A survey of Hierarchical Classification Standards](./hierarchical_scoring_system.md)
   2. [A proposal with Worked Examples](./hcss.md)

**Hierarchical Classification Standards look the most promising.**