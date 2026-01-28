# LAB02 - Oppgave 2: Random Fact Tool

## JSON-RPC 2.0 Tool Manifest

### Verktøy Definisjon: get_random_fact

```json
{
  "name": "get_random_fact",
  "title": "Tilfeldig Faktum Leverandør",
  "description": "Få et tilfeldig interessant faktum basert på kategori",
  "inputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "category": {
        "type": "string",
        "description": "Faktakategori (general, space)",
        "enum": ["general", "space"],
        "default": "general"
      }
    },
    "required": ["category"],
    "additionalProperties": false
  },
  "outputSchema": {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
      "category": {
        "type": "string"
      },
      "fact": {
        "type": "string"
      },
      "timestamp": {
        "type": "string"
      }
    }
  }
}
```

## Implementasjonsguide

### 1. Verktøy Funksjon (i mcp-server/app.py)

```python
async def get_random_fact(category: str = "general") -> Dict[str, Any]:
    """Få et tilfeldig interessant faktum."""
    try:
        facts = {
            "general": [
                "Honningbien produserer mat spist av mennesker.",
                "Bananer er bær, men jordbær er ikke det."
            ],
            "space": [
                "En dag på Venus er lengre enn året sitt.",
                "Saturn ville flyte i vann."
            ]
        }
        
        import random
        fact = random.choice(facts.get(category, facts["general"]))
        
        result = {
            "category": category,
            "fact": fact,
            "timestamp": datetime.now().isoformat()
        }
        
        return result
        
    except Exception as e:
        logger.error(f"Faktum henting feil: {e}")
        return {"error": f"Kunne ikke hente faktum: {str(e)}"}
```

### 2. Legg til i Verktøylisten (i handle_tools_list())

Legg til dette i `tools` arrayen i `handle_tools_list()` funksjonen:

```python
{
    "name": "get_random_fact",
    "title": "Tilfeldig Faktum Leverandør",
    "description": "Få et tilfeldig interessant faktum basert på kategori",
    "inputSchema": {
        "$schema": "https://json-schema.org/draft/2020-12/schema",
        "type": "object",
        "properties": {
            "category": {
                "type": "string",
                "description": "Faktakategori (general, space)",
                "enum": ["general", "space"],
                "default": "general"
            }
        },
        "required": ["category"],
        "additionalProperties": False
    },
    "outputSchema": {
        "$schema": "https://json-schema.org/draft/2020-12/schema",
        "type": "object",
        "properties": {
            "category": {"type": "string"},
            "fact": {"type": "string"},
            "timestamp": {"type": "string"}
        }
    }
}
```

### 3. Legg til Verktøy Ruting (i handle_tools_call())

Legg til denne elif grenen:

```python
elif tool_name == "get_random_fact":
    category = arguments.get("category", "general")
    
    result = await get_random_fact(category)
    
    if "error" in result:
        return {
            "content": [{"type": "text", "text": json.dumps(result, ensure_ascii=False)}],
            "isError": True
        }
    
    return {
        "content": [{"type": "text", "text": json.dumps(result, ensure_ascii=False, indent=2)}],
        "structuredContent": result,
        "isError": False
    }
```

## Testing

### Test med JSON-RPC 2.0

```bash
# List tilgjengelige verktøy
curl -X POST "http://localhost:8000/message" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/list"
  }'

# Kall get_random_fact med space kategori
curl -X POST "http://localhost:8000/message" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "get_random_fact",
      "arguments": {"category": "space"}
    }
  }'

# Kall med general kategori (standard)
curl -X POST "http://localhost:8000/message" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "get_random_fact",
      "arguments": {"category": "general"}
    }
  }'
```

## Samsvar Sjekkliste

- ✅ Følger JSON-RPC 2.0 protokoll
- ✅ Bruker MCP 2025-11-25 kompatibel format
- ✅ Implementerer inputSchema med JSON Schema 2020-12
- ✅ Implementerer outputSchema
- ✅ Håndterer feil korrekt med error dict
- ✅ Returnerer MCP-kompatibel respons (content, structuredContent, isError)
- ✅ Integrert med handle_tools_call() ruting
- ✅ Registrert i tools/list manifest
