# SAP HANA MCP Server

**English** · [Deutsch](docs/README.de.md)

**Connect SAP HANA to Claude, ChatGPT and Copilot: SAP's data dictionary (table and field labels, code values, join paths, CDS views), the organization structure and read-only SQL as MCP tools.** Powered by [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

SAP HANA MCP Server gives Claude, ChatGPT, Copilot and Cursor 10 tools for SAP HANA: SAP's data dictionary (table and field labels, code values, join paths, CDS views), the organization structure and read-only SQL. Every tool only reads. It runs on AnythingMCP: one click on AnythingMCP Cloud, or self-hosted with Docker. Credentials are stored encrypted and every call is audited.

**Last verified:** 2026-09-27 against an SAP S/4HANA 2025 Private Cloud system, a QAS copy of production (every tool run live through the adapter's live test suite).  
**Adapter synced:** <!-- synced -->2026-09-27

Maintained by [helpcode.ai](https://helpcode.ai), the team that builds and maintains [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

## Quick start (AnythingMCP Cloud)

1. Sign in at [cloud.anythingmcp.com](https://cloud.anythingmcp.com) and open the [install link](https://cloud.anythingmcp.com/connectors/store?install=sap-s4hana-hana).
2. Enter `SAP_HANA_HOST`, `SAP_HANA_PORT`, `SAP_HANA_TENANT`, `SAP_HANA_SCHEMA`, `SAP_CLIENT`, `SAP_LANGUAGE`, `SAP_HANA_TLS`, `SAP_HANA_USER`, `SAP_HANA_PASSWORD` (see [Authentication](#authentication)).
3. Copy the URL of your MCP server under **MCP Servers** and add it to your AI client ([below](#connect-claude-chatgpt-copilot-or-cursor)).

AnythingMCP Cloud is the same open-source code, operated by helpcode.ai in Frankfurt, Germany.

## Self-hosted (Docker)

Needs Docker 24+, openssl and Node 18+.

```bash
git clone https://github.com/HelpCode-ai/sap-hana-mcp-server.git
cd sap-hana-mcp-server
./scripts/install.sh
```

`install.sh` writes `.env` with fresh secrets, starts AnythingMCP, creates the first admin, installs the connector if `SAP_HANA_HOST` and `SAP_HANA_PORT` and `SAP_HANA_TENANT` and `SAP_HANA_SCHEMA` and `SAP_CLIENT` and `SAP_LANGUAGE` and `SAP_HANA_TLS` and `SAP_HANA_USER` and `SAP_HANA_PASSWORD` are set in `.env` and creates an MCP API key. Without credentials it prints the install link instead: `http://localhost:3000/connectors/store?install=sap-s4hana-hana`. Then check the whole chain:

```bash
npm install && node scripts/smoke.mjs
```

## Connect Claude, ChatGPT, Copilot or Cursor

- **Claude (claude.ai, Desktop, mobile):** *Customize → Connectors → Add custom connector*, paste your MCP server URL and sign in. Claude connects from Anthropic's cloud, so the URL must be public HTTPS: your AnythingMCP Cloud URL, or your own instance behind TLS.
- **Claude Code:**

  ```bash
  claude mcp add --transport http sap-hana-mcp-server http://localhost:4000/mcp --header "X-API-Key: <MCP_API_KEY>"
  ```
- **Cursor** (`.cursor/mcp.json`) and **VS Code / GitHub Copilot** (`.vscode/mcp.json`, key `servers` instead of `mcpServers`, plus `"type": "http"`):

  ```json
  { "mcpServers": { "sap-hana-mcp-server": { "url": "http://localhost:4000/mcp", "headers": { "X-API-Key": "<MCP_API_KEY>" } } } }
  ```
- **ChatGPT:** add the public HTTPS URL as a connector (app) in ChatGPT's settings. A `localhost` URL does not work there.

## Tools

10 tools, generated from [`adapter/sap-s4hana-hana.json`](adapter/sap-s4hana-hana.json). **read** tools cannot change anything in the source system.

<!-- tools:start (generated from adapter/*.json, do not edit) -->
| Tool | What it does | Access |
|---|---|---|
| `sap_guide` | The SAP data model explained for SQL: how to work (overview), client and data types (basics), finance, sales, inventory, procurement, operations, pitfalls… | read |
| `sap_org_structure` | Company codes (with currency, chart of accounts and fiscal year variant), controlling areas, plants, sales organizations and purchasing organizations of… | read |
| `sap_search_tables` | Find SAP tables and database views by name pattern or by words in their description (e.g. "billing document", "VBRK", "ZSD%"). | read |
| `sap_describe_table` | Every field of an SAP table with its business label, key flag, type, the currency or unit field that goes with each amount or quantity, and the check table… | read |
| `sap_find_fields` | Find which tables hold a business field, by words in its label (e.g. "payment terms", "net due date") or by data element name. | read |
| `sap_field_values` | What the codes in a field mean: the fixed values of its domain with their descriptions (e.g. VBRK VBTYP, ACDOCA KOART). | read |
| `sap_table_relations` | How a table joins to others: its foreign keys with the check table and the join condition, and the text table that holds its descriptions. | read |
| `sap_search_cds_views` | Search SAP's CDS views (the S/4HANA virtual data model) by name or label, e.g. "journal entry", "billing", "stock". | read |
| `sap_describe_cds_view` | Columns of a CDS view with their labels, default aggregation (SUM marks a measure), currency and unit fields, text fields and foreign-key associations. | read |
| `sap_query` | Run one read-only SQL SELECT against the SAP HANA database and return up to 1000 rows. | read |
<!-- tools:end -->

## Example prompts

- Which company codes, plants and sales organizations does this SAP system have, and in which currencies?
- What was our revenue per company code in this fiscal year, from the Universal Journal, with currency?
- Which customers had the highest billed net value last quarter, per sales organization?
- Which tables hold the net due date of an open item, and how do they join to the customer master?
- Is there a released CDS view for billing documents? Show its measures and dimensions.
- What do the values of field KOART in ACDOCA mean?

More in [examples/prompts.md](examples/prompts.md).

## Authentication

**What this connects to**: the HANA database under an SAP S/4HANA system (on-premise, Private Cloud / RISE, or HANA Enterprise Cloud), read directly with SQL. SAP's table and column names are abbreviations (BUKRS, HSL, VBELN); the tools below read SAP's own dictionary so the model sees what each table and field means before it queries.

**Start with `sap_guide`**: it costs nothing and explains the SAP data model (client, dates, currencies, finance, sales, inventory) and the order in which to use the other tools.

**Setup (SAP Basis, one-off)**
1. Create a technical database user in the tenant that holds the ABAP schema (usually `SAPHANADB` or `SAPABAP1`), with a password that does not expire:
   ```sql
   CREATE USER AMCP_READER PASSWORD "<strong password>" NO FORCE_FIRST_PASSWORD_CHANGE;
   ALTER USER AMCP_READER DISABLE PASSWORD LIFETIME;
   ```
2. Grant `SELECT` on what the agent may read. The dictionary tools need `DD02L DD02T DD03L DD03T DD03ND DD04T DD07T DD08L DD05S DDHEADANNO DDFIELDANNO ARS_W_API_STATE` and `sap_org_structure` needs `T001 T001K T001W TKA01 TVKO TVKOT T024E`. Add the business tables and CDS views for your use cases, or `GRANT SELECT ON SCHEMA SAPHANADB` for broad analytics. Never grant HR tables you do not want read.
3. Recommended: cap the user's statements on the server side with a workload class (`STATEMENT TIMEOUT`, `STATEMENT MEMORY LIMIT`) mapped to the user.
4. Fill in the variables: `SAP_HANA_HOST`; `SAP_HANA_PORT`, the tenant's SQL port (3<instance>15 for the first tenant, 3<instance>41 and up for others; the system database port 3<instance>13 also works together with the tenant name); `SAP_HANA_TENANT`, the tenant database name; `SAP_HANA_SCHEMA` (e.g. `SAPHANADB`); `SAP_CLIENT`, the three-digit SAP client (e.g. `100`); `SAP_LANGUAGE`, the one-letter SAP language for texts (`E` English, `D` German); `SAP_HANA_TLS`, `verify`, `no-verify` (self-signed certificate) or `off`; `SAP_HANA_USER` and `SAP_HANA_PASSWORD`.

**Network**: the HANA SQL port is normally reachable only inside the company network. Self-host AnythingMCP there (or connect it through a VPN) and add the host to `SSRF_ALLOWED_HOSTS`.

**Read-only by design**: the session runs `SET TRANSACTION READ ONLY`, writes and locking reads are refused, results are capped at 1000 rows and each statement times out after 60 seconds. SQL bypasses SAP's application authorizations, so the database grants are the real boundary. HR and user/password tables are on the connector's denied-tables list by default (edit it in the connector settings).

**Licensing**: direct SQL access to the ABAP schema by a third-party application is governed by your SAP HANA licence. A runtime licence bundled with S/4HANA usually does not cover it; a full-use licence does. Check your contract before connecting production.

**Driver**: the bundled `hdb` driver needs nothing installed. To use SAP's native `@sap/hana-client` instead, see the SAP HANA page of the AnythingMCP docs.

## Security

- **Read or write is your choice.** All 10 tools only read. Assign the connector to an MCP server whose role whitelists only the tools you want, and the rest are invisible to that client.
- **Credentials** are encrypted with AES-256-GCM and never shown to the model.
- **Response mapping** drops or reshapes fields per tool before they reach the model, e.g. bank details or personal data.
- **Audit log:** every call is recorded with input, output, duration and status, in your own database when self-hosted.
- **SSO, RBAC and SCIM** are included in the self-hosted build.

## FAQ

### Is there an SAP HANA MCP server?
Yes, this one. It connects the HANA database under SAP S/4HANA to Claude, ChatGPT and Copilot through AnythingMCP. It brings a guide to SAP's data model, eight tools that read SAP's own data dictionary and organization structure (table and field labels, code values, join paths, released CDS views) and one that runs a `SELECT`, so the model knows what `BUKRS`, `HSL` or `VBRK` mean before it writes a query.

### HANA SQL or OData: which one should I use?
SQL reaches every table and CDS view and aggregates on the database, which suits analytics across finance, sales and inventory. It bypasses SAP's application authorizations, so the database grants are the boundary, and your HANA licence must allow third-party SQL access. OData goes through SAP Gateway, checks the technical user's authorizations on every call and only reaches published services, which is what SAP's API policy expects. The OData route is the SAP S/4HANA (OData) connector in sap-mcp-server; both can run side by side.

### Can the AI change data in SAP?
No. The session runs `SET TRANSACTION READ ONLY`, only a single `SELECT` gets through, locking reads are refused, results stop at 1000 rows and each statement times out after 60 seconds. Create a database user that can only read as well.

### What do I need to connect it?
A HANA database user with `SELECT` on the dictionary tables and the business tables you want read (the grants are under Authentication), the tenant's SQL port, the ABAP schema (usually `SAPHANADB` or `SAPABAP1`) and the SAP client. The HANA port is normally internal, so AnythingMCP runs self-hosted inside the network or reaches it through a VPN.

### Does it work on RISE with SAP and Private Cloud?
Yes; it was verified on an S/4HANA 2025 Private Cloud system. In SAP's private cloud offerings the database user and the network path are requested from SAP.

### Does my SAP licence allow this?
Direct SQL access to the ABAP schema by a third-party application is governed by your SAP HANA licence. A runtime licence bundled with S/4HANA usually does not cover it; a full-use licence does. Check with your SAP account team before you connect production.

### Can it read HR data?
HR and user/password tables are on the connector's denied-tables list by default, and statements naming them are refused. Grant the database user only what the agent needs; `GRANT SELECT ON SCHEMA` also makes HR tables readable.

### Which driver does it use?
SAP's pure-JavaScript `hdb` driver, bundled, with TLS and multi-tenant systems. SAP's native `@sap/hana-client` (Kerberos, `hdbuserstore`) can be installed into your own deployment; AnythingMCP may not redistribute it. A HANA database that is not an S/4HANA system connects as a plain database connector with a `hana://` connection string.

## Troubleshooting

| Problem | Fix |
|---|---|
| `401` / `403` from the vendor | The credentials are wrong or lack rights. Re-enter them on the connector page; the import runs a test call and shows the result. |
| Tools missing in the AI client | The connector is not assigned to the MCP server the client uses. Check **MCP Servers**, then run `node scripts/smoke.mjs`. |
| The host is on your internal network | Self-host AnythingMCP on that network and add the hostname to `SSRF_ALLOWED_HOSTS`, or the outbound guard blocks the call. |
| Works locally, fails on AnythingMCP Cloud | The system must be reachable from the internet with a valid TLS certificate. |

## Related

- [sap-mcp-server](https://github.com/HelpCode-ai/sap-mcp-server): SAP MCP server: connect SAP Business One, S/4HANA (Cloud, on-premise via OData or HANA SQL) and Concur to Claude & ChatGPT.
- [sql-to-mcp](https://github.com/HelpCode-ai/sql-to-mcp): SQL to MCP: connect PostgreSQL, MySQL, SQL Server, Oracle, SAP HANA or MongoDB to Claude & ChatGPT. Read-only, audited, no code.
- [odata-to-mcp](https://github.com/HelpCode-ai/odata-to-mcp): OData to MCP: turn any OData V2 or V4 service, SAP Gateway included, into MCP tools for Claude & ChatGPT. Reads $metadata, no code.
- [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp): the open-source MCP server and gateway this repository is built on.

## License

AGPL-3.0-only. The adapter definition in `adapter/` comes from AnythingMCP (AGPL-3.0).
