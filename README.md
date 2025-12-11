# AI2JSON — Turn Any Webpage Into Clean AI-Ready JSON

AI2JSON is a minimal API that converts any public webpage into a clean, structured JSON format designed for AI agents, LLMs, and automation pipelines.

Instead of parsing raw HTML, you call AI2JSON with a URL and receive a stable JSON document following the `webpage.ai.v0.1` specification.

## What AI2JSON does

From a single URL, AI2JSON returns:

- Main content text
- Sections and headings
- Page title and canonical URL
- Language detection
- Basic metadata (site name, dates, author when available)
- Factual summary (v0.1: simple heuristic, future: LLM-based)
- SHA-256 content hash

The response is a single JSON document that is easy to feed into:

- LLMs
- Agents
- RAG pipelines
- Monitoring tools
- Custom crawlers

## API overview (v0.1)

Base URL (example):

- https://api.ai2json.com

Endpoint:

- GET /v1/transform

Authentication:

- Header: Authorization: Bearer YOUR_API_KEY

Query parameters:

- url (required): the public page URL to transform
- lang (optional): preferred language for summary (for future versions)
- no_cache (optional): when true, forces a fresh fetch

Example HTTP request (conceptual):

GET /v1/transform?url=https://example.com/article
Authorization: Bearer YOUR_API_KEY
Accept: application/json

## Response format — webpage.ai.v0.1

The response is a JSON object with this top-level structure:

- spec: string, always "webpage.ai.v0.1"
- url: string, the requested URL
- canonical_url: string, the canonical URL if detected
- last_fetched: string, ISO 8601 UTC timestamp of the snapshot
- language: string, best-guess language code (for example "en" or "fr")
- content_hash: string, SHA-256 hash of the extracted content, prefixed with "sha256:"
- title: string, page title
- summary: string, short factual summary
- type: string, one of "article", "homepage", "unknown"
- sections: array of section objects
- links: array of extracted links (may be empty in v0.1)
- meta: object with optional metadata fields
- proof: reserved object for future integrity proofs (empty in v0.1)

Each entry in sections has:

- heading: string or null
- level: integer (0, 1, 2, 3)
- id: string stable identifier
- text: string containing clean section text

Minimal example:

{
  "spec": "webpage.ai.v0.1",
  "url": "https://example.com/article-ai",
  "canonical_url": "https://example.com/article-ai",
  "last_fetched": "2025-01-02T10:11:12Z",
  "language": "en",
  "content_hash": "sha256:abc123...",
  "title": "Example Article About AI",
  "summary": "Short factual summary of the page content.",
  "type": "article",
  "sections": [
    {
      "heading": "Introduction",
      "level": 1,
      "id": "introduction",
      "text": "Clean extracted text for the introduction section."
    }
  ],
  "links": [],
  "meta": {
    "site_name": "Example",
    "author": "John Doe",
    "published_at": "2025-01-01"
  },
  "proof": {}
}

The full, formal specification is described in:

- spec/webpage-ai-v0.1.md

## Typical use cases

AI2JSON is designed for developers and teams who need reliable text extraction for AI:

- LLM agents that must read web pages in a predictable format
- RAG pipelines that need pre-cleaned, sectioned content
- Monitoring of regulatory or documentation pages with content hashes
- Custom crawlers that prefer JSON over HTML
- Compliance workflows where deterministic snapshots matter

Instead of re-implementing parsing and cleaning logic in every project, AI2JSON centralises the work and exposes a single contract.

## Quickstart (conceptual)

1. Obtain an API key from AI2JSON.
2. Send a GET request to /v1/transform with:
   - the target page in the url parameter
   - your key in the Authorization header
3. Receive a JSON document in the webpage.ai.v0.1 format.
4. Use the JSON directly in your LLM prompts, agents, RAG pipelines, or data processing code.

Example flow:

- User or agent chooses a URL.
- Backend calls AI2JSON.
- Backend passes the resulting JSON to a model or stores it.

## Project structure

Repository layout (planned):

ai2json/
  api/
    worker.js                Cloudflare Worker implementing /v1/transform
  site/
    index.html               Landing page and minimal documentation
  spec/
    webpage-ai-v0.1.md       Formal specification of the JSON format
  PLAN.md                    Build plan for v0.1
  README.md                  This file

The API is implemented as a Cloudflare Worker.  
The site is a static landing and docs page, deployable on Vercel or any static host.

## Status

- Version: v0.1 (design and initial implementation)
- Scope:
  - Single endpoint: /v1/transform
  - HTML fetch and main-content extraction
  - Simple summary (heuristic)
  - Basic language detection
  - No JavaScript rendering yet
  - No public self-service billing yet

This repository tracks the format, implementation, and roadmap as AI2JSON evolves.

## Roadmap

Planned steps:

- v0.1
  - Stable webpage.ai.v0.1 spec
  - Basic HTML extraction and sectioning
  - Simple summary without LLM
  - Single API key for early users

- v0.2
  - LLM-generated summaries with explicit language control
  - Better metadata extraction
  - First public Free and Pro plans

- v0.3
  - JavaScript rendering for dynamic pages
  - Batch endpoint for multiple URLs

- v0.4
  - Basic dashboard for API keys and usage
  - Per-key rate limiting and quotas

- v1.0
  - Production-grade monitoring and logging
  - Webhooks and change detection
  - Billing integration
  - Stable, versioned specification

Roadmap items may be adjusted as the project gets feedback from real usage.

## License

MIT License.  
You are free to use, modify, and integrate AI2JSON in your own projects under the terms of the MIT license.
