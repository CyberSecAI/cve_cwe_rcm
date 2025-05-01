# Why

## Background
Accurate CVE-CWE Root Cause Mapping (RCM) facilitates trend analysis, investment prioritization, exploitability insights, and SDLC feedback. 

Per [Vulnerability Root Cause Mapping with CWE: Challenges, Solutions, and Insights from Grounded LLM-based Analysis](https://www.first.org/conference/vulncon2025/program#pVulnerability-Root-Cause-Mapping-with-CWE-Challenges-Solutions-and-Insights-from-Grounded-LLM-based-Analysis), the Value of RCM is that it:
* Enables vulnerability trend analysis and greater visibility into their patterns over time
* Illuminates where investments, policy, and practices can address the weaknesses responsible for product vulnerabilities so that they can be eliminated
* Provides further insight to potential “exploitability” based on weakness type
* Provides valuable feedback loop into an SDLC or architecture design planning

But there are known mapping quality issues.

## Purpose
To address CVE-CWE Root Cause Mapping Quality issues by enabling a Benchmark dataset and Leaderboard that allows sharing of common measurements and improvement of tools/technology.

Advancing CWE mapping technology, showcasing technology and talent via a leaderboard based on the benchmark, and ultimately improving CVE-CWE data quality for us all!

To reflect CWE assignment by CWE experts (MITRE CWE Top 25). This includes the Root Cause CWE and other CWEs for a given CVE.

## Vision
**The** CVE CWE Root Cause Mapping Benchmark
1. allows measurement and improvement of tools/processes to address the CVE-CWE Quality issues: from ChatBots to Bulk assignment/validation.
2. is used by industry and academia
3. is endorsed by MITRE CWE as being fit for purpose
4. has an associated Leaderboard where people showcase their tools and talent against the CVE CWE Root Cause Mapping Benchmark
5. promotes doing the right thing e.g. using the Reference content, and additional CVE info to do CWE mapping
6. reflects real world usage

---
# What and When

## Deliverables Q1 2026
1. a presentation submittal for FIRST VulnCon 2026 (Scottsdale, AZ is the venue)
2. a paper
3. a published known-good CWE-RCM dataset that can be used by industry and academia to improve CWE RCM. 
4. a published leaderboard where people can compete using the benchmark with results from at least 3 tools.


---

# How

## Resources We Have
1. Access to the non-public MITRE CWE Top25 2023, 2022 datasets + guidance/expertise of the MITRE CWE team + participation in the MITRE CWE RCMWG.
2. Top SME participants: Data scientists, architects, developers, academics.
3. Access to community


## Components

### Exploratory Data Analysis 
1. Shows CWEs used (in published CVEs) by count

### Benchmark Dataset
See [dataset requirements](dataset.md) that defines characteristics that address [Limitations](https://cybersecai.github.io/cti_models/cyber_security_models/#limitations) of existing RCM datasets.

### Benchmark Comparison Algorithm
See [comparison requirements](comparison.md).

### Guidance on Implementing Tools
1. Based on learnings to date e.g. from the solution presented at VulnCon.

### Leaderboard
1. Hosting e.g. HuggingFace
2. Battle format and algorithm e.g. https://huggingface.co/spaces/lmarena-ai/chatbot-arena-leaderboard
3. UX
4. Requirements to participate e.g. OSS License

## Milestones
- [ ] May 15: Get list of committed collaborators
- [ ] Define housekeeping: wiki (github), comms channel (slack, discord), ticket tracking (github).... 
- [ ] Define roadmap / milestones
- [ ] Define user scenarios, roles


