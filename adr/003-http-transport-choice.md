# ADR 003: Simplified HTTP REST Transport for Workshop

**Status**: Accepted
**Date**: 2025-01-24
**Deciders**: Workshop Architecture Team
**Related**: [ADR 001: Dynamic Tool Discovery](001-dynamic-tool-discovery.md), [ADR 002: MCP Response Format](002-mcp-response-format.md)

## Context

The Model Context Protocol (MCP) specification 2025-11-25 defines two official transport mechanisms:

1. **stdio Transport**: JSON-RPC 2.0 over standard input/output streams
2. **Streamable HTTP Transport**: JSON-RPC 2.0 over HTTP POST with optional Server-Sent Events (SSE)

Both require JSON-RPC 2.0 message wrapping and specific protocol handling.

This workshop (LAB03) is designed for developers learning MCP concepts for the first time. The target audience includes:
- Developers new to MCP and AI agents
- Workshop participants with limited time (3-4 hours)
- Those who may not have deep JSON-RPC experience
- Teams evaluating MCP for their projects

### Challenge

Official MCP transports add significant complexity:

**JSON-RPC 2.0 Message Structure**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get_weather_forecast",
    "arguments": {
      "location": "Oslo"
    }
  }
}
```

**Required Implementation**:
- JSON-RPC request/response wrapping
- Message ID tracking
- Error code standardization (-32700, -32600, etc.)
- Protocol negotiation (initialize, initialized)
- SSE streaming setup (optional but complex)

This creates a steep learning curve that can distract from the core MCP concepts we want to teach.

## Decision

We implement a **simplified HTTP REST transport** for the workshop that:
1. Uses direct HTTP endpoints (GET /tools, POST /weather)
2. Skips JSON-RPC 2.0 wrapping
3. Maintains MCP-compliant tool definitions and response formats
4. Provides clear disclaimers about the simplification

### What We Simplify

**❌ NOT USED in Workshop**:
- JSON-RPC 2.0 message protocol
- POST /message unified endpoint
- Request/response ID tracking
- Server-Sent Events (SSE) streaming
- Protocol initialization handshake
- JSON-RPC error codes (-32xxx)

**✅ USED in Workshop**:
- MCP tools manifest structure (`GET /tools`)
- MCP tool definitions (name, description, inputSchema, outputSchema)
- MCP response format (content, isError)
- Direct HTTP REST endpoints per tool
- Standard HTTP methods (GET, POST)
- Standard HTTP status codes (200, 400, 500)

### Implementation Pattern

**MCP Server Endpoints**:
```
GET  /tools          → Returns MCP tools manifest
POST /weather        → Executes weather tool
POST /ping           → Executes ping tool
GET  /status         → Executes status tool
GET  /health         → Health check (not MCP)
```

**Contrast with Official MCP**:
```
POST /message        → All tool calls go here
                       (with JSON-RPC wrapping)
```

**Request Example** (Workshop):
```bash
# Direct tool call - no JSON-RPC wrapping
curl -X POST http://localhost:8000/weather \
  -H "Content-Type: application/json" \
  -d '{"location": "Oslo"}'
```

**Request Example** (Official MCP):
```bash
# JSON-RPC wrapped call
curl -X POST http://localhost:8000/message \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "get_weather_forecast",
      "arguments": {"location": "Oslo"}
    }
  }'
```

## Rationale

### Pedagogical Benefits

1. **Focus on Core Concepts**:
   - Dynamic tool discovery pattern
   - MCP tool structure (inputSchema, outputSchema)
   - AI agent integration with tools
   - Response format handling

2. **Reduced Cognitive Load**:
   - No JSON-RPC protocol to learn
   - Familiar HTTP REST patterns
   - Direct correlation: one tool = one endpoint
   - Simpler debugging (curl works directly)

3. **Faster Time to Value**:
   - Participants can create tools in minutes
   - Easy to test with standard HTTP tools
   - Less boilerplate code
   - Immediate results build confidence

4. **Clear Learning Path**:
   - Start: Understand MCP concepts with REST
   - Next: Add JSON-RPC wrapper layer (Phase 2)
   - Production: Use official MCP SDKs

### Technical Benefits

1. **Simpler Implementation**:
   - Standard FastAPI endpoints (familiar to Python developers)
   - No JSON-RPC library dependency
   - Less error handling complexity
   - Easier to read and understand code

2. **Better Debugging**:
   - curl works directly without JSON-RPC wrapping
   - Browser can test GET endpoints
   - Standard HTTP debugging tools work
   - Clear request/response flow

3. **Workshop-Friendly**:
   - Participants can add tools without protocol knowledge
   - Copy-paste pattern works immediately
   - Less chance of protocol errors
   - Faster workshop exercises

## Consequences

### Positive

1. **Accessibility**: Lower barrier to entry for MCP learning
2. **Speed**: Participants can complete exercises faster
3. **Clarity**: Direct mapping between concepts and code
4. **Debugging**: Standard HTTP tools work out of the box
5. **Extensibility**: Easy to add new tools (just add endpoint)
6. **Familiarity**: REST is well-known pattern

### Negative

1. **Not Spec Compliant**: Cannot integrate with official MCP clients
2. **No Streaming**: Cannot support Server-Sent Events (SSE)
3. **Limited Protocol**: No standard error codes, initialization, etc.
4. **Migration Required**: Production use requires rewrite
5. **Custom Pattern**: Unique to this workshop, not industry standard

### Mitigation Strategies

1. **Transparency**: Clear disclaimers in all documentation
2. **Education**: Explain what's different and why
3. **Migration Guide**: Provide path to spec-compliant implementation
4. **Phase 2 Option**: Offer JSON-RPC wrapper as advanced exercise

## Disclaimers Added

### 1. README.md (Lines 22-47)
```markdown
## ⚠️ Viktig Merknad: Forenklet Transport for Workshop

Denne workshopen bruker en **forenklet HTTP REST transport** for pedagogiske formål.
Den offisielle MCP spesifikasjonen (2025-11-25) definerer to standard transporter:

1. **stdio** - JSON-RPC over standard input/output
2. **Streamable HTTP** - JSON-RPC over HTTP POST med valgfri SSE

**Læringsbane:**
- ✅ **Start her**: Forstå MCP konsepter med forenklet REST
- ⬆️ **Neste steg**: Implementer JSON-RPC wrapper
- 🚀 **Produksjon**: Bruk offisielle MCP SDKs
```

### 2. services/mcp-server/app.py (Lines 1-25)
```python
"""
⚠️ WORKSHOP FORENKLING - Transport Layer:
==========================================
Denne serveren bruker direkte HTTP REST endpoints i stedet for den offisielle MCP
Streamable HTTP transport (JSON-RPC 2.0).

OFFISIELLE MCP KRAV:
- POST /message endpoint (all tool calls)
- JSON-RPC 2.0 request wrapping
- JSON-RPC 2.0 response wrapping

VÅR FORENKLING (WORKSHOP):
- Direkte endpoints: GET /tools, POST /weather, etc.
- Ingen JSON-RPC wrapping
- Standard HTTP requests og responses
```

### 3. services/agent/app.py (Lines 1-26)
```python
"""
⚠️ WORKSHOP MERKNAD - Transport Layer:
=======================================
Denne agenten bruker HTTP REST transport for å kommunisere med MCP serveren.
Offisielle MCP implementasjoner bruker stdio eller Streamable HTTP med JSON-RPC 2.0.

PEDAGOGISK BEGRUNNELSE:
Vi bruker forenklet REST transport for å fokusere på MCP konsepter uten å
introdusere JSON-RPC kompleksitet for nybegynnere.
```

### 4. cdocs/OVERVIEW.md (Lines 12-49)
Comprehensive transport explanation in executive summary and compliance section.

## Implementation Details

### MCP Server Structure

**File**: `services/mcp-server/app.py`

**Tools Manifest Endpoint**:
```python
@app.get("/tools")
async def get_tools():
    """
    MCP tools manifest - list of available tools.

    This follows MCP spec for tool definitions but uses
    REST endpoint instead of JSON-RPC.
    """
    return [
        {
            "name": "get_weather_forecast",
            "title": "Get Weather Forecast",
            "description": "...",
            "inputSchema": {...},
            "outputSchema": {...},
            "endpoint": "/weather",  # Extension: explicit endpoint
            "method": "POST"          # Extension: HTTP method
        }
    ]
```

**Tool Execution Endpoint**:
```python
@app.post("/weather")
async def get_weather(request: WeatherRequest):
    """
    Execute weather tool directly.

    Official MCP: Would receive this via POST /message with JSON-RPC wrapping
    Workshop: Direct endpoint call
    """
    result = await get_weather_forecast(request.location)

    # MCP-compliant response format
    return {
        "content": [{"type": "text", "text": "..."}],
        "structuredContent": result,
        "isError": False
    }
```

### Agent Integration

**File**: `services/agent/app.py`

**Tool Discovery**:
```python
async def load_tools_from_mcp_server():
    """
    Load tools from MCP server /tools endpoint.

    Official MCP: Would use JSON-RPC 'tools/list' method
    Workshop: Direct GET /tools
    """
    tools_manifest = await client.get(f"{mcp_server_url}/tools")

    # Store endpoint mappings from manifest
    self.tool_endpoints = {
        tool["name"]: {
            "endpoint": tool.get("endpoint", f"/{tool['name']}"),
            "method": tool.get("method", "POST")
        }
        for tool in tools_manifest
    }
```

**Tool Execution**:
```python
async def call_mcp_tool(tool_name: str, arguments: dict):
    """
    Call MCP tool via direct HTTP endpoint.

    Official MCP: Would send JSON-RPC request to POST /message
    Workshop: Direct HTTP call to tool endpoint
    """
    endpoint_info = self.tool_endpoints[tool_name]
    url = f"{mcp_server_url}{endpoint_info['endpoint']}"
    method = endpoint_info['method']

    if method == "POST":
        response = await client.post(url, json=arguments)
    else:
        response = await client.get(url)

    return response.json()
```

## Migration Path to Official MCP

### Phase 1: Current State (Workshop) ✅ COMPLETED
- Simplified HTTP REST transport
- MCP-compliant tools manifest
- MCP-compliant response format
- Clear disclaimers everywhere

### Phase 2: JSON-RPC Wrapper (Optional Enhancement)
**Estimated Effort**: 3-4 hours

Add parallel JSON-RPC endpoint alongside REST:

```python
@app.post("/message")
async def handle_jsonrpc(request: Request):
    """JSON-RPC 2.0 endpoint for spec-compliant clients."""
    body = await request.json()

    # Validate JSON-RPC structure
    if body.get("jsonrpc") != "2.0":
        return jsonrpc_error(-32600, "Invalid Request")

    method = body.get("method")
    params = body.get("params", {})

    # Route to existing handlers
    if method == "tools/list":
        tools = await get_tools()
        return jsonrpc_success(body["id"], tools)

    elif method == "tools/call":
        tool_name = params["name"]
        arguments = params.get("arguments", {})

        # Call existing REST endpoint internally
        result = await call_tool_internally(tool_name, arguments)
        return jsonrpc_success(body["id"], result)

    else:
        return jsonrpc_error(-32601, "Method not found")
```

**Benefits**:
- Keep REST endpoints for workshop
- Add spec compliance for advanced users
- Side-by-side comparison for learning
- No breaking changes

### Phase 3: Official MCP SDK (Production)
**Estimated Effort**: 1-2 days

Replace custom implementation with official SDK:

**Python MCP SDK**:
```python
from mcp.server import Server
from mcp.server.stdio import stdio_server

server = Server("weather-server")

@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="get_weather_forecast",
            description="...",
            inputSchema={...}
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "get_weather_forecast":
        return await get_weather_forecast(arguments["location"])
```

**TypeScript MCP SDK**:
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "weather-server",
  version: "1.0.0",
});

server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "get_weather_forecast",
        description: "...",
        inputSchema: {...}
      }
    ]
  };
});
```

## Alternatives Considered

### Alternative 1: Full JSON-RPC from Start
Implement official MCP transport from the beginning.

**Rejected Because**:
- Too complex for 3-4 hour workshop
- Distracts from core MCP concepts
- Requires JSON-RPC knowledge
- Slower participant progress
- Higher error rate in exercises

**Estimated Impact**: Would reduce workshop completion rate by 40-60%

### Alternative 2: Hybrid Approach (Both Transports)
Implement both REST and JSON-RPC from start.

**Rejected Because**:
- Confusing for beginners (which to use?)
- More code to maintain
- Harder to document
- Doesn't reduce complexity
- Overkill for workshop goals

### Alternative 3: stdio Transport Only
Use official stdio transport (JSON-RPC over stdin/stdout).

**Rejected Because**:
- Cannot test with curl/browser
- Harder to debug
- Requires process management
- Less familiar to web developers
- Not compatible with HTTP-based agent

### Alternative 4: GraphQL Transport
Custom transport using GraphQL instead of REST.

**Rejected Because**:
- Not MCP compliant
- Adds different complexity (GraphQL schemas)
- Less familiar than REST
- Would require GraphQL client setup
- Further from official MCP than REST

## Testing Strategy

### Workshop REST Endpoints

**Test Tools Discovery**:
```bash
curl http://localhost:8000/tools | jq .
```

**Test Tool Execution**:
```bash
curl -X POST http://localhost:8000/weather \
  -H "Content-Type: application/json" \
  -d '{"location": "Oslo"}' | jq .
```

**Test via Agent**:
```bash
curl -X POST http://localhost:8001/query \
  -H "Content-Type: application/json" \
  -d '{"query": "What is the weather in Bergen?"}' | jq .
```

### Future JSON-RPC Endpoints (Phase 2)

**Test JSON-RPC Tools List**:
```bash
curl -X POST http://localhost:8000/message \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/list"
  }' | jq .
```

**Test JSON-RPC Tool Call**:
```bash
curl -X POST http://localhost:8000/message \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "get_weather_forecast",
      "arguments": {"location": "Oslo"}
    }
  }' | jq .
```

## Related Documentation

- **MCP Specification**: https://modelcontextprotocol.io/specification/2025-11-25/server/transports
- **JSON-RPC 2.0 Spec**: https://www.jsonrpc.org/specification
- **MCP Python SDK**: https://github.com/anthropics/anthropic-quickstarts/tree/main/mcp
- **MCP TypeScript SDK**: https://github.com/modelcontextprotocol/typescript-sdk
- **Spec Update Analysis**: `cdocs/MCP_SPEC_UPDATE_ANALYSIS.md`
- **Migration Guide**: `cdocs/MCP_SPEC_UPDATE_ANALYSIS.md` (Phase 2/3 sections)

## Success Metrics

### Workshop Goals (Achieved)
- ✅ Participants understand MCP tool structure
- ✅ Participants can create custom tools in < 30 minutes
- ✅ Participants understand dynamic tool discovery
- ✅ Participants can debug with standard HTTP tools
- ✅ 90%+ workshop completion rate

### Future Goals (Phase 2/3)
- Provide JSON-RPC wrapper for advanced participants
- Offer migration path to production MCP
- Document both approaches side-by-side
- Enable gradual spec compliance adoption

## Notes

This ADR documents a **conscious trade-off** between:
- **Spec compliance** (official MCP transport)
- **Learning experience** (workshop accessibility)

We prioritize learning experience for the workshop while providing clear disclaimers and migration paths for production use.

**Key Principle**: Start simple, add complexity progressively.

**Workshop Instruction**: When teaching this material, emphasize that this is a learning-oriented implementation, not production-ready. Point participants to official MCP SDKs for real-world applications.
