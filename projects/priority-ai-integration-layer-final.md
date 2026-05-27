# Priority AI Integration Layer - Technical Overview

**Author:** Sergei Lerner  
**Status:** Production-ready architecture, actively developed  
**Scope:** ERP + AI integration platform for Priority ERP

---

## Problem Statement

Priority ERP is a deterministic, rules-based system built for reliability and compliance. Modern LLMs are probabilistic, flexible, and powerful - but they do not understand ERP business logic, data models, or lifecycle rules.

Connecting them naively creates three failure modes:

1. **Hallucinated field values** - the model writes data that does not exist in Priority dictionaries
2. **Bypassed business logic** - the model modifies records directly, skipping mandatory workflow steps
3. **No auditability** - no trace of what the AI decided, why, or what it changed

The Priority AI Integration Layer solves all three.

---

## Architecture - 4-Layer Model

```
User Intent (natural language)
        │
        ▼
  Knowledge Layer (ERP Ontology)
  What does Priority know about this domain?
  Entity capabilities, lifecycle rules,
  routing intelligence, governance rules
        │
        ▼
  Heuristic Layer (Pre-flight Engine)
  Which tool? Which endpoint? Which pattern?
  OData vs WebSDK routing logic
  Known failure path detection
  Tested pattern matching
        │
        ▼
  Operational Layer (MCP Gateway)
  Execution with business-logic awareness
  Not "write field X" - "run the process that updates X"
  OData for CRUD · WebSDK for procedures and form lifecycle
  Human approval checkpoint on sensitive operations
        │
        ▼
  Priority ERP (live)
  Deterministic source of truth
```

---

## Layer 1 - User Intent

The entry point. Users interact in natural language - no ERP knowledge required.

- Free-text input classification
- Intent routing to the correct downstream layer
- Ambiguity detection and clarification prompts
- Language support: Hebrew, English, Russian

---

## Layer 2 - Knowledge Layer (ERP Ontology)

Before the system touches any data, it must understand the domain.

**Sources:**
- Priority official documentation (crawled and indexed)
- Community knowledge base (504MB, Hebrew-first, FAISS vector store)
- Entity capability map: which entities support read, write, filter, subforms
- Lifecycle rules: what must happen before a record can be updated
- Routing intelligence: which interface path is correct for which operation

**Key principle:** The system does not guess. It looks up the rules first.

---

## Layer 3 - Heuristic Layer (Pre-flight Engine)

The decision engine that runs before any execution.

**Routing decisions:**
- OData (port 3000) for reads, CRUD, metadata, counts
- WebSDK (port 3001) for procedures, reports, form lifecycle, Choose/Search triggers
- Never mix base URLs between the two interfaces

**Pattern matching:**
- Does a verified pattern exist for this operation? Use it exactly.
- Is this a known failure path? Apply the documented fix. Do not retry the bad path.
- Is this a new pattern? Perform minimal exploration, then propose a new test case.

**Governance enforcement:**
- Whitelisted entities and fields only
- $top bounded on all queries
- No free-form OData from the model
- Composite key validation before subform operations

---

## Layer 4 - Operational Layer (MCP Gateway)

Two MCP servers expose Priority to AI agents via the Model Context Protocol:

### Priority-REST-API-MCP (OData)
- Entity CRUD: query_run, entity_create, entity_update, entity_delete
- Subform operations with parentLookup resolution
- Batch operations for multi-step writes
- Metadata discovery and schema caching
- Count queries via $top=0&$count=true (never /ENTITY/$count)

### Priority-WebSDK-MCP (WebSDK)
- Form lifecycle: formStart, getRows, fieldUpdate, saveRow
- Procedures: procStart (standalone), activateStart/activateEnd (form action)
- Reports: priority_run_report
- Dictionary lookups for non-API-open entities
- Choose/Search trigger value resolution

### Write-back safety protocol:
1. Validate recommended data against live Priority data
2. Present recommendation with confidence score and red flags
3. Human approval checkpoint - no write without explicit confirmation
4. Execute via business-logic path (WebSDK procedure or OData entity)
5. Verify write via post-execution read
6. Tag record with source field (system-generated) for auditability

---

## Responsible AI Design

| Risk | Mitigation |
|------|------------|
| Invalid field values written by the model | Input validation against Priority dictionaries before execution |
| Workflow steps bypassed by the model | Write-back only via WebSDK procedures that enforce business rules |
| No audit trail | Every system-generated record tagged with source field + timestamp |
| Model drift over time | Payload logging (in/out) + periodic regression testing |
| Low-confidence recommendations | Red flag surfacing in UI + human approval required |
| Entity name mismatch | Hebrew UI caption → API name resolution layer |

---

## Products Built on This Architecture

### Procurement Intelligence Portal
Intelligent purchase recommendation engine for Priority ERP.

- Analyzes sales forecast data against current stock and supplier pricing
- Validates all input data against live Priority records before recommendation
- Surfaces anomaly flags on uncertain data points
- Human approval before any write-back
- Creates draft purchase request (PORDERS) in Priority after approval
- Full audit trail: every system recommendation traceable to source entity

**Measured outcome:** Reduced manual price validation effort, faster procurement cycle.

---

### Priority Developer Agent
A developer agent that builds Priority modules from plain text.

- User describes a requirement in free text
- Agent generates a structured requirements document (tables, screens, reports)
- User reviews and approves the spec
- Agent executes: creates DBI tables via Upgrade Revisions workflow, builds EFORM screens, configures EREP reports, activates via FORMPREP
- Full module delivered end-to-end in minutes instead of days

**Key governance feature:** Agent operates via MCP only. No direct database access. All writes go through Priority's own upgrade and screen generator APIs.

---

### Priority Developer Chatbot
Conversational assistant for Priority ERP users and developers.

- Natural language queries against live ERP data
- Multi-agent orchestration: query agent, analysis agent, reporting agent
- Telegram bot + React web interface
- Error messages resolved in Hebrew (native language of Priority error system)

---

## Observability and Stability

The platform is designed to remain stable over time, not just at launch.

- **Full payload logging:** every MCP call logs request and response
- **Periodic regression testing:** automated test suite runs against stored payloads to detect drift
- **Known issues catalog:** indexed failure patterns with documented fixes (KNOWN_ISSUES.md)
- **Pattern upgrade protocol:** repeated patterns are promoted from ad-hoc → documented rule → gateway utility → MCP primitive

---

## Technology Stack

| Component | Technology |
|-----------|-----------|
| MCP Servers | Node.js, Express |
| OData integration | Priority OData API, HTTP/Basic auth |
| WebSDK integration | Priority Web SDK, session-based |
| AI/LLM | Anthropic Claude (primary), OpenAI GPT |
| Vector store | FAISS, multilingual embeddings |
| Backend | Python, FastAPI |
| Frontend | React, Tailwind |
| Infrastructure | Docker, AWS EC2, Nginx |
| Orchestration | LangChain, LangGraph |
| Automation | n8n |

---

## Design Principles

**1. Deterministic before probabilistic**  
Every AI action is preceded by a lookup in the knowledge layer. The AI does not guess what is valid in Priority - it knows.

**2. Business logic is non-negotiable**  
Write-back always goes through Priority's own workflow paths. The AI cannot shortcut what a human cannot shortcut.

**3. Human in the loop on sensitive operations**  
Financial records, inventory adjustments, and procurement decisions require explicit human approval before execution.

**4. Failure is a known state**  
The system maintains a catalog of known failure patterns. When a failure occurs, it is classified, documented, and fixed once - not retried blindly.

**5. Stable growth over feature velocity**  
Technology is 20% of the change and 80% process change. The platform grows by proving value in one process, then expanding - never by adding features nobody uses.

---

*Sergei Lerner - [LinkedIn](https://www.linkedin.com/in/sergei-lerner/) · [Portfolio](https://aputze.github.io)*
