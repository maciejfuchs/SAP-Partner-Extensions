# Claude Skills — SAP Store Extensions

A collection of Claude Code skills for querying and visualizing SAP Store product data.

---

## sap-store-extensions

Search and browse SAP Store extensions by **publisher/partner name** or **industry**, with an optional "Works With" product filter. 

### What it does

- Searches the SAP Store product catalog via the MXP Production API
- Supports two search modes:
  - **Publisher mode** — find all products from a specific partner
  - **Industry mode** — find all products tagged for a given industry (e.g. `Retail`, `Utilities`, `Finance`)
- Applies a "Works With" filter (default: `SAP S/4HANA Cloud Public Edition`) to scope results to a specific SAP platform

### Usage

```
/sap-store-extensions <publisher or industry> [works with: <product>]
```

**Examples:**
```
/sap-store-extensions <Publisher Name>                           # publisher, default Works With filter
/sap-store-extensions <Industry Name>                            # industry, default Works With filter
/sap-store-extensions <Industry Name> with: SAP ERP             # industry, custom Works With filter
/sap-store-extensions <Publisher Name> all                      # publisher, no Works With filter
```

### Output

A self-contained HTML file saved to `~/Output/<search-term>_extensions.html` and opened in your default browser.

---

## Prerequisites

### 1. MXP MCP Server

This skill queries SAP's internal MXP (MXPresso) content platform via an MCP server. You need access to the MXP MCP server configured in your Claude Code settings.

**Server name:** `mxp-mcp`

**Configuration** — add to your `~/.claude/claude_desktop_config.json` or Claude Code MCP settings:

```json
{
  "mcpServers": {
    "mxp-mcp": {
      "command": "<path-to-mxp-mcp-server>",
      "args": []
    }
  }
}
```

> Contact your SAP internal tooling team for the MXP MCP server binary and access credentials. The server connects to MXP Production at:
> `https://acms-gateway.cfapps.eu10-004.hana.ondemand.com`

**Worksphere used:** `abd0b7ce-ccf3-4abb-895a-8e5f20c2706a` (SAP Store product catalog)

### 2. Skill installation

Install with the SAP Joule Desktop

---

## Tested with

- Claude Code (claude-sonnet-latest)
- MXP MCP server (Production)
- macOS (browser open via `open` command)

---

## License

Copyright 2026 SAP SE or an SAP affiliate company and automation-pilot-agent-skills contributors. See LICENSE for details. Third-party component information is available via the REUSE tool.

## Author

Maciej Fuchs
