# xmlvalid

The xmlvalid MCP server validates XML documents and e-invoices and explains what fails.

- `search_schemas` finds the catalog slug for a format ("peppol invoice", "xrechnung cii", "pain.001"). Use `auto` instead when the document is an e-invoice that names its profile (BT-24).
- `validate_xml` checks a document, given as its text or a public URL, against that schema. It needs the user to sign in to xmlvalid (OAuth, the browser opens on the first call) or an API key, and counts as one API validation. Without a schema it checks well-formedness.
- `create_invoice` writes an e-invoice (XRechnung, Peppol BIS, ZUGFeRD / Factur-X, France CTC) from the invoice's data (profile, parties, lines, payment) and checks it the same way; it returns the XML with the result and the fields it had no place for. Same sign-in and quota as `validate_xml`.
- `get_validation` reads the result of a validation that was still running.
- `explain_error` explains a parser or validator message (libxml2, Java's Xerces, .NET) and how to fix it. `explain_rule` explains a business rule such as `BR-CO-15`. Neither needs an account.

When a validation fails, fix the first errors first: later ones often follow from them, and business rules run only once the schema check passes. Each finding has its line, column and rule id; quote them when you explain the fix.
