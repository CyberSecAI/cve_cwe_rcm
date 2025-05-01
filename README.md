## Why

### Background
Accurate CVE-CWE Root Cause Mapping facilitates trend analysis, investment prioritization, exploitability insights, and SDLC feedback. 
But there are known mapping quality issues.

### Purpose
To address CVE-CWE Root Cause Mapping Quality issues by enabling a Benchmark dataset and Leaderboard that allows sharing of common measurements and improvement of tools/technology.

Advancing CWE mapping technology, showcasing technology and talent via a leaderboard based on the benchmark, and ultimately improving CVE-CWE data quality for us all!

### Vision
**The** CVE CWE Root Cause Mapping Benchmark
1. that allows measurement and improvement of tools/processes to address the CVE-CWE Quality issues: from ChatBots to Bulk assignment/validation.
2. used by industry and academia
3. endorsed by MITRE CWE as being fit for purpose
4. that a Leaderboard is based on where people showcase their tools and talent against the CVE CWE Root Cause Mapping Benchmark
5. that promotes doing the right thing e.g. using the Reference content, and additional CVE info to do CWE mapping
6. reflects real world usage

---
## What and When

### Deliverables Q1 2026
1. a presentation submittal for FIRST VulnCon 2026 (Scottsdale, AZ is the venue)
2. a paper
3. a published known-good CWE-RCM dataset that can be used by industry and academia to improve CWE RCM. 
4. a published leaderboard where people can compete using the benchmark with results from at least 3 tools.


---

## How

### Resources We Have
1. Access to the non-public MITRE CWE Top25 2023, 2022 datasets + guidance/expertise of the MITRE CWE team + participation in the MITRE CWE RCMWG.
2. Top SME participants: Data scientists, architects, developers, academics.
3. Access to community


### Components

#### Exploratory Data Analysis 
1. Shows CWEs used (in published CVEs) by count

#### Benchmark Dataset
1. See [dataset requirements](dataset.md) that define characteristics that address [Limitations](https://cybersecai.github.io/cti_models/cyber_security_models/#limitations) of existing RCM datasets.

#### Benchmark Comparison Algorithm
1. Define algorithm that supports comparison of CVE-CWEs where there are:
   - degrees of matching e.g. exact match, direct ChildOf/ParentOf, indirect ChildOf/ParentOf, Cousin,...
   - more than 1 CWE in the benchmark and assigned CWEs
   - comparison by Abstraction
   - Metrics e.g. Balanced Accuracy, Precision, Recall, F1 Score, Exact Match Rate, At Least One Match Rate
   - Grounding / Hallucination included? e.g. very low info CVEs

#### Guidance on Implementing Tools
1. Based on learnings to date e.g. from the solution presented at VulnCon.

#### Leaderboard
1. Hosting e.g. HuggingFace
2. Battle format and algorithm e.g. https://huggingface.co/spaces/lmarena-ai/chatbot-arena-leaderboard
3. UX
4. Requirements to participate e.g. OSS License

### Milestones
- [ ] May 15: Get list of committed collaborators
- [ ] Define housekeeping: wiki (github), comms channel (slack, discord), ticket tracking (github).... 
- [ ] Define roadmap / milestones
- [ ] Define user scenarios, roles