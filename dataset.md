# CVE-CWE Root Cause Mapping Dataset Requirements

> [!TIP]
> This document outlines the requirements for creating a comprehensive benchmark dataset for CVE-CWE Root Cause mapping.

## Table of Contents

- [CVE-CWE Root Cause Mapping Dataset Requirements](#cve-cwe-root-cause-mapping-dataset-requirements)
  - [Table of Contents](#table-of-contents)
  - [Coverage Requirements](#coverage-requirements)
  - [Quality Requirements](#quality-requirements)
  - [Data Sources](#data-sources)
    - [Datasets](#datasets)
    - [CVE Info](#cve-info)
  - [JSON Schema](#json-schema)
    - [Example Record](#example-record)
    - [Example Directory Layout](#example-directory-layout)
  - [Validation \& Tests](#validation--tests)
  - [References](#references)
  - [ToDo](#todo)

## Coverage Requirements

<a id="REQ_COVERAGE_PRACTICAL_CWES"></a>**REQ_COVERAGE_PRACTICAL_CWES**: The dataset MUST include all CWEs used in practice i.e. a CWE is used in at least 1 published CVE(s) (approximately 400).

<a id="REQ_COVERAGE_CWE1003_VIEW"></a>**REQ_COVERAGE_CWE1003_VIEW**: The dataset MUST include all CWEs in [CWE-1003 view](https://cwe.mitre.org/data/definitions/1003.html) (approximately 130).

<a id="REQ_COVERAGE_TOP25_CWES"></a>**REQ_COVERAGE_TOP25_CWES**: The dataset MUST include all CWEs in https://cwe.mitre.org/top25/ from [2019](https://cwe.mitre.org/top25/archive/) to [current 2024 CWE Top 25](https://cwe.mitre.org/top25/index.html).

<a id="REQ_COVERAGE_MIN_ENTRIES_PER_CWE"></a>**REQ_COVERAGE_MIN_ENTRIES_PER_CWE**: The dataset MUST include at least 5 entries per CWE (if such CVE examples exist).

<a id="REQ_COVERAGE_BALANCED_SUBSET"></a>**REQ_COVERAGE_BALANCED_SUBSET**: The dataset MUST include a balanced subset that has at most 5 entries per CWE.

<a id="REQ_COVERAGE_DIVERSE_DESCRIPTIONS"></a>**REQ_COVERAGE_DIVERSE_DESCRIPTIONS**: For a given CWE, the dataset MUST include CVE Descriptions that are sufficiently different per CVE Description, measured quantitatively using fuzzy and semantic similarity against a threshold.

<a id="REQ_COVERAGE_MULTIPLE_CWES"></a>**REQ_COVERAGE_MULTIPLE_CWES**: The dataset MUST include CVEs with multiple CWEs assigned, as reflected by the Top25 datasets.

<a id="REQ_COVERAGE_REF_CONTENT_RAW"></a>**REQ_COVERAGE_REF_CONTENT_RAW**: The dataset MUST include CVE Reference Link Content as the raw content in text format.

<a id="REQ_COVERAGE_REF_CONTENT_SUMMARY"></a>**REQ_COVERAGE_REF_CONTENT_SUMMARY**: The dataset MUST include CVE Reference Link Content as a summary (e.g., as in https://github.com/CyberSecAI/cve_info_refs).

<a id="REQ_COVERAGE_LICENSE"></a>**REQ_COVERAGE_LICENSE**: The dataset MUST include a LICENSE as agreed by MITRE CWE.

<a id="REQ_COVERAGE_README"></a>**REQ_COVERAGE_README**: The dataset MUST include a README.

<a id="REQ_COVERAGE_MAINTENANCE_PLAN"></a>**REQ_COVERAGE_MAINTENANCE_PLAN**: The dataset MUST include a documented Maintenance Plan.

<a id="REQ_COVERAGE_CHANGE_TRACKING"></a>**REQ_COVERAGE_CHANGE_TRACKING**: The dataset MUST include change tracking and full history (e.g., via GitHub).

<a id="REQ_COVERAGE_VALIDATION_PROCESS"></a>**REQ_COVERAGE_VALIDATION_PROCESS**: The dataset MUST include a documented Validation process/script.

<a id="REQ_COVERAGE_FEEDBACK_PROCESS"></a>**REQ_COVERAGE_FEEDBACK_PROCESS**: The dataset MUST include a documented feedback process.

<a id="REQ_COVERAGE_SEMVER"></a>**REQ_COVERAGE_SEMVER**: The dataset MUST use [Semantic Versioning](https://semver.org/).

<a id="REQ_COVERAGE_ALL_CWES"></a>**REQ_COVERAGE_ALL_CWES**: The dataset SHOULD include all CWEs (approximately 1000).

<a id="REQ_COVERAGE_RECENT_CVES"></a>**REQ_COVERAGE_RECENT_CVES**: The dataset SHOULD include mostly CVEs from recent CVE publication years to reflect current reality.

<a id="REQ_COVERAGE_KEYPHRASES"></a>**REQ_COVERAGE_KEYPHRASES**: The dataset SHOULD include the CVE Description Vulnerability KeyPhrases (e.g., as in https://github.com/CyberSecAI/cve_info).

<a id="REQ_COVERAGE_NO_REJECTED_CVES"></a>**REQ_COVERAGE_NO_REJECTED_CVES**: The dataset SHOULD NOT include CVEs that are Rejected at the time of creating the dataset. Over time, CVEs in the dataset may become REJECTED, and these should be removed or identified/labeled.

<a id="REQ_COVERAGE_NO_DEPRECATED_CWES"></a>**REQ_COVERAGE_NO_DEPRECATED_CWES**: The dataset SHOULD NOT include CWEs that are Deprecated, Obsolete, or Prohibited at the time of creating the dataset. Over time, CWEs in the dataset may become deprecated or obsolete, and these should be removed or identified/labeled.

<a id="REQ_COVERAGE_TOOL_AGNOSTIC"></a>**REQ_COVERAGE_TOOL_AGNOSTIC**: The dataset MUST be tool/solution agnostic. E.g. it MUST NOT include prompts per https://huggingface.co/datasets/AI4Sec/cti-bench/viewer/cti-rcm?views%5B%5D=cti_rcm&row=1

## Quality Requirements

<a id="REQ_QUALITY_KNOWN_GOOD"></a>**REQ_QUALITY_KNOWN_GOOD**: The dataset MUST be known-good (i.e., the assigned CWE(s) is correct).

<a id="REQ_QUALITY_UTF8"></a>**REQ_QUALITY_UTF8**: The dataset MUST be UTF-8 only.

<a id="REQ_QUALITY_SCHEMA_VALIDATOR"></a>**REQ_QUALITY_SCHEMA_VALIDATOR**: The dataset MUST have a schema and validator script that is part of the dataset.

<a id="REQ_QUALITY_LOW_QUALITY_CVES"></a>**REQ_QUALITY_LOW_QUALITY_CVES**: The dataset MUST include and identify low-quality CVEs. A sufficient number of low-info CVEs are labeled such that the results for those CVEs can be isolated to e.g., test for hallucinations/grounding.

<a id="REQ_QUALITY_DATA_SPLITS"></a>**REQ_QUALITY_DATA_SPLITS**: The dataset MUST have train/validate/test splits.

<a id="REQ_QUALITY_PRESERVE_ISSUES"></a>**REQ_QUALITY_PRESERVE_ISSUES**: The dataset SHOULD NOT fix existing typos or other issues in the CVEs.

## Data Sources

### Datasets

<a id="REQ_DATASRC_TOP25_2023"></a>**REQ_DATASRC_TOP25_2023**: The dataset SHOULD use Top25 2023 (approximately 7K entries) as a source.

<a id="REQ_DATASRC_TOP25_2022"></a>**REQ_DATASRC_TOP25_2022**: The dataset SHOULD use Top25 2022 (approximately 7K entries) as a source.

<a id="REQ_DATASRC_NO_OBSERVED_EXAMPLES"></a>**REQ_DATASRC_NO_OBSERVED_EXAMPLES**: The dataset SHOULD NOT use CWE Observed Examples (approximately 3K) as many examples are pre-2010 and worded differently from modern CVE descriptions.

### CVE Info

<a id="REQ_CVEINFO_JSON_5_0"></a>**REQ_CVEINFO_JSON_5_0**: https://github.com/CVEProject/cvelistV5 is the canonical source for CVE descriptions & references.

## JSON Schema

<a id="REQ_SCHEMA_FORMAT"></a>**REQ_SCHEMA_FORMAT**: The dataset MUST conform to the following JSON schema (excerpt):



```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "CVE_CWE_Record",
  "type": "object",
  "required": ["cve_id", "cwe_ids", "year", "description", "ref_links"],
  "properties": {
    "cve_id":    {"type": "string", "pattern": "^CVE-\\d{4}-\\d{4,}$"},
    "cwe_ids":   {"type": "array", "items": {"type": "string"}, "minItems": 1},
    "description": {"type": "string"},
    "ref_summaries": {"type": "array", "items": {"type": "string"}},
    "keyphrases":    {"type": "array", "items": {"type": "string"}},
    "quality_flags": {"type": "array", "items": {"type": "string"}},
  }
}
```

### Example Record

```json
{
  "cve_id": "CVE-2024-29009",
  "cwe_ids": ["CWE-352"],
  "description": "CWE-352 Cross-Site Request Forgery (CSRF)",
  "ref_summaries": ["Summary of Reference content goes here"],
  "keyphrases": ["keyphrases go here"],
  "quality_flags": [],
}
```

### Example Directory Layout

```bash
top_level
  ├─ full.jsonl            # 1 CVE/line – gold labels
  ├─ schema.json           # JSON Schema (Draft 2020‑12)
  ├─ scripts/
  │    ├─ fetch.py         # pull raw NVD entries
  │    ├─ clean.py         # transform → schema, keyphrases, de‑dup
  │    ├─ benchmark.py     # benchmarks a dataset against the benchmark
  │    └─ utils.py         # shared helpers (similarity, hashing)
  ├─ README.md
  ├─ LICENSE
  ├─ Feedback.md
  └─ MaintenancePlan.md
```

## Validation & Tests

<a id="REQ_VALIDATION_SCHEMA"></a>**REQ_VALIDATION_SCHEMA**: JSON schema validation must pass with schema.json.

<a id="REQ_VALIDATION_REQUIREMENTS"></a>**REQ_VALIDATION_REQUIREMENTS**: The dataset must be validated against the requirements.

<a id="REQ_VALIDATION_STATISTICS"></a>**REQ_VALIDATION_STATISTICS**: Basic statistics (token lengths, keyphrase counts) must be generated.

## References

1. [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) for definitions of "SHOULD", "MUST", etc.

## ToDo
1. Define [Known-Good](#REQ_QUALITY_KNOWN_GOOD). Specify the validation standard and process.
2. Define Quantitative Metrics: Specify thresholds/methods for [sufficiently different](#REQ_COVERAGE_DIVERSE_DESCRIPTIONS)
3. Define Qualitative Labels: Specify criteria for [low-quality](#REQ_QUALITY_LOW_QUALITY_CVES).