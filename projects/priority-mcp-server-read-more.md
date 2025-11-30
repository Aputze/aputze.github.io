# Priority MCP Server – Full Technical Overview

## Overview

Priority MCP Server is a Model Context Protocol server that connects modern AI applications with Priority ERP data through a clean and consistent integration interface. It converts Priority REST OData endpoints into a unified JSON-RPC 2.0 API that is easy to consume by MCP clients, automation tools, and AI agents.

---

## Problem

Priority REST API is powerful but has operational and development challenges such as complex OData queries, variable authentication methods, repeated integration logic, and composite key handling. This project reduces friction and accelerates development by providing a standardized interface with built-in error handling, subform support, and automatic API access detection.

---

## Solution

The server acts as a unified bridge between Priority ERP and MCP-compatible systems. It handles all the Priority-specific logic, exposing predictable MCP tools for CRUD, queries, metadata, batch operations, and API configuration. The server enforces strict data integrity policies—no mock data is ever returned, ensuring all data comes directly from the Priority ERP system.

---

## Architecture

```
MCP Client → JSON-RPC 2.0 or SSE → Priority MCP Server → OData → Priority ERP
```

**Transport Options:**
- HTTP POST/GET endpoints (`/mcp`)
- Server-Sent Events (SSE) for streaming responses
- Standard JSON-RPC 2.0 protocol

---

## Key Features

- **Standard MCP Tools**: Full MCP protocol compliance with JSON-RPC 2.0
- **Comprehensive CRUD**: Create, read, update, delete operations with subform support
- **Advanced Querying**: OData filtering, sorting, pagination, field selection, and subform expansion
- **Batch Operations**: Execute multiple operations in a single request with dependencies
- **Entity Discovery**: List all entities, filter API-accessible entities, get entity schemas
- **API Configuration**: Enable/disable REST API access for forms via FORMLIMITED
- **Metadata Management**: Refresh entity metadata when customizations are added
- **Data Integrity**: Strict no-mock-data policy—all data must come from Priority ERP
- **Multiple Authentication**: Basic auth, Personal Access Tokens (PAT), OAuth2, per-app licensing
- **Transport Options**: HTTP and optional Server-Sent Events (SSE)
- **Docker-Ready**: Containerized deployment with Docker and Docker Compose
- **AWS Deployment**: Full support for EC2, ECR, and ECS Fargate
- **No Build Step**: Runs directly from compiled JavaScript in `dist/` directory

---

## MCP Tools

### Core Tools

- **`priority_version.get`** - Fetch Priority service version and relevant headers
- **`priority_entities.list`** - List all OData entity sets (optionally filter to API-accessible only)
- **`priority_api_entities.list`** - List entities available via REST API (RESTFLAG=Y in FORMLIMITED)
- **`priority_instructions.get`** - Return comprehensive guidance for interacting with Priority REST API
- **`priority_entity_schema.get`** - Get metadata and schema information for an entity

### Query & Read Operations

- **`priority_entity.get`** - Fetch a single entity by key with optional subform expansion
- **`priority_query.run`** - Run OData queries with filtering, sorting, pagination, and subform expansion

### Write Operations

- **`priority_entity.create`** - Create entities with support for related subforms
- **`priority_entity.update`** - Update entities and subforms (supports composite keys)
- **`priority_entity.delete`** - Delete entities and subforms (supports composite keys)
- **`priority_batch_operations`** - Execute multiple create/update/delete operations with dependencies

### Configuration Tools

- **`priority_formlimited_restflag.update`** - Enable/disable REST API access for forms (handles composite keys automatically)
- **`priority_metadata_refresh`** - Clear and refresh metadata for entities (useful after customizations)

---

## Examples

### List All Entities
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "priority_entities.list",
    "args": {}
  }
}
```

### List API-Accessible Entities Only
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "priority_entities.list",
    "args": {
      "apiOnly": true
    }
  }
}
```

### Query Orders with Filtering
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "priority_query.run",
    "args": {
      "entity": "ORDERS",
      "top": 10,
      "select": ["ORDNAME", "CUSTNAME", "ORDDATE"],
      "filter": "STATUS eq 'Active'",
      "orderby": "ORDDATE desc"
    }
  }
}
```

### Get Order with Subform Expansion
```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "priority_entity.get",
    "args": {
      "entity": "ORDERS",
      "key": "SO15000005",
      "select": ["ORDNAME", "CUSTNAME", "CDES"],
      "expand": "ORDERITEMS_SUBFORM,SHIPTO2_SUBFORM"
    }
  }
}
```

### Create Order with Related Items
```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "tools/call",
  "params": {
    "name": "priority_entity.create",
    "args": {
      "entity": "ORDERS",
      "data": {
        "CUSTNAME": "007",
        "ORDERITEMS_SUBFORM": [
          {
            "PARTNAME": "111-001",
            "DUEDATE": "2016-08-01T00:00:00+03:00"
          }
        ]
      }
    }
  }
}
```

### Batch Operations
```json
{
  "jsonrpc": "2.0",
  "id": 6,
  "method": "tools/call",
  "params": {
    "name": "priority_batch_operations",
    "args": {
      "requests": [
        {
          "id": "1",
          "method": "POST",
          "url": "ORDERS",
          "body": { "CUSTNAME": "T000001" }
        },
        {
          "id": "2",
          "method": "POST",
          "url": "$1/ORDERITEMS_SUBFORM",
          "dependsOn": ["1"],
          "body": {
            "PARTNAME": "MS0001",
            "DUEDATE": "2022-08-01T00:00:00+03:00"
          }
        }
      ]
    }
  }
}
```

### Enable API Access for a Form
```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "method": "tools/call",
  "params": {
    "name": "priority_formlimited_restflag.update",
    "args": {
      "formName": "WAREHOUSES",
      "restFlag": "Y"
    }
  }
}
```

---

## Data Integrity Policy

**CRITICAL**: The server enforces a strict **NO MOCK DATA** policy. All data must come directly from the Priority ERP system via authenticated API calls. When data cannot be fetched, operations **MUST FAIL** with clear error messages. **NEVER** returns empty arrays, empty objects, or placeholder data as fallback.

**Key Principles:**
- All data must originate from real Priority ERP API calls
- Fail fast with clear error messages
- Runtime validation ensures no mock data is returned
- Full audit trail of all API calls

See [Data Integrity Policy Documentation](https://github.com/Aputze/PriorityMCP/blob/main/docs/DATA_INTEGRITY_POLICY.md) for complete details.

---

## Deployment

### Local Execution
```bash
node dist/index.js
```

### Docker
```bash
cd deployment/local/docker
docker-compose up -d --build
```

### AWS Deployment
- **EC2**: Direct deployment to EC2 instances
- **ECR**: Container registry for Docker images
- **ECS Fargate**: Serverless container deployment

**Quick Start:**
```bash
# Push to ECR
./deployment/aws/scripts/push-to-ecr.sh

# Deploy to EC2
./deployment/aws/scripts/deploy-to-ec2.sh
```

### Configuration

Environment variables for Priority API connection:
- `PRIORITY_BASE_URL` - Priority OData service root URL (required)
- `PRIORITY_AUTH_TYPE` - Authentication method: `basic`, `pat`, `oauth2`, or `none`
- `PRIORITY_USERNAME` / `PRIORITY_PASSWORD` - For Basic auth
- `PRIORITY_PAT` - Personal Access Token or OAuth2 bearer token
- `PRIORITY_APP_ID` / `PRIORITY_APP_KEY` - For per-app licensing
- `HTTP_PORT` - Server port (default: 3000)
- `SSE_ENABLED` - Enable Server-Sent Events transport
- `STRICT_DATA_INTEGRITY` - Enable strict data integrity checks (default: true)

---

## Project Structure

```
PriorityMCP/
├── dist/                    # Compiled JavaScript (no build step required)
│   ├── index.js             # Main entry point
│   ├── server.js            # Express HTTP server
│   ├── sseServer.js         # SSE transport support
│   ├── mcp/                 # MCP protocol handlers
│   ├── priority/            # Priority API client
│   └── tools/               # MCP tool definitions
├── src/                     # Source code
│   ├── index.js
│   ├── server.js
│   ├── sseServer.js
│   ├── config.js
│   ├── mcp/                 # MCP handlers and registry
│   ├── priority/            # Priority OData client
│   ├── tools/               # Tool implementations
│   └── utils/               # Error handling, data integrity
├── deployment/              # Deployment configurations
│   ├── local/               # Docker deployment
│   └── aws/                 # AWS deployment scripts
├── docs/                    # Documentation
│   └── DATA_INTEGRITY_POLICY.md
├── tests/                   # Test suite and reports
└── package.json
```

---

## Advanced Features

### Subform Support
All tools support expanding and manipulating subforms (related entities):
- `ORDERITEMS_SUBFORM` for order line items
- `SHIPTO2_SUBFORM` for shipping addresses
- Multiple subforms can be expanded in a single request

### Composite Key Handling
Automatic handling of composite keys (e.g., `"IVNUM='T9696',IVTYPE='A',DEBIT='D'"` for AINVOICES):
- Tools automatically format composite keys correctly
- `priority_formlimited_restflag.update` handles composite keys automatically

### Automatic API Access Detection
Tools automatically detect when an entity is not accessible via REST API and provide:
- Clear error messages
- Current RESTFLAG value
- Ready-to-use solution with correct tool and arguments

### Batch Operations with Dependencies
Execute multiple operations in a single request:
- Reference previous request results using `$1`, `$2`, etc.
- Define dependencies between operations
- Get clear error messages for each operation

---

## Integration

### Cursor IDE
Add to `~/.cursor/mcp.json` (Linux/Mac) or `C:\Users\<username>\.cursor\mcp.json` (Windows):

```json
{
  "mcpServers": {
    "priority-mcp": {
      "url": "http://your-server:3000/mcp",
      "allowInsecure": true
    }
  }
}
```

### Other MCP Clients
Any MCP-compatible client can connect by sending JSON-RPC 2.0 messages to the `/mcp` endpoint.

---

## Technology Stack

- **Runtime**: Node.js 18+
- **Framework**: Express.js
- **Protocol**: JSON-RPC 2.0, MCP
- **Transport**: HTTP, Server-Sent Events (SSE)
- **API Client**: Axios
- **Deployment**: Docker, AWS (EC2, ECR, ECS)

---

## Recruiter Notes

This project demonstrates:
- **Priority ERP Expertise**: 13+ years of Priority ERP experience, deep understanding of OData API, FORMLIMITED, subforms, composite keys
- **API Architecture**: Clean separation of concerns, standardized interfaces, comprehensive error handling
- **Node.js Backend Engineering**: Express.js, JSON-RPC 2.0, SSE, async/await patterns
- **Cloud Deployment**: Docker, AWS EC2, ECR, ECS Fargate, deployment automation
- **MCP Ecosystem Mastery**: Model Context Protocol implementation, tool registration, protocol compliance
- **Data Integrity**: Strict policies, runtime validation, audit trails
- **Developer Experience**: Comprehensive documentation, clear error messages, automatic API access detection

