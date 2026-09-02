# Evergreen eFax Search Improvements

## Current Change

The v2 Inbox and Sent Faxes pages now use a broader client-side search model:

- All Search: searches fax metadata, dates, filenames, status fields, phone numbers, contact names, organizations, contact email, contact notes, and any indexed content fields returned by the Worker.
- Person / Company: searches saved eFax contacts plus fax caller/recipient number fields.
- Fax Content: searches extracted/OCR text fields when those fields exist in the fax or archive record.

The pages also load the shared contacts list from the existing contacts API, match contacts to fax records by normalized phone number, display the contact/company on matching fax rows, and include contact/company columns in CSV exports.

## Content Search Reality

SRFax returns fax PDFs. Most fax PDFs are image-based, so searching the actual body text requires a server-side extraction/indexing step. The front end is now ready to search extracted text fields, but full document-body search is only as complete as the OCR/index data provided by the Worker.

Recommended next build path:

1. Add an archive indexing job that retrieves fax PDFs through the existing secured Worker path.
2. Extract text from each PDF and store only the minimum searchable text/metadata needed for lookup.
3. Extend the archive Worker with an `Archive_Search` action for server-side search once the index is populated.
4. Keep live SRFax retrieval separate from the archive search index so the current send/inbox/outbox workflows remain stable.
5. Keep the existing origin and `X-Client-ID` controls, and continue storing SRFax credentials only as Cloudflare Worker secrets.

Cloudflare-supported implementation options:

- Cloudflare Workers AI Markdown Conversion can convert documents to Markdown through an `AI` binding, which is useful for text extraction workflows: https://developers.cloudflare.com/workers-ai/features/markdown-conversion/
- Cloudflare R2's PDF processing tutorial demonstrates extracting PDF text with Worker-side logic and using Workers AI in an event-driven indexing workflow: https://developers.cloudflare.com/r2/tutorials/summarize-pdf/
- Cloudflare AI Search can be considered if Evergreen wants semantic/hybrid search beyond exact keyword matching: https://developers.cloudflare.com/changelog/product-group/developer-platform/8/

## Data Handling Guardrails

- Do not commit SRFax credentials, fax PDFs, raw fax bodies, OCR output, PHI, or attachment names to Git.
- Do not paste raw fax body content into Hudu, tickets, runbooks, or agent instructions.
- Store search text only in the secured Cloudflare data layer selected for the production build.
- Keep exported customer evidence redacted to counts, field names, and operational status.

## Verification

Completed local verification:

- JavaScript syntax check passed for `v2/inbox.html`.
- JavaScript syntax check passed for `v2/outbox.html`.
- Mocked browser smoke test passed for:
  - contact-enriched row display,
  - Person / Company search,
  - Fax Content search using an indexed text field,
  - normalized phone-number search.
