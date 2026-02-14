# OAP Reference Architecture

**A complete discovery stack that runs on a laptop.**

---

## Design Principle

Knowledge lives where it belongs. Manifests stay on publishers' domains. Discovery runs locally. The expensive model does the work. The cheap model finds the work. No cloud dependency for discovery. No central service to go down, get acquired, or start charging rent.

## The Stack

```
┌─────────────────────────────────────────────────┐
│                   Agent Task                     │
│         "Transcribe last week's Portland         │
│          city council meeting"                   │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│            Discovery Layer (local)               │
│                                                  │
│  ┌───────────┐    ┌──────────────────────────┐  │
│  │ Small LLM │◄──►│  Vector DB               │  │
│  │ (3B-8B)   │    │  (embedded manifest      │  │
│  │           │    │   index)                  │  │
│  └───────────┘    └──────────────────────────┘  │
│                                                  │
│  "myNewscast Meeting Processor matches this      │
│   task with 0.94 similarity"                     │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│           Execution Layer (local or cloud)        │
│                                                  │
│  ┌─────────────┐    ┌────────────────────────┐  │
│  │ Frontier LLM│───►│ Invoke capability via   │  │
│  │ (Claude,    │    │ manifest's invoke field  │  │
│  │  GPT, etc.) │    │                          │  │
│  └─────────────┘    └────────────────────────┘  │
│                                                  │
│  Reads manifest, constructs request, reasons     │
│  about output, composes next step if needed       │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│               Crawl Layer (background)           │
│                                                  │
│  Periodically fetches /.well-known/oap.json      │
│  from known domains. Embeds new/updated          │
│  manifests. Refreshes vector index.              │
│  Checks health endpoints. Prunes dead entries.   │
└─────────────────────────────────────────────────┘
```

## Three Cognitive Jobs, Three Cost Points

The key insight: discovery and execution are different cognitive tasks requiring different levels of intelligence. Don't waste a frontier model on finding work. Don't trust a tiny model with doing the work.

| Job | What It Does | Model Size | Cost |
|-----|-------------|-----------|------|
| **Similarity search** | Finds candidate manifests matching agent intent | No LLM — pure vector math | Near zero |
| **Manifest reasoning** | Reads top candidates, picks the best fit for the task | Small LLM (3B-8B) | Minimal |
| **Task execution** | Invokes capability, reasons about output, composes pipeline | Frontier LLM | Standard |

The expensive model never wastes tokens on discovery. The cheap model never attempts complex reasoning. Each model does what it's good at.

---

## Minimum Hardware

### Tier 1: Laptop (Development / Personal Agent)

For a developer running their own discovery stack locally.

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | Any modern 4-core | 8+ cores (Apple M-series, AMD Ryzen 7, Intel i7) |
| RAM | 16 GB | 32 GB |
| GPU | Not required (CPU inference works) | Apple M-series unified memory, or NVIDIA GPU with 8+ GB VRAM |
| Storage | 20 GB free (SSD) | 50 GB free (NVMe SSD) |
| OS | macOS, Linux, Windows (WSL2) | macOS or Linux |

This runs the small LLM for manifest reasoning, the vector database for similarity search, and the crawler — all locally. The frontier model for task execution is called via API (Claude, GPT, etc.) or runs locally if you have the hardware.

**Performance expectations:** Discovery latency under 500ms. Vector search under 50ms. Manifest reasoning 1-3 seconds on CPU, under 500ms with GPU. Handles a manifest index of 100,000+ capabilities comfortably.

### Tier 2: Server (Team / Production)

For a team running a shared discovery service.

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 8 cores | 16+ cores |
| RAM | 32 GB | 64 GB |
| GPU | NVIDIA with 12+ GB VRAM (RTX 3060 or better) | NVIDIA A10 / L4 (24 GB VRAM) |
| Storage | 100 GB NVMe SSD | 500 GB NVMe SSD |
| Network | 100 Mbps | 1 Gbps |

Supports concurrent discovery queries from multiple agents. Can index millions of manifests. Runs the small LLM with GPU acceleration for sub-100ms manifest reasoning.

### Tier 3: Virtual / Cloud

Any of the above can run in cloud VMs. Recommended instances:

| Provider | Instance Type | Monthly Cost (approx.) |
|----------|--------------|----------------------|
| AWS | g5.xlarge (1x A10G, 24GB VRAM, 4 vCPU, 16GB RAM) | ~$500 |
| GCP | g2-standard-4 (1x L4, 24GB VRAM, 4 vCPU, 16GB RAM) | ~$450 |
| Azure | Standard_NC4as_T4_v3 (1x T4, 16GB VRAM, 4 vCPU, 28GB RAM) | ~$400 |
| Budget | Any 8-core, 32GB RAM VPS (CPU-only inference) | ~$50-100 |

A CPU-only VPS works fine for moderate loads — the small LLM runs slower but discovery latency remains acceptable for most use cases.

---

## Small LLM Recommendations

The discovery LLM has one job: read a handful of manifest descriptions and decide which one best matches the agent's task. This requires instruction following, reading comprehension, and basic reasoning — not world knowledge, creative writing, or code generation.

### Recommended Models (as of February 2026)

| Model | Parameters | VRAM (quantized) | Why |
|-------|-----------|-------------------|-----|
| **Qwen 3 4B** | 4B | ~3 GB (Q4) | Best balance of size and reasoning. Hybrid think/no-think modes — use fast mode for manifest matching. Apache 2.0 license. |
| **Phi-4-mini-instruct** | 3.8B | ~3 GB (Q4) | Strong instruction following and reasoning at minimal size. 128K context window. MIT license. |
| **Llama 3.2 3B** | 3B | ~2 GB (Q4) | Smallest viable option. Runs on almost anything. Good enough for manifest matching. Meta license. |
| **SmolLM3 3B** | 3B | ~2 GB (Q4) | Fully open from Hugging Face. Outperforms Llama 3.2 3B on most benchmarks. Transparent training methodology. |
| **Qwen 2.5 7B Instruct** | 7B | ~5 GB (Q4) | If you have the VRAM, best-in-class instruction following at this size. Strong at structured reasoning. |

### Model Selection Guidance

- **Constrained hardware (laptop, edge):** Llama 3.2 3B or SmolLM3 3B. They run on CPU with 16GB system RAM.
- **Good hardware (M-series Mac, gaming GPU):** Qwen 3 4B or Phi-4-mini. Best reasoning per parameter.
- **Server with GPU:** Qwen 2.5 7B Instruct. Overkill for manifest matching, but fast with GPU acceleration and handles edge cases better.

### Runtime

**Ollama** is the recommended runtime for all models. Single command install, no Python environment, no CUDA configuration:

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull your chosen model
ollama pull qwen3:4b          # Recommended default
ollama pull phi4-mini          # Alternative
ollama pull llama3.2:3b        # Minimal option
ollama pull qwen2.5:7b         # If you have the VRAM
```

Ollama exposes a local API at `http://localhost:11434` that your discovery service calls.

---

## Vector Database Recommendations

The vector database stores embedded manifest descriptions and performs similarity search when an agent has a task. Requirements: fast similarity search, metadata filtering, easy to run locally, and handles 100K-10M vectors without breaking a sweat.

### Recommended Options

| Database | Best For | Deployment | License |
|----------|---------|-----------|---------|
| **ChromaDB** | Getting started, prototyping, small indexes (<1M manifests) | Embedded (in-process), no server needed | Apache 2.0 |
| **Qdrant** | Production, larger indexes, filtered search | Docker container or embedded (Rust) | Apache 2.0 |
| **LanceDB** | Edge/embedded, serverless, zero-ops | Embedded (in-process), no server | Apache 2.0 |
| **Milvus Lite** | Local development with path to enterprise scale | Embedded (Python library) | Apache 2.0 |

### Selection Guidance

- **Weekend prototype:** ChromaDB. `pip install chromadb` and you're running. In-memory mode eliminates all setup friction. Rewritten in Rust in 2025 — 4x faster than original.
- **Production local service:** Qdrant. Written in Rust. Sub-100ms queries on millions of vectors. Excellent metadata filtering for narrowing results by tags, capability type, or trust level. Run via Docker:

```bash
docker run -p 6333:6333 qdrant/qdrant
```

- **Embedded/edge:** LanceDB. No server process at all — it's a library that reads/writes directly to disk. Perfect for agents running on laptops or edge devices. Zero operational overhead.

### Embedding Model

Manifests need to be converted to vectors before storage. The embedding model determines how well similarity search matches intent to capability.

| Model | Dimensions | Size | Notes |
|-------|-----------|------|-------|
| **all-MiniLM-L6-v2** | 384 | 80 MB | Fast, good enough for most cases. The default starting point. |
| **nomic-embed-text** | 768 | 274 MB | Better semantic understanding. Runs via Ollama: `ollama pull nomic-embed-text` |
| **mxbai-embed-large** | 1024 | 670 MB | Highest quality local embeddings. Use if accuracy matters more than speed. |

For the OAP use case, `nomic-embed-text` via Ollama is the sweet spot — good quality, runs locally alongside your discovery LLM with no additional infrastructure.

---

## Crawler Design

The crawler is the background process that builds and maintains your local manifest index. It's conceptually identical to a web crawler, but only looks for one file.

### Basic Crawl Loop

```
1. Maintain a list of domains to crawl (seed list)
2. For each domain:
   a. Fetch https://{domain}/.well-known/oap.json
   b. If found and valid:
      - Embed the description field
      - Store manifest + vector in the database
      - Record timestamp and health status
   c. If not found or invalid:
      - Mark as inactive, retry with backoff
3. Optionally check health endpoints for active manifests
4. Sleep. Repeat.
```

### Seed Discovery

Where does the initial list of domains come from?

- **Manual curation:** Start with domains you know. Your own apps. Partner services. Community lists.
- **DNS TXT scanning:** Look for `_oap` TXT records on known domains.
- **Sitemap-style submission:** Accept domain submissions via a simple API or web form.
- **Referral crawling:** When a manifest includes a `publisher.url`, crawl that domain too.
- **Web crawl piggyback:** If you're running a general web crawler, check for `/.well-known/oap.json` on every domain you visit.

### Crawl Frequency

| Manifest Status | Crawl Interval |
|----------------|---------------|
| New / recently changed | Every 6 hours |
| Stable (unchanged for 7+ days) | Every 24-48 hours |
| Inactive (health check failing) | Every 72 hours, then prune |

Respect `updated` field in manifests — if it hasn't changed, don't re-embed.

---

## Putting It All Together

### Minimal Viable Discovery Stack

A working OAP discovery system in four components:

```
┌──────────────┐     ┌──────────────┐
│   Crawler    │────►│  ChromaDB    │
│  (Python     │     │  (embedded)  │
│   script)    │     │              │
└──────────────┘     └──────┬───────┘
                            │
                     ┌──────▼───────┐
                     │   Ollama     │
                     │  (Qwen 3 4B │
                     │   + nomic   │
                     │   embed)    │
                     └──────┬───────┘
                            │
                     ┌──────▼───────┐
                     │  Discovery   │
                     │  API         │
                     │  (FastAPI)   │
                     └──────────────┘
```

**Total dependencies:** Python, Ollama, ChromaDB (pip install). No Docker required. No Kubernetes. No cloud account.

**Total hardware:** Any machine with 16GB RAM and 10GB disk.

**Setup time:** Under an hour. Most of that is downloading the model.

### Example Discovery Query

```python
# Agent has a task
task = "I need to transcribe a Portland city council meeting from last week"

# 1. Embed the task intent
task_vector = ollama.embed("nomic-embed-text", task)

# 2. Search the manifest index
results = chromadb.query(
    query_embeddings=[task_vector],
    n_results=5
)

# 3. Small LLM picks the best match
candidates = [load_manifest(r) for r in results]
prompt = f"""Given this task: {task}

Which of these capabilities best fits? Read each description carefully.

{format_candidates(candidates)}

Respond with the name of the best match and why."""

best_match = ollama.generate("qwen3:4b", prompt)

# 4. Frontier LLM executes the task using the manifest's invoke field
selected_manifest = get_manifest(best_match.name)
# Hand off to Claude/GPT/etc. with the manifest as context
```

### Production Discovery Stack

For a team or service handling concurrent discovery queries:

```
┌──────────────┐     ┌──────────────┐
│   Crawler    │────►│   Qdrant     │
│  (scheduled, │     │  (Docker)    │
│   distributed│     │              │
│   workers)   │     └──────┬───────┘
└──────────────┘            │
                     ┌──────▼───────┐
                     │   Ollama     │
                     │  (Qwen 2.5  │
                     │   7B + GPU) │
                     └──────┬───────┘
                            │
                     ┌──────▼───────┐
                     │  Discovery   │
                     │  Service     │
                     │  (FastAPI +  │
                     │   caching)   │
                     └──────────────┘
```

Add Redis for caching frequent queries. Add a queue (Celery, etc.) for crawl job distribution. Same architecture, just hardened for concurrent load.

---

## What This Architecture Enables

**Anyone can run the whole stack.** A small LLM, a vector database, a crawler, and a thin API layer. That's a weekend project. No cloud dependency. No API costs for discovery. The frontier model only gets called when there's actual work to do.

**Discovery is private.** Your agent's queries never leave your machine during the discovery phase. Only the execution phase — invoking the actual capability — requires a network call. What you're searching for stays local.

**Multiple competing indexes can coexist.** Just like there are multiple web search engines, there can be multiple OAP crawlers and indexes. A general-purpose index. A healthcare-specific index. An enterprise-internal index. Competition at the discovery layer. Standardization at the manifest layer.

**The manifest quality drives discovery quality.** The `description` field is simultaneously what the small LLM reads for reasoning, and what gets embedded for vector similarity search. A well-written description clusters near relevant queries in vector space. A vague description gets lost. The quality of discovery is directly proportional to the quality of the manifest — same as the web.

---

## Not Prescribed, Illustrated

This document describes one way to build an OAP discovery stack. It is not part of the specification. The spec defines only the manifest format and publishing convention. How you discover, index, and reason about manifests is your business.

We publish this reference architecture to prove the ecosystem is buildable — by anyone, on commodity hardware, in a weekend. The stack described here is functional, affordable, and open. Improve on it.

---

*This document accompanies the [Open Application Protocol specification](SPEC.md), [trust overlay](TRUST.md), and [manifesto](MANIFESTO.md). OAP is released under CC0 1.0 Universal — no rights reserved.*
