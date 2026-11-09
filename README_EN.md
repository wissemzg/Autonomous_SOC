# Autonomous SOC
### Intelligent Cybersecurity Incident Investigation and Response Support Platform

**Autonomous SOC** is an AI-assisted incident investigation platform inspired by the operations of a *Security Operations Center (SOC)*. Its goal is to help security analysts turn alerts and system or network logs into structured, documented, and actionable investigations.

The platform is built around a multi-agent architecture orchestrated with **LangGraph**, a **Retrieval-Augmented Generation (RAG)** approach for querying a cybersecurity knowledge base, and specialized tools exposed through the **Model Context Protocol (MCP)**.

> **Positioning:** Autonomous SOC is an analysis and decision-support tool. It distinguishes observed facts from generated hypotheses, highlights limitations in the available data, and requires human approval for sensitive actions. It is not a replacement for the expertise of a SOC analyst.

---

## Why Autonomous SOC?

Investigating a security incident often requires correlating multiple sources: alerts, system events, network traffic, indicators of compromise, and technical documentation. This process can be time-consuming and fragmented.

Autonomous SOC aims to bring these steps together in a consistent workflow in order to:

- analyze and normalize security events;
- extract and enrich Indicators of Compromise (IOCs), such as IP addresses, domains, URLs, and file hashes;
- retrieve information from a cybersecurity knowledge base using RAG;
- correlate events and reconstruct an incident timeline;
- suggest investigation hypotheses and candidate MITRE ATT&CK techniques;
- recommend response actions, including their risks and prerequisites;
- generate traceable incident reports supported by evidence and references;
- identify missing information when the available evidence is insufficient to reach a conclusion.

## Overview of the Workflow

```text
             Alerts / Logs / Incident Description
                            |
                            v
                 Ingestion and Normalization
                            |
                            v
                    LangGraph Workflow
                            |
         +------------------+------------------+
         |                  |                  |
         v                  v                  v
    Alert Analyst    Threat Intelligence   Knowledge Base
       Agent                Agent                RAG
         |                  |                  |
         +------------------+------------------+
                            |
                            v
                   Investigation Agent
                Correlation, Evidence, Timeline
                            |
                            v
                Incident Response Agent
                 Recommendations and Review
                            |
                            v
                      Report Agent
                            |
                            v
              Technical Report and Executive Summary

       MCP: Specialized analysis and enrichment tools
```

The workflow coordinates five specialized agents:

| Agent | Responsibility |
|---|---|
| **Alert Analyst** | Analyzes alerts and extracts relevant information, including timestamps, hosts, users, IP addresses, processes, and event types. |
| **Threat Intelligence** | Enriches IOCs and retrieves information related to CVE vulnerabilities and MITRE ATT&CK techniques. |
| **Investigation** | Correlates events, builds an incident timeline, and distinguishes facts, hypotheses, and missing information. |
| **Incident Response** | Recommends containment, eradication, and recovery measures, including their risks and prerequisites. |
| **Report** | Generates a technical report and an executive summary for decision-makers. |

Agents are designed to return structured results. A hypothesis must never be presented as a fact without supporting evidence.

## Technical Architecture

| Component | Planned Technology | Purpose |
|---|---|---|
| Agent orchestration | **LangGraph** | Manages shared state, transitions, conditional routing, and human approval steps. |
| AI framework | **LangChain** | Handles document loading, chunking, embeddings, prompts, and structured outputs. |
| Knowledge retrieval | **RAG** | Retrieves relevant document passages together with their metadata and sources. |
| External tools | **MCP** | Exposes specialized tools to the agents. |
| Backend | **FastAPI** | Provides the application API. |
| Frontend | **React + TypeScript** | Provides the investigation and visualization interface. |
| Vector database | **Qdrant** | Stores and retrieves document embeddings. |
| Relational database | **PostgreSQL** | Stores incidents, events, and investigation results. |

### Knowledge Base and RAG

The knowledge base is intended to include public or explicitly authorized resources, such as:

- MITRE ATT&CK;
- NIST incident response guidelines;
- Wazuh documentation;
- Suricata and Snort documentation;
- NVD documentation and CVE information;
- SOC procedures and technical documentation.

The planned document pipeline includes collection, text extraction, *chunking*, metadata enrichment, embedding generation, indexing in Qdrant, and hybrid retrieval (semantic and lexical search, including BM25). A reranking mechanism may also be added to improve retrieval relevance.

Answers based on documentation should retain their references. When the available sources are insufficient, the system should indicate this rather than generate an unsupported answer.

### Planned MCP Tools

| Tool | Function |
|---|---|
| `analyze_logs` | Analyzes and normalizes log data. |
| `search_cve` | Retrieves information about a CVE vulnerability. |
| `search_attack_technique` | Searches for a technique in MITRE ATT&CK. |
| `get_ioc_reputation` | Checks an IOC's reputation through an authorized source. |
| `search_security_docs` | Searches the RAG knowledge base. |
| `generate_incident_report` | Generates a structured report from investigation results. |

## Example Investigation Scenario

A demonstration scenario may use the following illustrative sequence:

| Time | Observed Event |
|---|---|
| 10:01 | Communication with an unusual external IP address. |
| 10:03 | Multiple failed authentication attempts on an administrator account. |
| 10:05 | Successful login from the same IP address. |
| 10:07 | Execution of a suspicious PowerShell command. |
| 10:09 | Attempt to access another internal server. |

The platform should correlate these events, build a timeline, and present observations, hypotheses, and items requiring further verification separately.

## Planned User Interface

The web interface is organized around several workspaces:

- **Security Overview** — overview of alerts, incidents, and security activity;
- **Investigation Center** — log import, investigation launch, and workflow progress tracking;
- **Threat Intelligence** — IOCs, CVEs, MITRE ATT&CK techniques, and related sources;
- **Incident Reports** — timelines, evidence, confidence levels, and recommendations;
- **Knowledge Base** — access to indexed documentation resources;
- **Evaluation Lab** — retrieval and generation benchmark results.

## Evaluation

Evaluation should measure the quality of document retrieval, information extraction, and report generation separately.

**Document retrieval**
- BM25, vector search, hybrid search, and hybrid search with reranking;
- Recall@K, Mean Reciprocal Rank (MRR), and nDCG@K.

**Analysis and generation**
- precision, recall, and F1 for IOC extraction;
- source faithfulness and answer relevance;
- citation accuracy;
- ability to abstain when evidence is insufficient;
- tool-call success rate, latency, and cost.

An annotated set of scenarios should be prepared to make evaluation results comparable and reproducible.

## Datasets and Resources

Experiments may use public cybersecurity datasets and reference frameworks:

| Source | Use Case | Resource |
|---|---|---|
| CICIDS2017 | Network traffic and intrusion scenarios | [Official website](https://www.unb.ca/cic/datasets/ids-2017.html) |
| CSE-CIC-IDS2018 | Network traffic data and malicious activity | [Official website](https://www.unb.ca/cic/datasets/ids-2018.html) |
| Splunk BOTS v3 | SOC investigation scenarios and datasets | [GitHub](https://github.com/splunk/botsv3) |
| MITRE ATT&CK | Adversary tactics and techniques | [Official website](https://attack.mitre.org/) |
| NVD | CVE vulnerability information | [Official website](https://nvd.nist.gov/) |

## Project Structure

The proposed target structure is shown below and may evolve as implementation progresses:

```text
autonomous-soc/
├── app/
│   └── frontend/                 # React + TypeScript interface
├── backend/
│   ├── api/                      # FastAPI application
│   ├── schemas/                  # Pydantic schemas
│   └── services/                 # Business logic and services
├── agents/                       # Specialized AI agents
├── orchestration/                # LangGraph state and workflow
├── rag/                          # Ingestion, indexing, and retrieval
├── mcp_server/                   # MCP server and tools
├── data/
│   ├── samples/                  # Non-sensitive sample logs
│   └── knowledge_base/           # Documents or source manifests
├── evaluation/                   # Datasets and benchmarks
├── reports/                      # Generated reports
├── tests/
├── docs/
├── .env.example
├── .gitignore
├── docker-compose.yml
├── requirements.txt
└── README.md
```

## Team

To be completed:

- Member 1 — zaouga wissem
- Member 2 — khchimi ghada
- Member 3 — akrem elouer