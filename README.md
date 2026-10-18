# Julian Shalash

**Data Scientist (Applied AI & Python Backend) · Dubai, UAE**

I build AI systems that run in production: LLM chatbots, RAG pipelines and FastAPI services on Azure. This repository is my portfolio, with case studies and architecture, security and data-flow documentation for each project.

**Live site:** [julianshalash.github.io](https://julianshalash.github.io)
**CV:** [Download PDF](assets/Julian_Shalash_CV.pdf)
**Contact:** [YOUR_EMAIL@gmail.com](mailto:YOUR_EMAIL@gmail.com) · [LinkedIn](https://www.linkedin.com/in/YOUR-LINKEDIN/)

---

## Flagship project: Carbon Emission Intelligence Platform

> **Status: in active development.** Scope 3 emissions and the commercial side (pricing, plans, billing) are **not implemented yet** and are documented as planned.

A full-stack platform for tracking and reporting carbon emissions, with a RAG layer that lets users query their own data in plain language.

- **Stack:** Next.js, FastAPI, SQL Server, MongoDB, RAG with vector embeddings, Docker, Azure
- **Data layer:** 10+ relational tables (users, projects, companies) and 12+ MongoDB collections, with schema validation and standardised preprocessing

| Document | What it covers |
|---|---|
| [Project overview](projects/carbon-platform/README.md) | Scope, status, how to run it |
| [System architecture](projects/carbon-platform/docs/system-architecture.md) | Components, deployment, design decisions |
| [Data flow](projects/carbon-platform/docs/data-flow.md) | Ingestion, calculation and RAG query flows |
| [Security overview](projects/carbon-platform/docs/security-overview.md) | Access control, secrets, AI-specific risks |
| [AI architecture](projects/carbon-platform/docs/ai-architecture.md) | RAG pipeline, grounding, limitations |
| [AI readiness](projects/carbon-platform/docs/ai-readiness.md) | Data quality, governance, monitoring |

---

## Other production work

Delivered at Alpin Limited, Dubai.

| Project | What it does | Stack |
|---|---|---|
| [LLM-powered chatbot](projects/llm-chatbot/README.md) | Answers domain-specific questions for about 100 users, with custom prompt engineering and conversation-context management; users and chat history stored in SQL | OpenAI API, Python, SQL |
| [Real-time multilingual translation](projects/translation-service/README.md) | Translates files while preserving their structure; cut translation time by up to 80% | Azure Cognitive Services, FastAPI |
| Emissions reporting dashboards | Power BI and Power Query reports giving stakeholders accurate carbon-emission figures; cut decision time by up to 70–75% | Power BI, Excel Power Query |

---

## Skills

- **Applied AI / ML:** LLMs, OpenAI API, prompt engineering, RAG, vector embeddings, TensorFlow, scikit-learn, regression modelling, model evaluation, reproducible training pipelines
- **Backend & integration:** Python, FastAPI, REST APIs, SQL, Microsoft SQL Server, MongoDB, Postman, Next.js
- **Cloud & DevOps:** Microsoft Azure (Azure AI / Cognitive Services, Storage), Docker, GitHub (version control and CI/CD workflows)
- **Data & reporting:** Pandas, NumPy, Jupyter Notebook, Power BI, Excel Power Query, stakeholder presentations, requirements gathering

## Certifications and education

- TensorFlow Developer Certificate, DeepLearning.AI (Apr 2025)
- Azure ML & MLOps, Udemy (May 2024)
- Data Science Bootcamp, Udemy (Feb 2024)
- IBM Data Analyst Professional, Coursera (Nov 2022)
- MBA, Syrian Virtual University (2023)
- B.Sc. Mechanical Engineering, Damascus University (2018)

---

## Repository structure

```
.
├── index.html            Portfolio landing page (GitHub Pages)
├── README.md             This file
├── assets/               CV and photo
└── projects/
    ├── carbon-platform/  README, docs/, images/
    ├── llm-chatbot/
    └── translation-service/
```

## A note on confidentiality

These projects were built at Alpin Limited. All documentation here is generalised: client names, real data, credentials and internal URLs have been removed.
