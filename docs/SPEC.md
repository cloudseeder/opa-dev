# OAP Manifest Specification v1.0

## Publishing

Host a JSON file at `/.well-known/oap.json` on your domain over HTTPS.

## Manifest Format

```json
{
  "oap": "1.0",
  "name": "string — Capability name",
  "description": "string — What this does, in plain English. Write for an LLM that needs to decide if this capability fits a task. Be specific about what makes this different from alternatives. Max 1000 chars.",
  "url": "string (optional) — Human-facing URL",

  "input": {
    "format": "string — MIME type (e.g., text/plain, application/json)",
    "description": "string — What this capability needs. Describe the shape and meaning of expected input.",
    "schema": "string (optional) — URL to JSON Schema or OpenAPI spec for structured input"
  },

  "output": {
    "format": "string — MIME type of what this produces",
    "description": "string — What comes back. Describe the shape and meaning of the output.",
    "schema": "string (optional) — URL to JSON Schema for structured output"
  },

  "invoke": {
    "method": "string — HTTP method (GET, POST) or 'stdio' for command-line tools",
    "url": "string — Endpoint URL, or command path for stdio",
    "auth": "string (optional) — none | api_key | oauth2 | bearer",
    "auth_url": "string (optional) — Where to get credentials if auth is required",
    "streaming": "boolean (optional) — Whether this supports streaming responses"
  },

  "publisher": {
    "name": "string (optional) — Person or organization",
    "contact": "string (optional) — Contact email",
    "url": "string (optional) — Publisher website"
  },

  "examples": [
    {
      "input": "string or object — Example input",
      "output": "string or object — What this produces for that input",
      "description": "string (optional) — What this example demonstrates"
    }
  ],

  "tags": ["string (optional) — Freeform tags for indexing. Not a taxonomy. Just hints."],

  "health": "string (optional) — URL returning HTTP 200 if the capability is operational",
  "docs": "string (optional) — URL to documentation",
  "version": "string (optional) — Capability version",
  "updated": "string (optional) — ISO 8601 date of last manifest update"
}
```

## Required Fields

Only four things are required. Everything else is optional.

| Field | Why |
|-------|-----|
| `oap` | Protocol version, for forward compatibility |
| `name` | What to call it |
| `description` | What it does — this is the most important field. An LLM reads this to decide if the capability matches a task. Write it well. |
| `invoke` | How to call it |

Input and output descriptions are technically optional but strongly recommended. A capability without described input/output is like a Unix tool without a `man` page — it works, but nobody will use it.

## Examples

### Simple: A text summarizer

```json
{
  "oap": "1.0",
  "name": "Summarize",
  "description": "Accepts any text and returns a concise summary. Handles documents up to 100,000 words. Returns plain text, not markdown. Optimized for meeting transcripts and legal documents but works on anything.",
  "input": {
    "format": "text/plain",
    "description": "The text to summarize. Any length up to 100k words."
  },
  "output": {
    "format": "text/plain",
    "description": "A concise summary, typically 10-20% of the original length."
  },
  "invoke": {
    "method": "POST",
    "url": "https://summarize.example.com/api/v1/summarize",
    "auth": "api_key",
    "auth_url": "https://summarize.example.com/developers"
  },
  "examples": [
    {
      "input": "The quarterly earnings report showed a 12% increase in revenue...",
      "output": "Revenue grew 12% with margins expanding to 34%. Management raised full-year guidance.",
      "description": "Financial document summarization"
    }
  ]
}
```

### Minimal: A civic meeting transcriber

```json
{
  "oap": "1.0",
  "name": "myNewscast Meeting Processor",
  "description": "Ingests government meeting videos and produces structured transcripts with speaker identification, topic segmentation, and voting records. Covers city councils, planning commissions, school boards, and similar civic bodies. Input is a URL to a video. Output is a structured JSON transcript.",
  "input": {
    "format": "text/plain",
    "description": "URL to a publicly accessible government meeting video"
  },
  "output": {
    "format": "application/json",
    "description": "Structured transcript with speakers, topics, timestamps, motions, and votes"
  },
  "invoke": {
    "method": "POST",
    "url": "https://api.mynewscast.com/v1/process",
    "auth": "bearer"
  }
}
```

### The one that started it all: grep

```json
{
  "oap": "1.0",
  "name": "grep",
  "description": "Searches text for lines matching a pattern. Accepts regular expressions or fixed strings. Reads from stdin or named files. Returns matching lines to stdout, one per line. Exit code 0 if matches found, 1 if not. Supports recursive directory search, case-insensitive matching, inverted matching (lines that don't match), and context lines before/after matches. The most common text search tool in Unix.",
  "input": {
    "format": "text/plain",
    "description": "Text on stdin or file paths as arguments, plus a search pattern as the first argument. Pattern can be a basic regex, extended regex (-E), or fixed string (-F)."
  },
  "output": {
    "format": "text/plain",
    "description": "Matching lines printed to stdout, one per line. With -c, prints only a count. With -l, prints only filenames containing matches. With -n, prefixes each line with its line number."
  },
  "invoke": {
    "method": "stdio",
    "url": "grep"
  },
  "examples": [
    {
      "input": "echo 'hello world\ngoodbye world\nhello again' | grep hello",
      "output": "hello world\nhello again",
      "description": "Simple pattern matching on stdin"
    }
  ],
  "docs": "https://www.gnu.org/software/grep/manual/grep.html",
  "tags": ["search", "text", "pattern", "regex", "filter", "cli"]
}
```

### Unix-native: A command-line tool

```json
{
  "oap": "1.0",
  "name": "jq",
  "description": "Command-line JSON processor. Filters, transforms, and formats JSON data using a concise expression language. Like sed for JSON. Takes JSON on stdin, applies a filter expression, produces transformed JSON on stdout.",
  "input": {
    "format": "application/json",
    "description": "JSON data on stdin, plus a filter expression as a command-line argument"
  },
  "output": {
    "format": "application/json",
    "description": "Transformed JSON on stdout"
  },
  "invoke": {
    "method": "stdio",
    "url": "jq"
  },
  "docs": "https://jqlang.github.io/jq/manual/",
  "tags": ["json", "transform", "filter", "cli"]
}
```

## Discovery

OAP does not define a discovery mechanism. It defines what you publish. Discovery is a separate concern.

Approaches that work:

- **Direct fetch.** If you know a domain, check `/.well-known/oap.json`. Like checking `robots.txt`.
- **Crawling.** Index services crawl domains and index manifests. Anyone can run a crawler. Many should.
- **Voluntary submission.** Submit your domain to index services. Like submitting a sitemap to Google.
- **DNS TXT hint.** Optionally add `_oap.example.com TXT "v=oap1"` to signal participation without requiring a fetch.

Multiple discovery mechanisms can and should coexist. Competition at the discovery layer. Standardization at the manifest layer.

## What This Intentionally Omits

| Omission | Reason |
|----------|--------|
| Categories / taxonomy | Taxonomies require committees and ossify. Tags are freeform. LLMs reason about descriptions, not category trees. |
| Trust scoring | Trust is in the eye of the beholder. An agent's trust model is its own business. |
| Pricing | Not every capability has a price. When it does, `description` or `docs` can cover it. |
| Composability metadata | The agent figures out composition by reading descriptions — the same way it reads `--help` and decides which flags to pass. |
| Rate limiting / quotas | Operational details belong in API docs, not the manifest. |
| Payment model | This is the honest one. Without a way for agents to carry a wallet, authorize spending against a budget, and transact autonomously, a huge piece of the puzzle is missing. Google's AP2 protocol is an early attempt. But solving payments inside a manifest spec would violate everything OAP stands for — it's a separate, hard problem that deserves its own protocol, not a field bolted onto a JSON file. OAP tells agents what exists. How agents pay for it is someone else's important work. |
| Registry API | There is no registry. Publish your manifest. The rest is the internet's job. |

## Design Philosophy

The best protocol is the one people actually use. People use things that are simple. OAP is simple.

If you can write a JSON file, you can publish a capability. If you can read English, you can understand a manifest. If you're an LLM, you can match manifests to tasks.

One file. One location. One page spec.
