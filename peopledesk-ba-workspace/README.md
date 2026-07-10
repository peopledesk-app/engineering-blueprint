# 🧠 PeopleDesk BA Workspace

### How PeopleDesk works, how we engineer it, and where we are taking it

[![Status](https://img.shields.io/badge/status-active-success?style=flat-square)](#)
[![Scope](https://img.shields.io/badge/scope-high--level%20overview-blue?style=flat-square)](#)
[![Updated](https://img.shields.io/badge/updated-2026--07-informational?style=flat-square)](#)

> [!NOTE]
> This is the **single entry point** for the PeopleDesk BA Workspace concept. It is a complete high-level overview on its own — architecture, engineering approach, and the transformation journey (past → present → future). Detailed documents will be added in subfolders as the concept evolves; this page will always remain the map.

---

## 1. What PeopleDesk is

**PeopleDesk** is an enterprise HRMS SaaS covering nine business domains: **Employee, Attendance, Leave, Payroll, Shift, Roster, Organization, Recruitment, and Approval.**

The product is one system built from five repositories:

| Repository | Role | Stack |
|------------|------|-------|
| `app-api` | Core backend | ASP.NET Core (.NET 9), Clean Architecture, CQRS + MediatR, EF Core, SQL Server |
| `app-front` | Frontend | React + TypeScript, Vite, Redux |
| `app-report-api` | Reporting | .NET reporting service |
| `app-scheduler-api` | Background jobs | .NET scheduler service |
| `peopledesk-business-ai` | **BA Workspace** | Markdown-only AI workspace: Knowledge Base, modes, skills, templates |

```mermaid
flowchart TB

subgraph Product["PeopleDesk Product"]
    FE[app-front<br/>React + TypeScript]
    API[app-api<br/>.NET 9 · Clean Architecture · CQRS]
    RPT[app-report-api]
    SCH[app-scheduler-api]
    DB[(SQL Server)]
end

subgraph AI["BA Workspace (AI Engineering Platform)"]
    BAI[peopledesk-business-ai]
    KB[(Knowledge Base<br/>12 layers)]
end

FE --> API
API --> DB
RPT --> DB
SCH --> DB

BAI --> KB
BAI -.reads source.-> Product
BAI -.read-only.-> DB

classDef env fill:#2563EB,stroke:#1E40AF,color:#FFFFFF
classDef ai fill:#7C3AED,stroke:#5B21B6,color:#FFFFFF
classDef kb fill:#0D9488,stroke:#0F766E,color:#FFFFFF
classDef db fill:#64748B,stroke:#475569,color:#FFFFFF
class FE,API,RPT,SCH env
class DB db
class BAI ai
class KB kb
style Product fill:#2563EB14,stroke:#2563EB
style AI fill:#7C3AED14,stroke:#7C3AED
```

The **BA Workspace** is the fifth repository's purpose: an AI-driven workspace that understands the whole product through a structured **Knowledge Base** and turns business requirements into analyses, specifications, and developer-ready tasks — grounded in how the system *actually* works.

---

## 2. Our engineering approach

One idea drives everything:

> **Make product knowledge explicit and machine-readable, make it the single source of truth, and let AI carry the engineering workload from that foundation — with humans approving every decision that matters.**

In practice:

- **Knowledge first.** The product (source code *and* database) is reverse-engineered into a 12-layer Knowledge Base — from enterprise context and business processes down to data, APIs, UI, security, and test cases. Facts are resolved there first; source code is consulted only for gaps, and every gap is fed back.
- **Analysis before action.** Every change starts with an impact analysis — database, API, UI, permissions, business rules, and regression scope are known before anything is specified or built.
- **Approval-gated automation.** AI proposes; humans approve. Knowledge updates, specifications, and implementation all pass explicit human gates.
- **Safety by construction.** The workspace is markdown-only, database access is read-only, schema changes ship as reviewed SQL scripts, and sensitive money logic with open questions is blocked from automation.
- **Every request makes the platform smarter.** Completed work merges what it learned back into the Knowledge Base — coverage is measured, never assumed.

---

## 3. The journey

Three stages — and the difference between them is **who orchestrates**:

| Stage | AI's role | Orchestration |
|-------|-----------|---------------|
| **3.1 Past** | AI assisted every team — business, product, BA, frontend, backend, QA | Traditional collaboration; humans coordinated everything manually |
| **3.2 Present** | One **AI Workspace** orchestrates the complete lifecycle, grounded in the Knowledge Base | Humans decide at explicit approval gates |
| **3.3 Future** | Customer requests automatically initiate the engineering pipeline | Humans govern a closed, self-improving loop |

> [!NOTE]
> **Color legend for all diagrams:** 🟣 purple = AI-executed · 🟠 amber = human decision / approval · 🩵 teal = knowledge · 🔵 blue = environments & infrastructure · 🟢 green = delivered / live.

### 3.1 Past — AI-assisted teams, manual orchestration

AI was never limited to development — it supported **every stage of the software lifecycle**. What made this the "past" is not the absence of AI, but that each team used AI **separately**, connected by traditional collaboration and manual orchestration:

- **Business & Product Team** — AI-assisted requirement analysis and documentation
- **Business Analyst (BA)** — AI-assisted specification and requirement refinement
- **Frontend Development** — AI-assisted implementation
- **Backend Development** — AI-assisted implementation
- **QA / Testing** — AI-assisted testing and validation

```mermaid
flowchart TB

BP["Business & Product Team<br/>🤖 AI-assisted requirement analysis & documentation"]
BA["Business Analyst<br/>🤖 AI-assisted specification & requirement refinement"]

subgraph DEV[Development]
    FE["Frontend<br/>🤖 AI-assisted implementation"]
    BE["Backend<br/>🤖 AI-assisted implementation"]
end

CI["Build + CI Pipeline<br/>DevOps"]
TEST["Testing Environment<br/>automatic deployment"]
QA["QA / Testing<br/>🤖 AI-assisted testing & validation"]
UATA{Approval}
UAT[UAT Environment]
PRODA{Approval}
PROD[Production Environment]

BP --> BA
BA --> FE
BA --> BE
FE --> CI
BE --> CI
CI -->|automatic| TEST
TEST --> QA
QA -->|bugs found| DEV
QA -->|passed| UATA
UATA -->|approved| UAT
UAT --> PRODA
PRODA -->|approved| PROD

classDef ai fill:#7C3AED,stroke:#5B21B6,color:#FFFFFF
classDef human fill:#F59E0B,stroke:#B45309,color:#1F2937
classDef env fill:#2563EB,stroke:#1E40AF,color:#FFFFFF
classDef ops fill:#64748B,stroke:#475569,color:#FFFFFF
classDef live fill:#16A34A,stroke:#15803D,color:#FFFFFF
class BP,BA,FE,BE,QA ai
class CI ops
class TEST,UAT env
class UATA,PRODA human
class PROD live
style DEV fill:#7C3AED14,stroke:#7C3AED
```

Deployment followed a fixed rhythm: the **Testing Environment deployed automatically** from CI, while **UAT** and **Production** each required an explicit **approval**.

The gaps were *between* the stages, not inside them: every handoff meant queue time, coordination meetings, and knowledge loss. Each team's AI started from zero context, product knowledge lived in people's heads and scattered documents, and every estimate or bug fix paid a "rediscovery tax" — someone re-reading code or finding the person who remembered.

### 3.2 Present — one AI Workspace orchestrates the lifecycle

The **Knowledge-Driven AI Engineering Platform (v2)** is operating now. The disconnected, per-team AI of the past is replaced by a **single AI Workspace** that orchestrates the complete engineering process — grounded in the Knowledge Base as its single source of truth, and gated by explicit human decisions:

```mermaid
flowchart TB

REQ[Business Requirement]
KB[(Knowledge Base<br/>12 layers)]

subgraph WS["🧠 AI WORKSPACE — one orchestrator for the complete lifecycle"]
    IA[Impact Analysis] --> HA{Human Approval}
    HA --> SPEC[Specification Generation]
    SPEC --> TASK[Developer Task Preparation]
    TASK --> EXP{"Explicit AI<br/>Implementation Request"}
    EXP --> IMPL[AI Implementation]
    IMPL --> VER["Verification<br/>build + prescribed tests"]
    VER --> HCR[Human Code Review]
    HCR --> PR[Pull Request Creation]
    PR --> AIPR["Automatic AI PR Review<br/>coding standards · architecture compliance · implementation quality"]
    AIPR --> MA{"Merge Approval<br/>final human review"}
    MA --> DEL[Delivery]
    DEL --> KF[Knowledge Feedback]
    KF --> KBU[Knowledge Base Update]
end

REQ --> IA
KB <-->|single source of truth| WS
KBU -->|approval-gated merge| KB

classDef ai fill:#7C3AED,stroke:#5B21B6,color:#FFFFFF
classDef human fill:#F59E0B,stroke:#B45309,color:#1F2937
classDef kb fill:#0D9488,stroke:#0F766E,color:#FFFFFF
classDef live fill:#16A34A,stroke:#15803D,color:#FFFFFF
classDef input fill:#64748B,stroke:#475569,color:#FFFFFF
class IA,SPEC,TASK,IMPL,VER,PR,AIPR ai
class HA,EXP,HCR,MA human
class DEL live
class KF,KBU,KB kb
class REQ input
style WS fill:#7C3AED14,stroke:#7C3AED
```

Every pull request passes a **dual review** before it can merge:

1. **AI reviews automatically** the moment the PR is created — validating coding standards, checking architecture compliance, and verifying implementation quality.
2. **A human performs the final review** — merge approval always remains a human decision.

And when the Knowledge Base Update lands, one full turn of the lifecycle is complete. The same cycle then repeats for the next request — starting from a smarter Knowledge Base:

```mermaid
flowchart LR

KB[(Knowledge Base)] --> WS[AI Workspace] --> DEV[Development] --> REV[Review] --> DEL[Delivery] --> KF[Knowledge Feedback] --> KB

classDef ai fill:#7C3AED,stroke:#5B21B6,color:#FFFFFF
classDef human fill:#F59E0B,stroke:#B45309,color:#1F2937
classDef kb fill:#0D9488,stroke:#0F766E,color:#FFFFFF
classDef live fill:#16A34A,stroke:#15803D,color:#FFFFFF
class WS,DEV ai
class REV human
class DEL live
class KB,KF kb
```

**Live today:** knowledge-driven analysis and specification, impact analysis before every change, duplicate-feature detection, honest coverage reporting, and approval-gated knowledge sync. **Piloting:** AI-assisted implementation with automatic AI PR review in the product repositories, under human merge approval.

### 3.3 Future — customer-support-driven AI engineering

The destination is an **autonomous, knowledge-driven engineering platform**: customer requests automatically initiate AI-powered requirement analysis, proposal generation, approval workflows, implementation, verification, deployment, and continuous knowledge evolution — one closed engineering loop.

```mermaid
flowchart TB

CUST[Customer] --> PORTAL[Customer Support Portal]
PORTAL --> AIRA[AI Requirement Analysis]
AIRA --> TYPE{Request type?}

subgraph OPTA["Option A — New Feature"]
    A1[AI analyzes the requirement] --> A2[AI prepares a proposal]
    A2 --> A3[Business Analyst reviews the proposal]
    A3 --> A4{Human Approval}
end

subgraph OPTB["Option B — Existing Feature · Bug · Enhancement"]
    B1[AI identifies the root cause] --> B2[AI analyzes the existing implementation]
    B2 --> B3[AI prepares a solution proposal]
    B3 --> B4{"Human Review & Approval"}
end

TYPE -->|new feature| A1
TYPE -->|existing · bug · enhancement| B1

A4 -->|approved| WS["🧠 AI Workspace pipeline<br/>executes the complete Section 3.2 lifecycle"]
B4 -->|approved| WS
WS --> KB[(Knowledge Base)]
KB -->|smarter next request| AIRA

classDef ai fill:#7C3AED,stroke:#5B21B6,color:#FFFFFF
classDef human fill:#F59E0B,stroke:#B45309,color:#1F2937
classDef kb fill:#0D9488,stroke:#0F766E,color:#FFFFFF
classDef env fill:#2563EB,stroke:#1E40AF,color:#FFFFFF
class CUST,PORTAL env
class AIRA,TYPE,A1,A2,B1,B2,B3,WS ai
class A3,A4,B4 human
class KB kb
style OPTA fill:#7C3AED14,stroke:#7C3AED
style OPTB fill:#7C3AED14,stroke:#7C3AED
```

The defining property: **every approved request — new feature or fix — enters the same standardized AI Workspace pipeline defined in Section 3.2.** There is one way software gets built, and the platform is fully self-improving: each delivery updates the Knowledge Base, and the updated knowledge sharpens the next requirement analysis.

The stepping stones remain the **Single Engineer Model** (one engineer supervising the workspace end-to-end) and **supervised autonomy** (AI operating bounded change classes while humans govern policies and audit outcomes). Approval gates never disappear — they move up from individual artifacts to policies.

---

## 4. Transformation roadmap

```mermaid
flowchart TB

P1[Phase 1<br/>Knowledge Foundation] --> P2[Phase 2<br/>Knowledge-Driven Analysis]
P2 --> P3[Phase 3<br/>AI-Assisted Implementation]
P3 --> P4[Phase 4<br/>Single Engineer Model]
P4 --> P5[Phase 5<br/>Supervised Autonomy]

classDef live fill:#16A34A,stroke:#15803D,color:#FFFFFF
classDef pilot fill:#F59E0B,stroke:#B45309,color:#1F2937
classDef future fill:#64748B,stroke:#475569,color:#FFFFFF
class P1,P2 live
class P3 pilot
class P4,P5 future
```

| Phase | Theme | Status |
|-------|-------|--------|
| 1 | Knowledge Foundation — build and validate the 12-layer Knowledge Base | 🟢 In production use |
| 2 | Knowledge-Driven Analysis — specs and impact analyses generated from knowledge | 🟢 In production use |
| 3 | AI-Assisted Implementation — approval-gated code with mandatory verification | 🟡 Piloting |
| 4 | Single Engineer Model — end-to-end delivery by one engineer + AI | ⚪ Planned |
| 5 | Supervised Autonomy — bounded change classes under human governance | ⚪ Future |

Phases advance on **evidence, not calendar**: knowledge coverage and confidence thresholds, verification pass rates, and regression results from pilots decide each promotion. Knowledge building never stops — it is the permanent foundation under every phase.

---

## 5. How this concept grows

This page stays the high-level map. As the concept matures, detail lands in subfolders — for example:

```text
peopledesk-ba-workspace/
├── README.md          ← this overview (always the entry point)
├── diagrams/          ← detailed diagrams, when needed
├── planning/          ← roadmap detail, milestones, research notes
└── designs/           ← architecture and design documents
```

> [!TIP]
> Rule of thumb: if a reader only opens this one page, they should still walk away understanding **what PeopleDesk is, how we engineer it, and where we are taking it.** Everything else is optional depth.
