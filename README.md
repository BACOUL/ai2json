# AI2JSON — Web → JSON for AI Reading

AI2JSON is a free, minimal API that converts any public webpage into clean, structured JSON, so AI systems can read web content without HTML noise.

It does not summarize, interpret, or judge content.  
It only translates webpages into a stable JSON structure.

---

## What AI2JSON does

From a single URL, AI2JSON returns:

- Page title
- Clean main text
- Sections and headings
- Language detection
- Canonical URL (when available)
- SHA-256 hash of the extracted content

The output is deterministic and designed to be directly usable by:

- LLMs
- AI agents
- RAG pipelines
- Monitoring and diffing tools
- Automation workflows

---

## What AI2JSON does NOT do

- No summary
- No semantic analysis
- No interpretation
- No truth, compliance, or validation logic

AI2JSON is a translator, not an intelligence.

---

## API (v0)

Endpoint:

GET /parse?url=https://example.com

- Free to use
- No API key required
- Rate-limited to prevent abuse

---

## Example response

{
  "spec": "ai2json.v0",
  "url": "https://example.com",
  "title": "Example Page",
  "language": "en",
  "content_hash": "sha256:abc123...",
  "sections": [
    {
      "heading": "Introduction",
      "level": 1,
      "text": "Clean extracted content."
    }
  ]
}

---

## Why use AI2JSON

- AI reads less noise
- Fewer hallucinations caused by layout or boilerplate
- Lower token usage
- No scraping or parsing logic to maintain
- Stable and reproducible inputs

---

## Typical use cases

- Feeding web pages into LLM prompts
- Pre-cleaning content for RAG pipelines
- Monitoring documentation or regulatory pages
- Comparing page versions using content hashes
- Building lightweight crawlers that prefer JSON over HTML

---

## Status

- Version: v0
- Free and public
- Single endpoint
- Deterministic output
- No authentication

If usage grows, the project may evolve.  
For now, AI2JSON focuses on doing one thing well.

---

## License

MIT License
