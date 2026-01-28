# ADR 001: Dynamic Tool Discovery via HTTP Manifest

**Status**: Accepted
**Date**: 2025-01-24
**Deciders**: Workshop Architecture Team
**Related**: [ADR 002: MCP Response Format](002-mcp-response-format.md), [ADR 003: HTTP Transport Choice](003-http-transport-choice.md)

## Context

AI agents traditionally hardcode their available functions and capabilities directly in the agent code. This creates tight coupling between the agent implementation and the tools it can use.

### Problems with Hardcoded Tools

1. **Tight Coupling**: Agent code must be modified every time a new tool is added
2. **Workshop Friction**: Participants must understand agent internals to add features
3. **Testing Difficulty**: Cannot easily test agent with different tool configurations
4. **Deployment Inflexibility**: Cannot change available tools without redeploying agent
5. **Code Duplication**: Tool definitions duplicated across multiple agents

### Example of Hardcoded Approach (What We Avoided)

```python
# BAD: Hardcoded tools in agent
class Agent:
    def __init__(self):
        self.tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_weather_forecast",
                    "description": "Get weather forecast",
                    # ... hardcoded schema
                }
            }
        ]

    async def call_tool(self, tool_name, args):
        if tool_name == "get_weather_forecast":
            return await self._call_weather_api(args)
        # Must add elif for each new tool!
```

## Decision

Implement **dynamic tool discovery** where the agent fetches a tools manifest from the MCP server at startup via `GET /tools` endpoint.

### Architecture Pattern

```
Agent Startup:
1. GET /tools from MCP Server
2. Parse tools manifest JSON
3. Convert MCP format to OpenAI function format
4. Store endpoint mappings
5. Agent ready with all available tools

User Query:
1. OpenAI determines tool is needed
2. Agent looks up endpoint from stored mappings
3. HTTP call to MCP server
4. Return result to OpenAI
```

### MCP Server Responsibility

```python
@app.get("/tools")
async def list_tools():
    return {
        "tools": [
            {
                "name": "get_weather_forecast",
                "description": "Get weather forecast",
                "inputSchema": {...},
                "endpoint": "/weather",  # Explicit routing
                "method": "POST"
            }
        ]
    }
```

### Agent Responsibility

```python
async def load_tools_from_mcp_server(self):
    response = await http_client.get(f"{mcp_server_url}/tools")
    tools_manifest = response.json()

    for tool in tools_manifest["tools"]:
        # Convert to OpenAI format
        self.tools.append({...})

        # Store endpoint mapping
        self.tool_endpoints[tool["name"]] = {
            "endpoint": tool["endpoint"],
            "method": tool["method"]
        }
```

### Extensions to MCP Specification

We added two fields not in the official MCP spec:

- `endpoint`: HTTP endpoint path (e.g., "/weather")
- `method`: HTTP method (e.g., "POST", "GET")

**Rationale**: The MCP spec doesn't mandate HTTP as a transport, so it doesn't specify routing. These fields make HTTP-based tool routing explicit rather than convention-based.

## Consequences

### Positive

1. **Zero Code Changes for New Tools**
   - Add tool to MCP server
   - Add to tools manifest
   - Restart services
   - Agent automatically discovers and uses it

2. **Workshop-Friendly**
   - Participants only need to understand MCP server
   - No need to modify agent code
   - Faster iteration on new features

3. **Loose Coupling**
   - Agent and MCP server evolve independently
   - Can swap MCP servers without agent changes
   - Multiple agents can use same MCP server

4. **Easy Testing**
   - Mock `/tools` endpoint with different manifests
   - Test agent with various tool configurations
   - No need to modify agent code for tests

5. **Plugin Architecture**
   - MCP server acts as plugin system
   - Tools can be enabled/disabled dynamically
   - Future: Load tools from multiple MCP servers

### Negative

1. **Runtime Discovery**
   - Tools not known until runtime
   - No static type checking for tool calls
   - Cannot validate tool existence at compile time

2. **Startup Dependency**
   - Agent depends on MCP server at startup
   - If `/tools` fails, agent runs without tools
   - Need health checks and retry logic

3. **Potential Runtime Errors**
   - Tool manifest could change while agent is running
   - Endpoint mappings could become stale
   - Need graceful error handling

4. **Additional Complexity**
   - More moving parts (manifest endpoint, conversion logic)
   - Debugging requires understanding manifest format
   - Documentation must explain discovery process

### Mitigations

For negative consequences:

1. **Comprehensive Logging**: Log tool discovery process verbosely
2. **Fallback Behavior**: Agent continues without tools if discovery fails
3. **Health Checks**: Verify MCP server availability before starting agent
4. **Error Messages**: Clear messages when tools fail to load
5. **Documentation**: Extensive comments in code explaining discovery

## Implementation

### Key Files

- **Agent tool loading**: `services/agent/app.py:59-168`
- **Agent tool execution**: `services/agent/app.py:170-282`
- **MCP tools manifest**: `services/mcp-server/app.py:192-404`
- **MCP tool implementation**: `services/mcp-server/app.py:406-495`

### Testing Strategy

```bash
# Test tools manifest endpoint
curl http://localhost:8000/tools

# Test agent discovers tools
docker compose logs travel-agent | grep "Lastet.*tools"

# Test adding new tool
# 1. Add tool to MCP server
# 2. Restart: docker compose restart mcp-server travel-agent
# 3. Verify agent loaded it
```

## Alternatives Considered

### Alternative 1: Configuration File

**Approach**: Store tool definitions in YAML/JSON config file

**Rejected because**:
- Still requires deploying config to agent
- Doesn't enable runtime extensibility
- Config duplication between agent and MCP server

### Alternative 2: Convention-Based Routing

**Approach**: Map tool names to endpoints by convention (e.g., `get_weather_forecast` → `/weather-forecast`)

**Rejected because**:
- Implicit mapping is error-prone
- Difficult to document and understand
- Doesn't support custom routing patterns
- No way to specify HTTP method

We do implement this as a **fallback** if MCP server doesn't provide explicit endpoints, but it's not the primary approach.

### Alternative 3: gRPC Service Discovery

**Approach**: Use gRPC reflection or service registry

**Rejected because**:
- Adds complexity (protobuf, gRPC infrastructure)
- HTTP/JSON is simpler for workshop context
- MCP spec already uses JSON schemas

## Related Documentation

- **ADR 002**: [MCP Response Format](002-mcp-response-format.md) - How tools return structured data
- **ADR 003**: [HTTP Transport Choice](003-http-transport-choice.md) - Why simplified REST instead of JSON-RPC
- **MCP Specification**: https://modelcontextprotocol.io/specification/2025-11-25/server/tools
- **OpenAI Function Calling**: https://platform.openai.com/docs/guides/function-calling
- **Workshop Guide**: `doc/WORKSHOP.md`
- **Norwegian Comments**: `cdocs/KOMMENTARER_NORSK.md`
- **Spec Analysis**: `cdocs/MCP_SPEC_UPDATE_ANALYSIS.md`

## Implementation Files

- **Agent tool loading**: `services/agent/app.py:59-168` (load_tools_from_mcp_server)
- **Agent tool execution**: `services/agent/app.py:170-282` (call_mcp_tool)
- **MCP tools manifest**: `services/mcp-server/app.py:192-404` (GET /tools)
- **MCP tool implementation**: `services/mcp-server/app.py:421-600` (POST /weather)

## Revision History

- **2025-01-24**: Initial decision (workshop LAB02)
- **2026-01-28**: Updated to LAB03
- **2025-01-24**: Updated spec reference to 2025-11-25, added related ADRs, improved documentation links
