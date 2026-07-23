---
name: sap-store-extensions
description: Show, list, or find SAP Store extensions, products, or solutions for a specific partner/publisher or industry (e.g. "show extensions for NTT DATA", "list Finance extensions on SAP Store"). Supports an optional Works With filter (default: SAP S/4HANA Cloud Public Edition). Queries MXP Production and renders results as a styled HTML table opened in the browser.
context: fork
user-invocable: true
---

Query MXP Production for SAP Store extensions by publisher/partner name or industry, optionally filtered by "Works With" product, and render results as a styled HTML table, then open in the browser.

## Usage

```
/sap-store-extensions <publisher name or industry> [works with: <product>]
```

Examples:
- `/sap-store-extensions NTT DATA` — publisher search, default Works With filter (SAP S/4HANA Cloud Public Edition)
- `/sap-store-extensions Finance` — industry search, default Works With filter
- `/sap-store-extensions Finance works with: SAP ERP` — industry search, custom Works With filter
- `/sap-store-extensions Finance all` — industry search, no Works With filter

## Works With Filter

- **Default**: `SAP S/4HANA Cloud Public Edition` — always apply unless user says otherwise
- User can specify a different value: "works with: SAP ERP", "filter by SAP BTP"
- User can disable: "no filter", "all", "without filter", "show all products"
- Filter uses `ilike` on `works_with` field: `{"column": "works_with", "ilike": "%<value>%"}`

## Instructions

The argument `$ARGUMENTS` is either a publisher name or an industry name, optionally followed by a Works With override.

1. **Parse `$ARGUMENTS`**:
   - Extract the primary search term (publisher or industry name)
   - Extract Works With value if specified (e.g. "works with: SAP ERP")
   - If "all", "no filter", "without filter" is present → set Works With to null (no filter)
   - Otherwise default to `SAP S/4HANA Cloud Public Edition`

2. **Detect search mode**:
   - **Industry mode**: primary term is clearly an industry or line-of-business name (Finance, Retail, Utilities, HCM, etc.) → use `industry.name` filter
   - **Publisher mode**: primary term is a company/partner name → use `publisher_name` filter

3. **Query SAP Store Products** from the `mxp-mcp` server:
   - Entity ID: `af92f525-c98a-4e3e-a5a9-231cb58df63d`
   - Worksphere ID: `abd0b7ce-ccf3-4abb-895a-8e5f20c2706a`
   - **Without Works With filter** — single filter:
     - Industry: `{"column": "industry.name", "ilike": "%<term>%"}`
     - Publisher: `{"column": "publisher_name", "ilike": "%<publisher>%"}`
   - **With Works With filter** — combine with `and`:
     - Industry: `{"and": [{"column": "industry.name", "ilike": "%<term>%"}, {"column": "works_with", "ilike": "%<works_with>%"}]}`
     - Publisher: `{"and": [{"column": "publisher_name", "ilike": "%<publisher>%"}, {"column": "works_with", "ilike": "%<works_with>%"}]}`
   - Fields: `product_name`, `publisher_name`, `product_description_short`, `solution_type`, `works_with`, `storefront_url`, `industry.name`
   - Top: 100

4. **Derive industry tags per product** from `product.industry[].name` (already returned in step 3 via `industry.name` subfield selection).

5. **Generate the output filename** as `~/<slug>_extensions.html` where `<slug>` is the primary term lowercased with spaces replaced by underscores.

6. **Write the HTML file** using hand-coded CSS with SAP Fiori colors (do NOT use UI5 Web Components via CDN):

   **Filter bar** (below `<h1>`): show active Works With filter as a styled chip. If no filter, show "Showing all products".
   - Filter bar: bg `#f0f4fe`, border `1px solid #c5dcf7`, border-radius `6px`, padding `8px 12px`
   - Chip: bg `#0064d9`, color `#fff`, border-radius `12px`, padding `2px 10px`, font-size `0.8rem`

   **Columns:** `#`, `Product` (bold), `Partner` (own `<td>`, never nested), `Type` (badge), `Description`, `Industry` (green tags — one per `industry[].name` value), `Works With` (purple tags — split on commas; highlight the matching value in darker purple if filter active), `SAP Store` (↗ Open link)

   **CSS color values:**
   - Header bg: `#0064d9`, text: `#fff`, col border: `#0058c0`
   - Row hover: `#f0f4fe`
   - Badge Extensions: bg `#e3f0ff`, text `#0064d9`, border `#c5dcf7`
   - Badge APIs: bg `#fef0d9`, text `#c44300`, border `#f5d5a3`
   - Industry tag: bg `#f0faf0`, text `#256f3a`, border `#b5debb`
   - Works With tag: bg `#f5f0fd`, text `#5a2a82`, border `#d5b8f5`
   - Works With tag **highlighted**: bg `#7300cc`, text `#fff`, border `#5a008f`
   - Link button: bg `#0064d9`, text `#fff`, hover `#0053b3`
   - Page bg: `#f5f6f7`, card bg: `#fff`
   - Card: `border-radius: 8px`, `box-shadow: 0 0 0 1px #e5e5e5, 0 2px 8px rgba(0,0,0,0.08)`

   **Page title:**
   - Industry mode: `<Industry> Extensions on SAP Store`
   - Publisher mode: `<Publisher> Extensions on SAP Store`

   **Footer:** product count | partner count | Works With filter (if active) | data source | date

7. **Open the file** in the browser using `open <filepath>`.
