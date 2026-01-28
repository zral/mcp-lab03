# ADR 002: MCP Response Format

**Status**: Accepted
**Date**: 2025-01-24
**Deciders**: Workshop Architecture Team
**Related**: [ADR 001: Dynamic Tool Discovery](001-dynamic-tool-discovery.md)

## Context

MCP tools must return structured responses that:
1. Are human-readable for end users
2. Can be parsed programmatically by AI agents
3. Clearly indicate success or failure
4. Follow the Model Context Protocol specification (2025-11-25)

The workshop needs a response format that is:
- Simple enough for beginners to understand
- Compliant with MCP specification
- Flexible enough for different tool types
- Easy to parse by OpenAI function calling

## Decision

We adopt the MCP-compliant response format with three required fields:

```json
{
  "content": [
    {
      "type": "text",
      "text": "Human-readable result text"
    }
  ],
  "structuredContent": {
    "key": "programmatically parseable data"
  },
  "isError": false
}
```

### Field Definitions

#### 1. `content` (Required - MCP Spec)
- **Type**: Array of content items
- **Purpose**: Human-readable output for display
- **MCP Spec**: Standard field from MCP specification
- **Workshop Usage**: Contains formatted text results

**Example**:
```json
"content": [
  {
    "type": "text",
    "text": "Temperature in Oslo: 5°C, Cloudy"
  }
]
```

**Future Extensions** (not implemented in workshop):
- `{"type": "image", "data": "base64..."}` - Images
- `{"type": "resource", "uri": "..."}` - File references

#### 2. `structuredContent` (Extension)
- **Type**: Object
- **Purpose**: Structured data for programmatic parsing
- **MCP Spec**: Our extension (not in base spec)
- **Workshop Usage**: Raw JSON data for AI agent to process

**Example**:
```json
"structuredContent": {
  "temperature": 5,
  "conditions": "Cloudy",
  "humidity": 78,
  "wind_speed": 12
}
```

**Rationale**:
- OpenAI can better understand structured data than parsing text
- Easier for workshop participants to extract specific values
- Separates presentation (`content`) from data (`structuredContent`)

#### 3. `isError` (Required - MCP Spec)
- **Type**: Boolean
- **Purpose**: Indicate success or failure
- **MCP Spec**: Standard field from MCP specification
- **Workshop Usage**: Agent checks this FIRST before processing

**Success Response**:
```json
{
  "content": [{"type": "text", "text": "Result data"}],
  "structuredContent": {...},
  "isError": false
}
```

**Error Response**:
```json
{
  "content": [{"type": "text", "text": "Error: Location not found"}],
  "isError": true
}
```

### Error Handling Strategy

Two types of errors are handled differently:

**1. Business Logic Errors** (Expected Failures):
- Location not found
- Invalid API key
- Rate limit exceeded

**Pattern**:
```python
if "error" in result:
    return {
        "content": [{"type": "text", "text": result["error"]}],
        "isError": True
    }
```

**2. Infrastructure Errors** (Unexpected Failures):
- Network timeout
- Server crash
- Malformed JSON

**Pattern**:
```python
except Exception as e:
    return {
        "content": [{"type": "text", "text": f"Error: {str(e)}"}],
        "isError": True
    }
```

## Consequences

### Positive

1. **MCP Compliance**: Follows MCP specification for `content` and `isError`
2. **Clear Error Handling**: Agent can check `isError` flag before parsing
3. **Dual Format**: Both human-readable (`content`) and structured (`structuredContent`)
4. **OpenAI Optimization**: Structured data improves AI understanding
5. **Educational Value**: Simple pattern for workshop participants to replicate
6. **Extensibility**: Can add more content types (images, resources) later

### Negative

1. **Duplication**: Same data appears in both `content` and `structuredContent`
2. **Extension Field**: `structuredContent` is not in base MCP spec (workshop-specific)
3. **Manual Consistency**: Developers must ensure `content` matches `structuredContent`

### Neutral

1. **Array Requirement**: `content` is always an array even with single item (MCP spec requirement)
2. **JSON Serialization**: Agent must parse `structuredContent` as JSON
3. **Error Messages**: Only appear in `content`, no error details in `structuredContent`

## Implementation

### MCP Server Side

**File**: `services/mcp-server/app.py`

**Success Response** (lines 569-581):
```python
return {
    "content": [
        {
            "type": "text",
            "text": json.dumps(result, ensure_ascii=False, indent=2)
        }
    ],
    "structuredContent": result,
    "isError": False
}
```

**Error Response** (lines 558-566):
```python
if "error" in result:
    return {
        "content": [
            {
                "type": "text",
                "text": json.dumps(result, ensure_ascii=False)
            }
        ],
        "isError": True
    }
```

### Agent Side

**File**: `services/agent/app.py`

**Response Parsing** (lines 250-262):
```python
# STEG 3: Sjekk om verktøyet returnerte en feil
if result.get("isError"):
    # MCP verktøy indikerte en feil
    tool_response = result["content"][0]["text"]
else:
    # Suksess: bruk structuredContent hvis tilgjengelig
    if "structuredContent" in result:
        tool_response = json.dumps(result["structuredContent"])
    else:
        # Fallback til content hvis ingen structuredContent
        tool_response = result["content"][0]["text"]
```

## Testing Pattern

**Test Success Response**:
```bash
curl -X POST http://localhost:8000/weather \
  -H "Content-Type: application/json" \
  -d '{"location": "Oslo"}'

# Expected:
# {
#   "content": [{"type": "text", "text": "{...}"}],
#   "structuredContent": {"temperature": 5, ...},
#   "isError": false
# }
```

**Test Error Response**:
```bash
curl -X POST http://localhost:8000/weather \
  -H "Content-Type: application/json" \
  -d '{"location": "NonexistentCity12345"}'

# Expected:
# {
#   "content": [{"type": "text", "text": "Error: ..."}],
#   "isError": true
# }
```

## Alternatives Considered

### Alternative 1: Simple String Response
```python
return "Temperature in Oslo: 5°C"
```

**Rejected Because**:
- Not MCP compliant
- No error indication mechanism
- Hard to parse programmatically
- No structured data for AI

### Alternative 2: OpenAI-Only Format
```python
return {
    "role": "function",
    "content": json.dumps(result)
}
```

**Rejected Because**:
- OpenAI-specific, not MCP compliant
- No human-readable format
- Tight coupling to OpenAI API

### Alternative 3: MCP Spec Only (No Extensions)
```python
return {
    "content": [{"type": "text", "text": json.dumps(result)}],
    "isError": false
}
```

**Rejected Because**:
- AI must parse JSON from text string
- Less efficient for OpenAI function calling
- Harder for workshop participants to work with structured data

## Migration Path

### Current State (Workshop)
- Uses MCP response format with `structuredContent` extension
- Simple HTTP REST transport
- Works well for educational purposes

### Next Step (Spec Compliance)
- Keep response format (it's compatible)
- Add JSON-RPC 2.0 wrapper layer
- Maintain `structuredContent` as custom extension

### Production
- Consider official MCP SDKs (Python, TypeScript)
- May need to remove `structuredContent` if strict compliance required
- Can keep as custom extension if documented

## Related Documentation

- **MCP Specification**: https://modelcontextprotocol.io/specification/2025-11-25/server/tools
- **Implementation Examples**: `cdocs/KOMMENTARER_NORSK.md`
- **Workshop Guide**: `doc/WORKSHOP.md`
- **Spec Update Analysis**: `cdocs/MCP_SPEC_UPDATE_ANALYSIS.md`

## Notes

This ADR documents the response format used throughout the workshop. All new tools MUST follow this pattern for consistency and MCP compliance.

**Workshop Instruction**: When creating custom tools, always copy the response format pattern from `services/mcp-server/app.py:569-581` (success) and `:558-566` (error).
