# MCP Travel Weather Server

Dette er en Model Context Protocol (MCP) server som kombinerer reisedata fra Google med værdata for å hjelpe med reiseplanlegging basert på værutsikter på destinasjonen.

## Prosjektstruktur
- MCP server implementasjon i Python
- Agent for reiseplanlegging 
- Docker containers for enkel deployment
- Integrasjon med Google Travel og værtjenester

## Instruksjoner
- Bruk Python for alle implementasjoner
- Følg MCP standarder for server implementasjon
- Bruk Docker for containerisering
- Implementer proper error handling og logging
- Løsningen skal ikke under noen omstendigheter kjøres på localhost, kun i docker, docker compose eller kubernetes
- Dokumenter alle endringer og tillegg i README.md
- Ingen hardkoding av endpoint URLer i agenten
