# Autonomous SOC
### Plateforme intelligente d'investigation et d'aide à la réponse aux incidents de cybersécurité

**Autonomous SOC** est un projet de plateforme d'investigation assistée par intelligence artificielle, inspirée du fonctionnement d'un *Security Operations Center (SOC)*. Son objectif est d'aider les analystes à transformer des alertes et des journaux système ou réseau en investigations structurées, documentées et exploitables.

La plateforme s'appuie sur une architecture multi-agents orchestrée avec **LangGraph**, une approche **Retrieval-Augmented Generation (RAG)** pour exploiter une base de connaissances en cybersécurité, ainsi que des outils spécialisés exposés via le **Model Context Protocol (MCP)**.

> **Positionnement :** Autonomous SOC est un outil d'aide à l'analyse et à la décision. Il distingue les faits observés des hypothèses générées, explicite les limites des données disponibles et soumet toute action sensible à une validation humaine. Il ne remplace pas l'expertise d'un analyste SOC.

---

## Pourquoi Autonomous SOC ?

L'investigation d'un incident peut nécessiter de croiser plusieurs sources : alertes, événements système, trafic réseau, indicateurs de compromission et documentation technique. Cette démarche peut être longue et dispersée.

Autonomous SOC vise à centraliser ces étapes dans un workflow cohérent afin de :

- analyser et normaliser des événements de sécurité ;
- extraire et enrichir des indicateurs de compromission (IOC) : adresses IP, domaines, URL et empreintes de fichiers ;
- consulter une base documentaire spécialisée grâce au RAG ;
- corréler les événements et reconstituer une chronologie ;
- proposer des hypothèses d'investigation et des techniques MITRE ATT&CK candidates ;
- recommander des mesures de réponse en précisant leurs risques et prérequis ;
- générer un rapport traçable, appuyé par des éléments de preuve et des références ;
- signaler les informations manquantes lorsque les éléments disponibles ne permettent pas de conclure.

## Fonctionnement général

```text
        Alertes / Logs / Description d'un incident
                         |
                         v
              Ingestion et normalisation
                         |
                         v
                 Workflow LangGraph
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
  Alert Analyst   Threat Intelligence   Base documentaire
      Agent             Agent              RAG
        |                |                |
        +----------------+----------------+
                         |
                         v
                Investigation Agent
             Corrélation, preuves et timeline
                         |
                         v
              Incident Response Agent
             Recommandations et validation
                         |
                         v
                   Report Agent
                         |
                         v
             Rapport technique et synthèse

       MCP : outils spécialisés d'analyse et d'enrichissement
```

Le workflow coordonne cinq agents spécialisés :

| Agent | Responsabilité |
|---|---|
| **Alert Analyst** | Analyse les alertes et extrait les informations utiles : horodatage, hôte, utilisateur, adresse IP, processus et type d'événement. |
| **Threat Intelligence** | Enrichit les IOC et recherche des informations liées aux vulnérabilités CVE et aux techniques MITRE ATT&CK. |
| **Investigation** | Corrèle les événements, construit une chronologie et distingue les faits, les hypothèses et les informations manquantes. |
| **Incident Response** | Propose des mesures de confinement, d'éradication et de reprise, avec leurs risques et prérequis. |
| **Report** | Produit un rapport technique et une synthèse destinée aux responsables. |

Les agents sont conçus pour produire des résultats structurés. Une hypothèse ne doit jamais être présentée comme un fait sans preuve correspondante.

## Architecture technique

| Composant | Technologie envisagée | Rôle |
|---|---|---|
| Orchestration des agents | **LangGraph** | Gestion de l'état partagé, des transitions, des conditions et des validations humaines. |
| Framework IA | **LangChain** | Chargement documentaire, découpage, embeddings, prompts et sorties structurées. |
| Recherche documentaire | **RAG** | Récupération de passages pertinents avec leurs métadonnées et leurs sources. |
| Outils externes | **MCP** | Exposition d'outils spécialisés aux agents. |
| Backend | **FastAPI** | Exposition des fonctionnalités via une API. |
| Frontend | **React + TypeScript** | Interface web d'investigation et de visualisation. |
| Base vectorielle | **Qdrant** | Indexation et recherche vectorielle des documents. |
| Base relationnelle | **PostgreSQL** | Stockage des incidents, événements et résultats d'investigation. |

### Base de connaissances et RAG

La base documentaire a vocation à regrouper des ressources publiques ou explicitement autorisées, notamment :

- MITRE ATT&CK ;
- guides NIST relatifs à la réponse aux incidents ;
- documentation Wazuh ;
- documentation Suricata et Snort ;
- documentation NVD et informations CVE ;
- procédures et documentation SOC.

Le pipeline documentaire prévu comprend la collecte, l'extraction du texte, le *chunking*, l'ajout de métadonnées, le calcul des embeddings, l'indexation dans Qdrant et la recherche hybride (sémantique et lexicale, notamment BM25). Un mécanisme de *reranking* pourra être ajouté pour améliorer la pertinence des résultats.

Les réponses fondées sur la documentation doivent conserver leurs références. Lorsque les sources disponibles sont insuffisantes, le système doit le signaler plutôt que de produire une réponse non étayée.

### Outils MCP prévus

| Outil | Fonction |
|---|---|
| `analyze_logs` | Analyse et normalisation de journaux. |
| `search_cve` | Recherche d'informations sur une vulnérabilité CVE. |
| `search_attack_technique` | Recherche d'une technique dans MITRE ATT&CK. |
| `get_ioc_reputation` | Recherche de réputation d'un IOC auprès d'une source autorisée. |
| `search_security_docs` | Recherche dans la base documentaire RAG. |
| `generate_incident_report` | Génération d'un rapport à partir des résultats d'investigation. |

## Exemple de scénario d'investigation

Un scénario de démonstration pourra s'appuyer sur la séquence illustrative suivante :

| Heure | Événement observé |
|---|---|
| 10:01 | Communication avec une adresse IP externe inhabituelle. |
| 10:03 | Série d'échecs d'authentification sur un compte administrateur. |
| 10:05 | Connexion réussie depuis la même adresse IP. |
| 10:07 | Exécution d'une commande PowerShell suspecte. |
| 10:09 | Tentative d'accès à un autre serveur interne. |

La plateforme doit rapprocher ces événements, construire une chronologie et présenter séparément les observations, les hypothèses et les éléments qui restent à vérifier.

## Interface utilisateur envisagée

L'interface web est organisée autour de plusieurs espaces :

- **Security Overview** — vue d'ensemble des alertes, incidents et activités ;
- **Investigation Center** — import des journaux, lancement d'une investigation et suivi du workflow ;
- **Threat Intelligence** — consultation des IOC, CVE, techniques MITRE ATT&CK et sources associées ;
- **Incident Reports** — chronologie, preuves, niveau de confiance et recommandations ;
- **Knowledge Base** — consultation des ressources documentaires indexées ;
- **Evaluation Lab** — suivi des benchmarks de recherche et de génération.

## Évaluation

L'évaluation doit mesurer séparément la qualité de la recherche documentaire, de l'extraction des informations et de la génération des rapports.

**Recherche documentaire**
- BM25, recherche vectorielle, recherche hybride et recherche hybride avec reranker ;
- Recall@K, Mean Reciprocal Rank (MRR) et nDCG@K.

**Analyse et génération**
- précision, rappel et F1 pour l'extraction des IOC ;
- fidélité aux sources (*faithfulness*) et pertinence des réponses ;
- exactitude des citations ;
- capacité à s'abstenir lorsque les preuves sont insuffisantes ;
- taux de réussite des appels d'outils, latence et coût.

Un jeu de scénarios annotés devra être préparé afin de rendre les résultats comparables et reproductibles.

## Sources de données et ressources

Les expérimentations pourront s'appuyer sur des jeux de données et référentiels publics :

| Source | Utilisation | Ressource |
|---|---|---|
| CICIDS2017 | Trafic réseau et scénarios d'intrusion | [Site officiel](https://www.unb.ca/cic/datasets/ids-2017.html) |
| CSE-CIC-IDS2018 | Données de trafic et activités malveillantes | [Site officiel](https://www.unb.ca/cic/datasets/ids-2018.html) |
| Splunk BOTS v3 | Scénarios et données orientés investigation SOC | [GitHub](https://github.com/splunk/botsv3) |
| MITRE ATT&CK | Tactiques et techniques adverses | [Site officiel](https://attack.mitre.org/) |
| NVD | Informations sur les vulnérabilités CVE | [Site officiel](https://nvd.nist.gov/) |


## Structure du projet

L'arborescence cible proposée est la suivante et pourra évoluer au fil de l'implémentation :

```text
autonomous-soc/
├── app/
│   └── frontend/                 # Interface React + TypeScript
├── backend/
│   ├── api/                      # API FastAPI
│   ├── schemas/                  # Schémas Pydantic
│   └── services/                 # Services métier
├── agents/                       # Agents IA spécialisés
├── orchestration/                # État et workflow LangGraph
├── rag/                          # Ingestion, indexation et retrieval
├── mcp_server/                   # Serveur MCP et outils
├── data/
│   ├── samples/                  # Logs de démonstration non sensibles
│   └── knowledge_base/           # Documents ou manifestes de sources
├── evaluation/                   # Jeux de données et benchmarks
├── reports/                      # Rapports générés
├── tests/
├── docs/
├── .env.example
├── .gitignore
├── docker-compose.yml
├── requirements.txt
└── README.md
```

## Équipe

À compléter :

- Membre 1 — zaouga wissem
- Membre 2 — khchimi ghada
- Membre 3 — akrem elouer
