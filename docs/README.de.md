# SAP HANA MCP Server

[English](../README.md) · **Deutsch**

**Verbinde SAP HANA mit Claude, ChatGPT und Copilot: das SAP Data Dictionary (Tabellen- und Feldtexte, Festwerte, Join-Pfade, CDS-Views), die Organisationsstruktur und lesendes SQL als MCP-Tools.** Basiert auf [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

SAP HANA MCP Server gibt Claude, ChatGPT, Copilot und Cursor 10 Tools für SAP HANA: das SAP Data Dictionary (Tabellen- und Feldtexte, Festwerte, Join-Pfade, CDS-Views), die Organisationsstruktur und lesendes SQL. Alle Tools lesen nur. Es läuft auf AnythingMCP: mit einem Klick in AnythingMCP Cloud oder selbst gehostet mit Docker. Zugangsdaten werden verschlüsselt gespeichert, jeder Aufruf landet im Audit-Log.

**Zuletzt geprüft:** 2026-09-27 gegen ein SAP S/4HANA 2025 Private Cloud-System, eine QAS-Kopie der Produktion (jedes Tool live über die Live-Testsuite des Adapters ausgeführt).  
**Adapter synchronisiert:** <!-- synced -->2026-09-29

Maintained by [helpcode.ai](https://helpcode.ai), the team that builds and maintains [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

## Schnellstart (AnythingMCP Cloud)

1. Melde dich bei [cloud.anythingmcp.com](https://cloud.anythingmcp.com) an und öffne den [Installationslink](https://cloud.anythingmcp.com/connectors/store?install=sap-s4hana-hana).
2. Trage `SAP_HANA_HOST`, `SAP_HANA_PORT`, `SAP_HANA_TENANT`, `SAP_HANA_SCHEMA`, `SAP_CLIENT`, `SAP_LANGUAGE`, `SAP_HANA_TLS`, `SAP_HANA_USER`, `SAP_HANA_PASSWORD` ein (siehe [Authentifizierung](#authentifizierung)).
3. Kopiere die URL deines MCP-Servers unter **MCP Servers** und füge sie in deinen KI-Client ein ([siehe unten](#claude-chatgpt-copilot-oder-cursor-verbinden)).

AnythingMCP Cloud ist derselbe Open-Source-Code, betrieben von helpcode.ai in Frankfurt.

## Selbst gehostet (Docker)

Benötigt Docker 24+, openssl und Node 18+.

```bash
git clone https://github.com/HelpCode-ai/sap-hana-mcp-server.git
cd sap-hana-mcp-server
./scripts/install.sh
```

`install.sh` schreibt `.env` mit neuen Secrets, startet AnythingMCP, legt den ersten Admin an, installiert den Connector, sofern `SAP_HANA_HOST` und `SAP_HANA_PORT` und `SAP_HANA_TENANT` und `SAP_HANA_SCHEMA` und `SAP_CLIENT` und `SAP_LANGUAGE` und `SAP_HANA_TLS` und `SAP_HANA_USER` und `SAP_HANA_PASSWORD` in `.env` gesetzt sind, und erzeugt einen MCP-API-Key. Ohne Zugangsdaten gibt es stattdessen den Installationslink aus: `http://localhost:3000/connectors/store?install=sap-s4hana-hana`. Danach die ganze Kette prüfen:

```bash
npm install && node scripts/smoke.mjs
```

## Claude, ChatGPT, Copilot oder Cursor verbinden

- **Claude (claude.ai, Desktop, Mobil):** *Customize → Connectors → Add custom connector*, MCP-Server-URL einfügen und anmelden. Claude verbindet sich aus der Cloud von Anthropic, die URL muss also öffentlich per HTTPS erreichbar sein: deine AnythingMCP-Cloud-URL oder deine eigene Instanz mit TLS.
- **Claude Code:**

  ```bash
  claude mcp add --transport http sap-hana-mcp-server http://localhost:4000/mcp --header "X-API-Key: <MCP_API_KEY>"
  ```
- **Cursor** (`.cursor/mcp.json`) und **VS Code / GitHub Copilot** (`.vscode/mcp.json`, Schlüssel `servers` statt `mcpServers`, dazu `"type": "http"`):

  ```json
  { "mcpServers": { "sap-hana-mcp-server": { "url": "http://localhost:4000/mcp", "headers": { "X-API-Key": "<MCP_API_KEY>" } } } }
  ```
- **ChatGPT:** die öffentliche HTTPS-URL in den ChatGPT-Einstellungen als Connector (App) hinzufügen. Eine `localhost`-URL funktioniert dort nicht.

## Tools

10 Tools, erzeugt aus [`adapter/sap-s4hana-hana.json`](../adapter/sap-s4hana-hana.json). Tools mit **lesen** können im Quellsystem nichts ändern.

<!-- tools:start (generated from adapter/*.json, do not edit) -->
| Tool | Funktion | Zugriff |
|---|---|---|
| `sap_guide` | The SAP data model explained for SQL: how to work (overview), client and data types (basics), finance, sales, inventory, procurement, operations, pitfalls… | lesen |
| `sap_org_structure` | Company codes (with currency, chart of accounts and fiscal year variant), controlling areas, plants, sales organizations and purchasing organizations of… | lesen |
| `sap_search_tables` | Find SAP tables and database views by name pattern or by words in their description (e.g. "billing document", "VBRK", "ZSD%"). | lesen |
| `sap_describe_table` | Every field of an SAP table with its business label, key flag, type, the currency or unit field that goes with each amount or quantity, and the check table… | lesen |
| `sap_find_fields` | Find which tables hold a business field, by words in its label (e.g. "payment terms", "net due date") or by data element name. | lesen |
| `sap_field_values` | What the codes in a field mean: the fixed values of its domain with their descriptions (e.g. VBRK VBTYP, ACDOCA KOART). | lesen |
| `sap_table_relations` | How a table joins to others: its foreign keys with the check table and the join condition, and the text table that holds its descriptions. | lesen |
| `sap_search_cds_views` | Search SAP's CDS views (the S/4HANA virtual data model) by name or label, e.g. "journal entry", "billing", "stock". | lesen |
| `sap_describe_cds_view` | Columns of a CDS view with their labels, default aggregation (SUM marks a measure), currency and unit fields, text fields and foreign-key associations. | lesen |
| `sap_query` | Run one read-only SQL SELECT against the SAP HANA database and return up to 1000 rows. | lesen |
<!-- tools:end -->

## Beispiel-Prompts

- Welche Buchungskreise, Werke und Verkaufsorganisationen hat dieses SAP-System, in welchen Währungen?
- Wie hoch war der Umsatz je Buchungskreis im laufenden Geschäftsjahr laut Universal Journal, mit Währung?
- Welche Kunden hatten im letzten Quartal den höchsten fakturierten Nettowert, je Verkaufsorganisation?
- In welchen Tabellen steht die Nettofälligkeit eines offenen Postens, und wie hängen sie am Kundenstamm?
- Gibt es einen freigegebenen CDS-View für Fakturen? Zeig mir seine Kennzahlen und Merkmale.
- Was bedeuten die Werte des Felds KOART in ACDOCA?

Weitere (auf Englisch) in [examples/prompts.md](../examples/prompts.md).

## Authentifizierung

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

## Sicherheit

- **Lesen oder schreiben entscheidest du.** Alle 10 Tools lesen nur. Weise den Connector einem MCP-Server zu, dessen Rolle nur die gewünschten Tools freigibt; die anderen sieht dieser Client gar nicht.
- **Zugangsdaten** werden mit AES-256-GCM verschlüsselt und nie an das Modell gegeben.
- **Response-Mapping** entfernt oder formt Felder pro Tool, bevor sie das Modell erreichen, etwa Bankdaten oder personenbezogene Daten.
- **Audit-Log:** Jeder Aufruf wird mit Eingabe, Ausgabe, Dauer und Status protokolliert, selbst gehostet in deiner eigenen Datenbank.
- **SSO, RBAC und SCIM** sind in der selbst gehosteten Version enthalten.

## Anleitungen

- [Use cases](../docs/use-cases.md)
- [Architecture](../docs/architecture.md)
- [Security and permissions](../docs/security.md)
- [Compared with other ways to reach SAP](../docs/comparison.md)

## FAQ

### Gibt es einen MCP-Server für SAP HANA?
Ja, diesen hier. Er verbindet die HANA-Datenbank unter SAP S/4HANA über AnythingMCP mit Claude, ChatGPT und Copilot. Dazu gehören ein Leitfaden zum SAP-Datenmodell, acht Tools, die das Data Dictionary und die Organisationsstruktur von SAP lesen (Tabellen- und Feldtexte, Festwerte, Join-Pfade, freigegebene CDS-Views), und eines, das ein `SELECT` ausführt. So weiß das Modell, was `BUKRS`, `HSL` oder `VBRK` bedeuten, bevor es eine Abfrage schreibt.

### HANA SQL oder OData: was nehme ich?
SQL erreicht jede Tabelle und jeden CDS-View und aggregiert in der Datenbank, gut für Auswertungen über Finanzen, Vertrieb und Bestand. Es umgeht die Berechtigungsprüfungen der SAP-Anwendung, die Grenze sind also die Datenbankrechte, und deine HANA-Lizenz muss SQL-Zugriff durch Drittanwendungen erlauben. OData läuft über das SAP Gateway, prüft bei jedem Aufruf die Berechtigungen des technischen Benutzers und erreicht nur veröffentlichte Services, so wie es SAPs API-Richtlinie erwartet. Der OData-Weg ist der Connector SAP S/4HANA (OData) in sap-mcp-server; beide laufen auch parallel.

### Kann die KI Daten in SAP ändern?
Nein. Die Session läuft mit `SET TRANSACTION READ ONLY`, nur ein einzelnes `SELECT` kommt durch, sperrende Lesezugriffe werden abgewiesen, Ergebnisse enden bei 1000 Zeilen und jede Anweisung bricht nach 60 Sekunden ab. Lege trotzdem einen Datenbankbenutzer an, der nur lesen darf.

### Was brauche ich für die Verbindung?
Einen HANA-Datenbankbenutzer mit `SELECT` auf die Dictionary-Tabellen und die Fachtabellen, die gelesen werden sollen (die Grants stehen unter Authentifizierung), den SQL-Port des Tenants, das ABAP-Schema (meist `SAPHANADB` oder `SAPABAP1`) und den SAP-Mandanten. Der HANA-Port ist normalerweise intern, AnythingMCP läuft also selbst gehostet im Netz oder erreicht es per VPN.

### Geht das mit RISE with SAP und Private Cloud?
Ja, geprüft wurde es auf einem S/4HANA 2025 Private Cloud-System. In SAPs Private-Cloud-Angeboten beantragst du Datenbankbenutzer und Netzwerkzugang bei SAP.

### Erlaubt meine SAP-Lizenz das?
Direkter SQL-Zugriff einer Drittanwendung auf das ABAP-Schema richtet sich nach deiner SAP-HANA-Lizenz. Eine mit S/4HANA gebündelte Runtime-Lizenz deckt das meist nicht ab, eine Full-Use-Lizenz schon. Kläre das mit deinem SAP-Ansprechpartner, bevor du ein Produktivsystem anbindest.

### Kann sie HR-Daten lesen?
HR- und Benutzer-/Passworttabellen stehen standardmäßig auf der Sperrliste des Connectors; Anweisungen, die sie nennen, werden abgewiesen. Gib dem Datenbankbenutzer nur, was der Agent braucht; `GRANT SELECT ON SCHEMA` macht auch HR-Tabellen lesbar.

## Fehlerbehebung

| Problem | Lösung |
|---|---|
| `401` / `403` vom Hersteller | Zugangsdaten falsch oder ohne Rechte. Auf der Connector-Seite neu eintragen; der Import macht einen Testaufruf und zeigt das Ergebnis. |
| Tools fehlen im KI-Client | Der Connector ist nicht dem MCP-Server zugewiesen, den der Client nutzt. **MCP Servers** prüfen, dann `node scripts/smoke.mjs` ausführen. |
| Das System steht im internen Netz | AnythingMCP in diesem Netz selbst hosten und den Hostnamen in `SSRF_ALLOWED_HOSTS` eintragen, sonst blockiert der Outbound-Guard den Aufruf. |
| Lokal ok, in AnythingMCP Cloud nicht | Das System muss aus dem Internet mit gültigem TLS-Zertifikat erreichbar sein. |

## Verwandte Repositories

- [sap-mcp-server](https://github.com/HelpCode-ai/sap-mcp-server): SAP MCP server: connect SAP Business One, S/4HANA (Cloud, on-premise via OData or HANA SQL) and Concur to Claude & ChatGPT.
- [sql-to-mcp](https://github.com/HelpCode-ai/sql-to-mcp): SQL to MCP: connect PostgreSQL, MySQL, SQL Server, Oracle, SAP HANA or MongoDB to Claude & ChatGPT. Read-only, audited, no code.
- [odata-to-mcp](https://github.com/HelpCode-ai/odata-to-mcp): OData to MCP: turn any OData V2 or V4 service, SAP Gateway included, into MCP tools for Claude & ChatGPT. Reads $metadata, no code.
- [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp): der Open-Source-MCP-Server und -Gateway, auf dem dieses Repository aufbaut.

## Lizenz

AGPL-3.0-only. Die Adapter-Definition in `adapter/` stammt aus AnythingMCP (AGPL-3.0).
