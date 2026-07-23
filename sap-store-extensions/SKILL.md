---
name: sap-store-extensions
description: Show, list, or find SAP Store extensions, products, or solutions for a specific partner/publisher or industry. Supports an optional Works With filter (default: SAP S/4HANA Cloud Public Edition). Queries MXP Production and renders results as a markdown table in the chat.
context: fork
user-invocable: true
---

Query MXP Production for SAP Store extensions by publisher/partner name or industry, optionally filtered by "Works With" product, and render results as a markdown table in the chat.

## Usage

```
/sap-store-extensions <publisher name or industry> [works with: <product>]
```

Examples:
- `/sap-store-extensions <partner name>` — publisher search, default Works With filter (SAP S/4HANA Cloud Public Edition)
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

4. **Derive industry tags** from `product.industry[].name` (returned in step 3). Join multiple values with `, `.

5. **Output a markdown table** in the chat with these columns:

   | # | Product | Partner | Type | Industry | SAP Store |
   |---|---------|---------|------|----------|-----------|

   - **Product**: product name
   - **Partner**: publisher name
   - **Type**: solution type (shorten "Extensions and Add-ons" → `Extension`, "APIs & Technical Components" → `API`)
   - **Industry**: comma-separated industry names from step 4
   - **SAP Store**: `[↗ Open](storefront_url)`

6. **After the table**, output a one-line summary:
   `> X products from Y partners` — append ` · Works With: <value>` if filter is active.
