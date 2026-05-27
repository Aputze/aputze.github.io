# Priority Developer Agent - Technical Overview

**Author:** Sergei Lerner  
**Status:** Production-ready agent, actively developed  
**Scope:** AI-driven Priority ERP module development

---

## Problem Statement

Building a custom Priority ERP module - from DBI table definitions through EFORM screens, EREP reports, OData exposure, and demo data - is a highly repetitive, multi-step process that requires deep knowledge of Priority's proprietary toolchain, SDK semantics, and UI patterns.

Developers must navigate:

1. **Complex upgrade workflows** - DBI revisions, screen generation via FORMPREP, report configuration, and activation - each with its own syntax and failure modes
2. **Scattered reference material** - SDK documentation, instance-specific quirks, and community knowledge across multiple formats and languages (primarily Hebrew)
3. **Brittle API surfaces** - OData and WebSDK each have distinct capabilities, error modes, and routing constraints; mixing them incorrectly causes silent failures
4. **No standard module template** - Each module is built from scratch with no reusable scaffolding, leading to inconsistency and rework

The Priority Developer Agent solves all four by encoding the full module-building workflow into an AI agent with structured skills, a curated knowledge base, and an MCP-only execution model.

---

## Architecture - Agent Runtime Model

```
User Request (free text)
        │
        ▼
  Agent Persona (System Prompt)
  Identity: "You are a Priority module developer"
  Mandatory 5-step workflow
  DeepSeek-first reasoning
  MCP-only discipline
        │
        ├── Knowledge Layer
        │   ├── Priority KB (local docs)
        │   ├── Reference PDFs (official SDK)
        │   ├── Entity capability maps
        │   ├── Failure catalog
        │   └── Routing heuristics
        │
        ├── Skill Layer
        │   ├── PRIORITY_DEVELOPER_AGENT.md (567 lines)
        │   │   └── Routing decision tree, capability matrix, diagnosis
        │   └── PRIORITY_MODULE_BUILDER.md (655 lines)
        │       └── Step-by-step DBI/EFORM/EREP/OData patterns
        │
        └── Execution Layer
            ├── MCP Gateway (OData + WebSDK)
            ├── Agent Tools (entity-scanner, doc-mapper, websdk-runner)
            └── Runtime Context API (decision loop, capability registry)
```

---

## The 5-Step Mandatory Workflow

Every module request goes through this structured process:

### Step 1 - Requirements Analysis
- Parse free-text request into structured requirements
- Identify entities, fields, relationships, screen layouts, reports
- Surface ambiguity and ask clarifying questions
- Output: Structured requirements document

### Step 2 - Specification Generation
- Produce detailed spec with table definitions, screen mockups, report layouts
- Include field types, lengths, mandatory constraints, key definitions
- Map Hebrew UI captions to API field names
- **Checkpoint:** Human review and approval required before execution

### Step 3 - Module Building (Execution)
- Create DBI tables via Upgrade Revisions workflow
- Build EFORM screens with proper field placement, subforms, Choose triggers
- Configure EREP reports with sorting, grouping, and filtering
- Activate via FORMPREP
- All execution through MCP tools - no direct database access

### Step 4 - OData Exposure
- Configure entity exposure via Priority OData API
- Validate that entity, fields, and filters work correctly
- Register in capability registry with read/create/update/delete risk levels

### Step 5 - Demo Data & Documentation
- Generate demo data via WebSDK procedures
- Document the module, its entities, and usage patterns
- Update failure catalog with any new patterns discovered

---

## Key Governance - MCP-Only Execution

The agent operates under strict MCP-only discipline:

- **No direct HTTP calls** - all Priority interactions go through MCP servers
- **No direct database writes** - all writes use Priority's own upgrade and screen generator APIs
- **No free-form OData** - only whitelisted entities and bounded queries
- **Human checkpoint gates** at spec, execution, and demo data stages

---

## Capability Registry

Every entity has a documented risk profile:

| Risk Level | Meaning | Examples |
|-----------|---------|----------|
| **Read** | Safe to query | `SUBSCRIBERS`, `CUSTOMERS` |
| **Create** | Can insert new records | `SERG_CITIES` |
| **Update** | Can modify existing records | `CUSTOMERS` (with lifecycle validation) |
| **Delete** | Restricted - requires special approval | None by default |

The capability registry (`docs/runtime/mcp-capability-registry.json`) maps each entity to its supported operations and associated risk.

---

## Responsible AI Design

| Risk | Mitigation |
|------|------------|
| Invalid field values | Input validation against Priority dictionaries before MCP calls |
| Workflow steps skipped | Mandatory 5-step workflow with checkpoint gates |
| Wrong API routing | Routing heuristics enforce OData vs WebSDK decision tree |
| Knowledge gaps | Curated KB + reference PDFs + failure catalog prevent guesswork |
| Module inconsistency | Standardized DBI/EFORM/EREP patterns in skill documents |
| No audit trail | Session logging and structured iteration records |

---

## Products Built by the Agent

The Priority Developer Agent has been used to build production Priority modules, including custom entity management screens, data entry forms with Choose/search triggers, and business report configurations - all from free-text requirements in minutes instead of days.

---

## Technology Stack

| Component | Technology |
|-----------|-----------|
| MCP Servers | Node.js, Express |
| OData integration | Priority OData API |
| WebSDK integration | Priority Web SDK |
| AI/LLM | Anthropic Claude (primary), DeepSeek (reasoning delegation) |
| Knowledge base | Local documentation, official Priority PDFs |
| Agent tools | Node.js (entity-scanner, doc-mapper, websdk-runner) |
| Infrastructure | GitHub (operational truth), Notion (executive dashboard) |
| Automation | PowerShell, Git hooks (secret scanning) |

---

## Design Principles

**1. MCP-only execution**  
The agent never makes direct HTTP calls or database writes. All Priority operations go through the MCP gateway, ensuring business logic enforcement and auditability.

**2. DeepSeek-first reasoning**  
Planning, architecture, and debugging are delegated to DeepSeek MCP before the main model engages - reducing cost and improving decision quality.

**3. Structured over ad-hoc**  
Every module follows the same 5-step workflow. Standardization eliminates guesswork and ensures consistent quality.

**4. Human in the loop**  
Checkpoint gates at spec, execution, and demo data stages ensure no automated decision reaches production without review.

**5. Knowledge over guesswork**  
The agent does not improvise Priority patterns. It consults curated knowledge bases, reference PDFs, and documented failure catalogs before acting.

**6. Iterative refinement**  
The agent evolves through structured iterations with clear milestones - each iteration adds capabilities, fixes gaps, and updates documentation.

---

*Sergei Lerner - [LinkedIn](https://www.linkedin.com/in/sergei-lerner/) · [Portfolio](https://aputze.github.io)*
