# xmlvalid MCP server

<img src="logo.png" alt="xmlvalid" width="96" align="right">

The [xmlvalid](https://xmlvalid.com) MCP server lets an AI agent validate XML documents and e-invoices and explain what fails: XSD, DTD and RelaxNG schemas, and the official Schematron business rules of EN 16931, Peppol BIS Billing 3.0, XRechnung, ZUGFeRD / Factur-X, France CTC, KSeF, FatturaPA, Peppol PINT and other national formats (178 schemas, 56 rule sets with 7,751 rules).

It is a hosted server: there is nothing to install or run. Point your client at

```
https://xmlvalid.com/mcp
```

(Streamable HTTP). Each finding comes back with its line and column, the business rule's id (such as `BR-CO-15`) and a link to a page that explains how to fix it.

[Tools](#tools) · [Cline](#cline) · [Other clients](#other-clients) · [Authentication](#authentication) · [Example prompts](#example-prompts) · [Privacy and limits](#privacy-and-limits)

## Tools

| Tool | What it does | Account |
| --- | --- | --- |
| `validate_xml` | Validates a document, given as its text (`xml`) or a public URL (`url`), against a catalog schema (`schema`: a slug from `search_schemas`), one of your saved schemas (`@name`) or `auto` (the e-invoice profile the document names in its specification identifier, BT-24). Without a schema it checks well-formedness. | Needed, counts as one API validation |
| `get_validation` | The result of a validation that was still running when `validate_xml` returned. | Needed |
| `search_schemas` | Finds the catalog slug for a format: "peppol invoice", "xrechnung cii", "pain.001". | Not needed |
| `explain_error` | Explains a parser or validator message from libxml2 (xmllint, PHP, Python lxml), Java (Xerces `cvc-*`, SAX) or .NET (System.Xml): what it means, why it happens, how to fix it, with an example. | Not needed |
| `explain_rule` | Explains a business rule by its id (`BR-CO-15`, `BR-DE-1`, `PEPPOL-EN16931-R001`, a national CIUS rule): its message, level, the rule set it belongs to and how to fix it. | Not needed |

Every tool carries its MCP annotations: `validate_xml` is the only one that isn't read-only, and none is destructive.

## Cline

Open **MCP Servers → Configure → Configure MCP Servers** and add xmlvalid to `cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "xmlvalid": {
      "type": "streamableHttp",
      "url": "https://xmlvalid.com/mcp",
      "headers": {
        "Authorization": "Bearer xv_your_api_key"
      },
      "disabled": false,
      "autoApprove": ["search_schemas", "explain_error", "explain_rule"]
    }
  }
}
```

Replace `xv_your_api_key` with your key (see [Authentication](#authentication)). Or leave out `headers` and sign in instead: `search_schemas`, `explain_error` and `explain_rule` work without an account, and the first `validate_xml` makes Cline report that the server requires OAuth authorization. Authorize xmlvalid from Cline's MCP servers (in Cline CLI, `cline mcp`) and sign in to xmlvalid in the browser.

The same works in [Cline CLI](https://docs.cline.bot/cline-cli/overview), which reads `~/.cline/data/settings/cline_mcp_settings.json`; the tools show up as `xmlvalid__explain_rule` and so on.

## Other clients

**Claude** (claude.ai, desktop and mobile): xmlvalid is in [Claude's connector directory](https://claude.ai/directory/xmlvalid). **Claude Code**:

```bash
claude mcp add --transport http xmlvalid https://xmlvalid.com/mcp
```

then `/mcp` to sign in.

**Cursor** (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "xmlvalid": {
      "url": "https://xmlvalid.com/mcp"
    }
  }
}
```

**VS Code** (`.vscode/mcp.json`), asking for the key once instead of storing it in the file:

```json
{
  "servers": {
    "xmlvalid": {
      "type": "http",
      "url": "https://xmlvalid.com/mcp",
      "headers": {
        "Authorization": "Bearer ${input:xmlvalid-key}"
      }
    }
  },
  "inputs": [
    { "type": "promptString", "id": "xmlvalid-key", "description": "xmlvalid API key", "password": true }
  ]
}
```

Any other client that speaks Streamable HTTP takes the same URL. The server is listed in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=com.xmlvalid/xmlvalid) as `com.xmlvalid/xmlvalid`; [`server.json`](server.json) is its entry.

## Authentication

`validate_xml` and `get_validation` run as your xmlvalid account, so the quota and results are those of the [API](https://xmlvalid.com/docs/api). Two ways to give it:

- **API key.** Sign up at [xmlvalid.com](https://xmlvalid.com/register) and confirm your email, then create a key under [Dashboard → API keys](https://xmlvalid.com/dashboard/api). It starts with `xv_`; send it as `Authorization: Bearer xv_…`. This is the way for Cline and for scripts.
- **OAuth.** Clients that support MCP authorization (Claude, Claude Code, Cursor, VS Code) ask you to sign in to xmlvalid the first time you validate; no key to copy. The server supports dynamic client registration. Connected apps are listed under Dashboard → API keys, where you can disconnect them.

## Example prompts

- "Validate invoice.xml against the official rules and tell me what to fix" (the agent reads the file and sends it with `schema: "auto"`)
- "Check https://example.com/feed.xml against its XSD"
- "My Peppol invoice fails with BR-CO-15. What does it mean and how do I fix it in UBL?"
- "Explain cvc-complex-type.2.4.a: Invalid content was found starting with element 'cbc:Note'"
- "Which schema do I use for a Peppol BIS credit note?"
- "Write a test that generates our XRechnung and validates it with xmlvalid"

When a validation fails, fix the first errors first: later ones often follow from them, and business rules run only once the schema check passes.

## Privacy and limits

- Documents sent to `validate_xml` are validated on xmlvalid's servers in Germany and deleted afterwards; findings are kept for 24 hours. See [Security](https://xmlvalid.com/security) and the [DPA](https://xmlvalid.com/dpa).
- The Free plan includes 100 API validations a month and files up to 10 MB; [Pro](https://xmlvalid.com/pricing) 10,000 a month and files up to 1 GB.
- To validate without sending the document anywhere, use the [web validator](https://xmlvalid.com/validate): it runs the same engine in the browser (WebAssembly), and nothing is uploaded.

## Links

- [API and MCP documentation](https://xmlvalid.com/docs/api#mcp)
- [Schema catalog](https://xmlvalid.com/schemas) · [Business rules](https://xmlvalid.com/rules) · [Error guide](https://xmlvalid.com/errors)
- Other clients: [CLI](https://github.com/xmlvalid/cli), [GitHub Action](https://github.com/xmlvalid/validate-action), [n8n node](https://github.com/xmlvalid/n8n-nodes-xmlvalid)
- Support: support@xmlvalid.com

This repository holds the server's documentation and registry entry; the server itself runs at xmlvalid.com.

## License

MIT
