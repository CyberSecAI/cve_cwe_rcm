# Benchmark Dataset

## Requirements


### Coverage
#### CWEs

2. The dataset MUST include 
   1. all CWEs used in practice i.e. a CWE is used in at least 1 published CVE(s) (~~400)
   2. all CWEs in [CWE-1003 view](https://cwe.mitre.org/data/definitions/1003.html) (130)
   3. all CWEs in https://cwe.mitre.org/top25/ from [2019](https://cwe.mitre.org/top25/archive/) to [current 2024 CWE Top 25](https://cwe.mitre.org/top25/index.html) 
   4. at least 5 entries per CWE (if such CVE examples exist)
   5. a Balanced subset that has at most 5 entries per CWE
   6. For a given CWE, CVE Descriptions that are sufficiently different per CVE Description 
      1. measured quantitatively using fuzzy and semantic similarity against a threshold
   7. CVEs with multiple CWEs assigned
      1. as reflected by the Top25 datasets
   1. CVE Reference Link Content 
      1. as the raw content 
      2. as a summary e.g. https://github.com/CyberSecAI/cve_info_refs
3. The dataset SHOULD include 
   1. All CWEs (~~1000)
   2. mostly CVEs from recent CVE publication years
       1.  to reflect current reality
   1.  the CVE Description Vulnerability KeyPhrases
       1.  e.g. https://github.com/CyberSecAI/cve_info
   2.  Attack surface diversity that is reflective of reality
       1.  e.g. web, network, mobile, OS kernel, supply-chain
4.  The dataset SHOULD NOT 
    1.  include CWEs that are Deprecated, Obsolete
        1. Over time, CWEs in the dataset may become deprecated or obsolete, and these don't have to be removed


### Quality

1. The dataset MUST be known-good 
   1. i.e. the assigned CWE(s) is correct
2. The dataset MUST be ASCII only??? TBD on the character set to use / not include
3. The dataset MUST include and identify low-quality CVEs 
   1. a sufficient number of low-info CVEs are labelled such that the results for those CVEs can be isolated to e.g. test for Hallucinations / grounding
4. The dataset SHOULD NOT fix existing typos etc... in the CVEs
5. The dataset MUST have a schema and be compliant with that schema


## Data Sources

1. CWE Observed Examples (~~3K)
2. Top25 2023 (~~7K)
3. Top25 2022 (~~7K)

### References
1. https://www.rfc-editor.org/rfc/rfc2119 for definitions of "SHOULD", "MUST",...