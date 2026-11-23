---
name: validate-xml
description: Validate an XML document or e-invoice (UBL, CII, XRechnung, Peppol BIS, Factur-X / ZUGFeRD, KSeF, FatturaPA) with the xmlvalid MCP server and fix what fails. Use when the user asks to check, validate or debug XML, an XSD / DTD / RelaxNG schema error, or a business rule such as BR-CO-15.
---

# Validate XML with xmlvalid

1. Pick the schema. For an e-invoice that names its profile (BT-24, `cbc:CustomizationID` / `ram:GuidelineSpecifiedDocumentContextParameter`), use `auto`. Otherwise call `search_schemas` with the format ("peppol invoice", "xrechnung cii", "pain.001") and take the slug. Without a schema only well-formedness is checked.
2. Call `validate_xml` with the document as `xml` (its text) or `url` (a public address) and the `schema`. The first call asks the user to sign in to xmlvalid in the browser; each call counts as one API validation.
3. If it returns a pending status, call `get_validation` with its id.
4. Read the findings in order. Fix the first errors first: later ones often follow from them, and business rules run only once the schema check passes. Each finding has a line, column, rule id and a link that explains it.
5. For a message you don't understand, call `explain_error` with it (libxml2, Java's Xerces `cvc-*` and .NET wordings are known). For a business rule id, call `explain_rule`. Neither needs an account.
6. After changing the document, validate it again until it passes.

To write a new invoice instead (XRechnung, Peppol BIS, ZUGFeRD / Factur-X, France CTC), call `create_invoice` with the profile, the parties, the lines and the payment; it returns the XML already checked, and the same findings to fix when it fails.
