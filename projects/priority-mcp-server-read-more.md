# Priority MCP Server – Full Technical Overview

## Overview

Priority MCP Server is a Model Context Protocol server that connects modern AI applications with Priority ERP data through a clean and consistent integration interface. It converts Priority REST OData endpoints into a unified JSON RPC API that is easy to consume by MCP clients, automation tools, and AI agents.

---

## Problem

Priority REST API is powerful but has operational and development challenges such as complex OData queries, variable authentication methods, and repeated integration logic. This project reduces friction and accelerates development.

---

## Solution

The server acts as a unified bridge between Priority ERP and MCP-compatible systems. It handles all the Priority-specific logic, exposing predictable MCP tools for CRUD, queries, and metadata.

---

## Architecture

MCP Client → JSON RPC or SSE → Priority MCP Server → OData → Priority ERP

---

## Features

- Standard MCP tools
- Query, CRUD, entity discovery
- HTTP and SSE transport
- Basic, PAT, OAuth2 authentication
- Docker-ready deployment
- No build step required

---

## MCP Tools

- priority/version.get
- priority/entities.list
- priority/instructions.get
- priority/entity.get
- priority/query.run
- priority/entity.create
- priority/entity.update
- priority/entity.delete

---

## Examples

### List entities
```
{
  "method": "tools/call",
  "params": { "name": "priority/entities.list", "args": {} }
}
```

### Query orders
```
{
  "method": "tools/call",
  "params": {
    "name": "priority/query.run",
    "args": {
      "entity": "ORDERS",
      "top": 10,
      "select": ["ORDNAME", "CUSTNAME", "ORDDATE"],
      "orderby": "ORDDATE desc"
    }
  }
}
```

---

## Deployment

- Local: node dist/index.js
- Docker: docker run -d -p 3000:3000 priority-mcp-server
- Docker Compose
- AWS ECR
- ECS Fargate

---

## Roadmap

- Metadata introspection
- Relationship navigation
- Caching layer
- Multi-tenant configuration
- Web admin UI
- Schema generation for AI
- Streaming cursors

---

## Recruiter Notes

Demonstrates Priority ERP expertise, API architecture skills, Node.js backend engineering, cloud deployment, and MCP ecosystem mastery.

