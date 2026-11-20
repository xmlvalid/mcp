# Installing the xmlvalid MCP server

xmlvalid is a hosted MCP server. There is nothing to clone, build or run: add its URL to the MCP settings.

- URL: `https://xmlvalid.com/mcp`
- Transport: Streamable HTTP (`"type": "streamableHttp"` in Cline)
- Authentication: an API key in the `Authorization` header, `Bearer xv_…`. Optional for `search_schemas`, `explain_error` and `explain_rule`; required for `validate_xml` and `get_validation`.

## Steps

1. Ask the user for their xmlvalid API key. If they don't have one: sign up at https://xmlvalid.com/register, confirm the email, then create a key at https://xmlvalid.com/dashboard/api (the Free plan includes 100 validations a month). The key starts with `xv_`. If the user only wants the lookup tools for now, skip the key and leave out `headers`.
2. Add this entry under `mcpServers` in `cline_mcp_settings.json`, keeping any servers already there:

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

3. Check the connection by calling `explain_rule` with `{"id": "BR-CO-15"}`. It should return the rule's message (the invoice total with VAT must equal the total without VAT plus the VAT total) and how to fix it.
4. With a key, check it with `search_schemas` `{"query": "peppol invoice"}` and then `validate_xml` on a small document such as `{"xml": "<note><to>A</to></note>"}` (no schema: a well-formedness check). A 401 saying "Validating needs an xmlvalid account" means the key wasn't sent: check the `Authorization` header.

## Using it

- To validate a local file, read it and pass its text as `xml`; for a file on the web pass `url`. Use `schema: "auto"` for e-invoices (the profile the invoice names), a slug from `search_schemas`, or leave it out for well-formedness only.
- Fix the first errors first: business rules run only once the schema check passes.
- Each finding has a `help_url` explaining the fix; `explain_rule` and `explain_error` return the same explanations.
