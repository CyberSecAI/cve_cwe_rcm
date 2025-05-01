# Guidance on CWE Assignment Tools

This document provides guidance for developers creating tools that assign Common Weakness Enumeration (CWE) identifiers to Common Vulnerabilities and Exposures (CVEs). 

Tools following these guidelines will produce outputs compatible with the benchmark comparison framework.

## Table of Contents

- [Guidance on CWE Assignment Tools](#guidance-on-cwe-assignment-tools)
  - [Table of Contents](#table-of-contents)
  - [Observability and Logging](#observability-and-logging)
  - [Confidence Scoring](#confidence-scoring)
  - [Rationale and Evidence](#rationale-and-evidence)
  - [CWE Selection Principles](#cwe-selection-principles)
  - [Output Format Requirements](#output-format-requirements)
  - [Tool Behavior Recommendations](#tool-behavior-recommendations)
  - [References](#references)

## Observability and Logging

<a id="GUIDE_LOG_DETAILED"></a>**GUIDE_LOG_DETAILED**: Tools SHOULD generate detailed log files that capture the entire CWE assignment process, including initialization, data loading, processing steps, and result generation.

<a id="GUIDE_LOG_LEVELS"></a>**GUIDE_LOG_LEVELS**: Tools SHOULD implement multiple logging levels (e.g., DEBUG, INFO, WARNING, ERROR) to allow users to control the verbosity of logs.

<a id="GUIDE_LOG_FORMAT"></a>**GUIDE_LOG_FORMAT**: Log entries SHOULD include timestamps, component identifiers, and hierarchical context to facilitate tracing through the assignment process.

<a id="GUIDE_LOG_ERRORS"></a>**GUIDE_LOG_ERRORS**: Tools MUST log all errors and exceptions with sufficient context to understand what failed, why it failed, and the impact on results.

<a id="GUIDE_LOG_PERFORMANCE"></a>**GUIDE_LOG_PERFORMANCE**: Tools SHOULD log performance metrics such as processing time per CVE and memory usage to help identify bottlenecks.

<a id="GUIDE_LOG_CONFIG"></a>**GUIDE_LOG_CONFIG**: Tools MUST log all configuration settings and parameters used for the CWE assignment run to ensure reproducibility.

## Confidence Scoring

<a id="GUIDE_CONF_PER_CWE"></a>**GUIDE_CONF_PER_CWE**: Tools MUST provide a confidence score (between 0.0 and 1.0) for each CWE assigned to a CVE, indicating the tool's certainty in that specific assignment.

<a id="GUIDE_CONF_OVERALL"></a>**GUIDE_CONF_OVERALL**: Tools MUST provide an overall confidence score for the complete set of CWE assignments for each CVE, which may reflect factors beyond individual CWE confidence scores.

<a id="GUIDE_CONF_THRESHOLD"></a>**GUIDE_CONF_THRESHOLD**: Tools SHOULD allow configurable confidence thresholds below which CWEs are either flagged as uncertain or excluded from results.

<a id="GUIDE_CONF_FACTORS"></a>**GUIDE_CONF_FACTORS**: Tools SHOULD document the factors considered in confidence calculations, such as information completeness, ambiguity, semantic match strength, or historical pattern data.

<a id="GUIDE_CONF_LOW_INFO"></a>**GUIDE_CONF_LOW_INFO**: Tools MUST assign lower confidence scores when processing CVEs with minimal or vague information to indicate higher uncertainty.


## Rationale and Evidence

<a id="GUIDE_RAT_EXPLANATION"></a>**GUIDE_RAT_EXPLANATION**: Tools MUST provide a textual explanation for each assigned CWE, describing why that weakness type was selected for the CVE.

<a id="GUIDE_RAT_EVIDENCE"></a>**GUIDE_RAT_EVIDENCE**: Tools MUST cite specific evidence from the CVE description or reference materials that supports each CWE assignment.

<a id="GUIDE_RAT_KEYWORDS"></a>**GUIDE_RAT_KEYWORDS**: Tools SHOULD highlight key terms, phrases, or patterns in the CVE content that influenced the CWE selection.

<a id="GUIDE_RAT_ALTERNATIVES"></a>**GUIDE_RAT_ALTERNATIVES**: Tools SHOULD identify alternative CWEs that were considered but rejected, and explain the reasoning for not selecting them.

<a id="GUIDE_RAT_HIERARCHY"></a>**GUIDE_RAT_HIERARCHY**: When assigning a CWE, tools SHOULD explain its position in the CWE hierarchy and why that abstraction level was chosen over parent or child weaknesses.

<a id="GUIDE_RAT_REFERENCES"></a>**GUIDE_RAT_REFERENCES**: Tools SHOULD reference similar historical CVEs with known CWE assignments that informed the current assignment when applicable.

## CWE Selection Principles

<a id="GUIDE_SEL_ROOT_CAUSE"></a>**GUIDE_SEL_ROOT_CAUSE**: Tools MUST focus on identifying the root cause weakness rather than the vulnerability impact, following CWE mapping guidance.

<a id="GUIDE_SEL_ABSTRACTION"></a>**GUIDE_SEL_ABSTRACTION**: Tools SHOULD prioritize Base and Variant level CWEs over Class level when sufficient information is available, in accordance with CWE mapping best practices.

<a id="GUIDE_SEL_MULTIPLE"></a>**GUIDE_SEL_MULTIPLE**: Tools MUST assign multiple CWEs when a CVE exhibits multiple distinct weaknesses, rather than forcing a single classification.

<a id="GUIDE_SEL_SPECIFICITY"></a>**GUIDE_SEL_SPECIFICITY**: Tools SHOULD select the most specific applicable CWE rather than an overly general one, given sufficient evidence.

<a id="GUIDE_SEL_TECHNICAL"></a>**GUIDE_SEL_TECHNICAL**: Tools SHOULD prioritize technical weakness types (e.g., buffer overflow) over general impact descriptions (e.g., allows code execution) when both are apparent.

<a id="GUIDE_SEL_AMBIGUITY"></a>**GUIDE_SEL_AMBIGUITY**: When faced with genuine ambiguity, tools SHOULD document the uncertainty rather than making arbitrary selections. 

## Output Format Requirements

<a id="GUIDE_OUT_SCHEMA"></a>**GUIDE_OUT_SCHEMA**: Tools MUST output CWE assignments in a structured, machine-readable format (e.g., JSON) that conforms to the benchmark comparison schema.

<a id="GUIDE_OUT_META"></a>**GUIDE_OUT_META**: Output files MUST include metadata such as tool version, timestamp, configuration parameters, and overall processing statistics.

<a id="GUIDE_OUT_CVES"></a>**GUIDE_OUT_CVES**: For each CVE, the output MUST include the CVE ID, all assigned CWEs with confidence scores, rationales, and supporting evidence.

<a id="GUIDE_OUT_SAMPLE"></a>**GUIDE_OUT_SAMPLE**: Sample output format:

```json
{
  "meta": {
    "tool_name": "CWEAssignPro",
    "tool_version": "2.1.3",
    "timestamp": "2025-05-01T14:30:22Z",
    "config": {
      "confidence_threshold": 0.6,
      "abstraction_preference": "base"
    }
  },
  "results": [
    {
      "cve_id": "CVE-2024-21945",
      "overall_confidence": 0.92,
      "assignments": [
        {
          "cwe_id": "CWE-79",
          "confidence": 0.95,
          "abstraction_level": "Base",
          "rationale": "Evidence of stored cross-site scripting where user input in the title field is rendered as JavaScript",
          "evidence": [
            {
              "source": "CVE Description",
              "text": "allows attackers to execute arbitrary JavaScript via the title field",
              "relevance": "high"
            }
          ],
          "alternatives_considered": [
            {
              "cwe_id": "CWE-80",
              "reason_rejected": "While related to XSS, CWE-79 more precisely captures the stored nature of the vulnerability"
            }
          ]
        }
      ],
      "processing_info": {
        "data_sources": ["NVD Description", "Vendor Advisory"],
        "processing_time_ms": 235
      }
    }
  ]
}
```

<a id="GUIDE_OUT_HUMAN"></a>**GUIDE_OUT_HUMAN**: In addition to machine-readable output, tools SHOULD provide human-readable summary reports for quick review.

<a id="GUIDE_OUT_VALIDATION"></a>**GUIDE_OUT_VALIDATION**: Tools SHOULD validate their output against the required schema before completion to ensure compatibility with benchmark comparison tools.

## Tool Behavior Recommendations

<a id="GUIDE_BEH_LOW_INFO"></a>**GUIDE_BEH_LOW_INFO**: When processing CVEs with minimal information, tools SHOULD flag the low-information status and adjust confidence scores accordingly rather than making tenuous assignments.

<a id="GUIDE_BEH_CONSISTENCY"></a>**GUIDE_BEH_CONSISTENCY**: Tools SHOULD maintain consistency in CWE assignments across similar vulnerabilities to support pattern recognition and predictability.

<a id="GUIDE_BEH_UPDATES"></a>**GUIDE_BEH_UPDATES**: Tools SHOULD regularly update their CWE knowledge base to incorporate new CWE entries, relationship changes, and improved mapping guidance.

<a id="GUIDE_BEH_LEARNING"></a>**GUIDE_BEH_LEARNING**: If tools use machine learning approaches, they SHOULD document training data sources, feature engineering practices, and evaluation metrics to establish credibility.

<a id="GUIDE_BEH_AUDITABILITY"></a>**GUIDE_BEH_AUDITABILITY**: Tools MUST make all decision factors transparent and auditable, avoiding "black box" assignments without explanation.

<a id="GUIDE_BEH_SPEED"></a>**GUIDE_BEH_SPEED**: Tools SHOULD process CVEs efficiently, with target performance metrics (e.g., CVEs per second) documented to set user expectations.

<a id="GUIDE_BEH_VERSIONING"></a>**GUIDE_BEH_VERSIONING**: Tools MUST follow semantic versioning and clearly document when updates might change assignment behavior.

## References

<a id="REF_CWE_GUIDANCE"></a>**REF_CWE_GUIDANCE**: [CWE - CVE → CWE Mapping "Root Cause Mapping" Guidance](https://cwe.mitre.org/documents/cwe_usage/guidance.html)

<a id="REF_RFC_2119"></a>**REF_RFC_2119**: [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) for definitions of "SHOULD", "MUST", etc.

<a id="REF_JSON_SCHEMA"></a>**REF_JSON_SCHEMA**: [JSON Schema](https://json-schema.org/) for output format validation