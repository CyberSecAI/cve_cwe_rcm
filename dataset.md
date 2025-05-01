# Benchmark Dataset

## Requirements


### Coverage

1. The dataset MUST include 
   1. all CWEs used in practice i.e. a CWE is used in at least 1 published CVE(s) (~~400)
   2. all CWEs in [CWE-1003 view](https://cwe.mitre.org/data/definitions/1003.html) (130)
   3. all CWEs in https://cwe.mitre.org/top25/ from [2019](https://cwe.mitre.org/top25/archive/) to [current 2024 CWE Top 25](https://cwe.mitre.org/top25/index.html) 
   4. at least 5 entries per CWE (if such CVE examples exist)
   5. a Balanced subset that has at most 5 entries per CWE
   6. For a given CWE, CVE Descriptions that are sufficiently different per CVE Description 
      1. measured quantitatively using fuzzy and semantic similarity against a threshold
   7. CVEs with multiple CWEs assigned
      1. as reflected by the Top25 datasets
   8. CVE Reference Link Content 
      1. as the raw content as text
      2. as a summary e.g. https://github.com/CyberSecAI/cve_info_refs
   9. a LICENSE as agreed by MITRE CWE
   10. a README
   11. a documented Maintenance Plan
   12. a documented Validation process / script
   13. a documented feedback process
   14. [Semantic Versioning](https://semver.org/)
2. The dataset SHOULD include 
   1. All CWEs (~~1000)
   2. mostly CVEs from recent CVE publication years
       1.  to reflect current reality
   3.  the CVE Description Vulnerability KeyPhrases
       1.  e.g. https://github.com/CyberSecAI/cve_info
3.  The dataset SHOULD NOT 
    1.  include CVEs that are Rejected
        1. At the time of creating the dataset, CVEs should not be REJECTED
        2. Over time, CVEs in the dataset may become REJECTED, and these should be removed but don't have to be e.g. could be identified / labeled instead
    2.  include CWEs that are Deprecated, Obsolete, Prohibited
        1. At the time of creating the dataset, CWEs should not be deprecated or obsolete
        2. Over time, CWEs in the dataset may become deprecated or obsolete, and these should be removed but don't have to be e.g. could be identified / labeled instead


### Quality

1. The dataset MUST 
   1. be known-good 
      1. i.e. the assigned CWE(s) is correct
   2. be UTF-8 only 
   3. have a schema and validator script that is part of the dataset
   4. include and identify low-quality CVEs 
      1. a sufficient number of low-info CVEs are labelled such that the results for those CVEs can be isolated to e.g. test for Hallucinations / grounding
   5. have train/validate/test splits
2. The dataset SHOULD NOT 
   1. fix existing typos etc... in the CVEs



## Data Sources

### Datasets

2. Top25 2023 (~~7K)
3. Top25 2022 (~~7K)
1. CWE Observed Examples (~~3K) should NOT be used
   1. many examples are pre-2010 and worded differently from modern CVE descriptions


### CVE Info
1. CVE JSON 5.0 is the canonical source for CVE descriptions & references.


## JSON Schema (excerpt)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "CVE_CWE_Record",
  "type": "object",
  "required": ["cve_id", "cwe_ids", "year", "description", "ref_links"],
  "properties": {
    "cve_id":    {"type": "string", "pattern": "^CVE-\\d{4}-\\d{4,}$"},
    "cwe_ids":   {"type": "array", "items": {"type": "string"}, "minItems": 1},
    "year":      {"type": "integer", "minimum": 1999},
    "description": {"type": "string"},
    "ref_links": {"type": "array", "items": {"type": "string", "format": "uri"}},
    "ref_summaries": {"type": "array", "items": {"type": "string"}},
    "keyphrases":    {"type": "array", "items": {"type": "string"}},
    "quality_flags": {"type": "array", "items": {"type": "string"}},
    "provenance":    {"type": "string"}
  }
}
```

### Example record
```json
{
  "cve_id": "CVE-2024-21945",
  "cwe_ids": ["CWE-79"],
  "year": 2024,
  "description": "Stored XSS in AcmeCMS 4.2 allows attackers to execute arbitrary JavaScript via the title field…",
  "ref_links": ["https://nvd.nist.gov/vuln/detail/CVE-2024-21945"],
  "ref_summaries": ["Vendor advisory 2024‑012"],
  "keyphrases": ["stored XSS", "user‑supplied input"],
  "quality_flags": [],
  "provenance": "NVD‑API"
}
```

```bash
example/
  ├─ full.jsonl            # 1 CVE/line – gold labels
  ├─ schema.json           # JSON Schema (Draft 2020‑12)
  ├─ scripts/
  │    ├─ fetch.py         # pull raw NVD entries
  │    ├─ clean.py         # transform → schema, keyphrases, de‑dup
  │    └─ utils.py         # shared helpers (similarity, hashing)
  ├─ README.md
  ├─ LICENSE
  ├─ Feedback.md
  └─ MaintenancePlan.md

````


Validation & Tests
1. jsonschema validation must pass with schema.json.
2. Validates the dataset against the requirements
2. Print basic stats (token lengths, keyphrase counts).
### References
1. https://www.rfc-editor.org/rfc/rfc2119 for definitions of "SHOULD", "MUST",...