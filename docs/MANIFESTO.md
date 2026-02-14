# Why Manifests Are the Cognitive API for Artificial Intelligence

**And why it took this long to name it.**

---

## The Question

On February 14, 2026, in a conversation between a human and an AI, a simple question surfaced: if an LLM already understands every standard Unix command from its training data, what happens when it encounters a capability that *wasn't* in its training data?

The answer was obvious once stated. The LLM reads a description — just like it reads `--help` output — and reasons about whether the capability fits the task. No retraining. No fine-tuning. No registration. Just text that a language model can understand.

A manifest isn't metadata. It's how AI learns.

This raised an uncomfortable follow-up: if the answer is this simple, why hasn't the industry converged on it yet?

## The Missing Lens

Three structural reasons explain why this pattern hasn't emerged from the existing communities — and why it took an outsider's perspective to name it.

### Perspective 1: ML Research Is Focused on Training-Time Learning

The entire field of machine learning is organized around a single premise: you make AI smarter by improving what happens *before* deployment. Better training data. Better architectures. Better fine-tuning. Better RLHF. Every paper, every benchmark, every PhD thesis is oriented around making the model's frozen knowledge more comprehensive.

Within that worldview, the idea that AI could learn about a new capability by reading a text file at runtime is understandably outside the field's focus. It doesn't require any ML innovation. There's no architecture to design, no benchmark to beat, no paper to publish. A JSON file with a description field isn't a research contribution. It's too simple to take seriously.

But simplicity isn't a flaw. It's a signal. The most important interface standards in computing history — ASCII, HTTP, HTML, DNS — were all derided as too simple by the people building more complex alternatives. The OSI model was seven layers of committee-designed perfection. TCP/IP was a hack that actually worked. The hack won.

### Perspective 2: Enterprise Architecture Defaults to Infrastructure

The enterprise technology world sees every problem as an infrastructure problem. Agent discovery? Build a registry. Agent communication? Design a protocol with lifecycle states, push notifications, and gRPC bindings. Agent payments? A whole separate protocol extension.

Google convened 150 organizations to build A2A — the Agent-to-Agent protocol. It's well-engineered. It's comprehensive. It has versioning, security cards, streaming support, and a growing ecosystem. It's also, as of version 0.3, still missing a registry specification. Their own community has over 100 comments on a GitHub discussion thread debating whether the registry should be centralized or decentralized. The protocol for agents to talk to each other doesn't yet include a standard way for agents to find each other.

This isn't an oversight. It's a symptom. Registry design is hard because registries may be the wrong abstraction for this particular problem. The internet tried centralized directories before — Yahoo, DMOZ, the semantic web. They all failed. Not because they were poorly executed, but because taxonomies require committees, committees require consensus, consensus requires compromise, and compromise produces specifications so broad they describe everything and match nothing.

The internet solved discovery with a format (HTML) and an ecosystem (crawlers, search engines). No registry. No taxonomy. No committee. Just publish and be found.

### Perspective 3: The Agent/App Distinction Is Economically Load-Bearing

The most powerful blind spot is economic. The AI industry needs "agents" to be a new and special category of software that requires new and special infrastructure.

If agents are just applications — if the thing being invoked doesn't change regardless of whether a human or an AI is calling it — then you don't need agent frameworks, agent marketplaces, agent orchestration platforms, agent observability tools, or agent-specific protocols. Billions of dollars in venture funding, thousands of startups, and entire product categories at Google, Microsoft, and Amazon depend on "agents" being fundamentally different from "applications."

It's worth asking: if a simpler answer exists, what structural forces would prevent it from surfacing?

But it is. An application that accepts input, does work, and produces output is a capability. `grep` is an agent — it accepts a task, executes autonomously, returns results. It just doesn't have a marketing team. The difference between an "agent" and an "application" is the invoker, not the thing being invoked.

Collapsing this distinction doesn't eliminate the need for complex multi-agent collaboration (A2A handles that well). It doesn't eliminate the need for structured tool interfaces (MCP handles that). What it eliminates is the assumption that AI needs an entirely new infrastructure to find things. It doesn't. It needs the same thing the web needed: a standard way to describe what exists and let the network figure out the rest.

## The Timing

This insight is only legible now. It couldn't have been articulated — or rather, it couldn't have been *useful* — even three years ago.

The concept of a "cognitive API" requires a reader. Someone — or something — has to read a natural language description of a capability and reason about whether it matches an intent. Before 2023, no system could do this reliably. You could publish all the manifests you wanted, but nothing could read them with enough comprehension to make discovery work.

Large language models changed that. An LLM can read a manifest description — "Ingests government meeting videos and produces structured transcripts with speaker identification, topic segmentation, and voting records" — and decide whether that capability matches the query "I need transcripts from last week's Portland city council meeting." The matching is semantic, not keyword-based. The reasoning is contextual, not rule-based.

The cognitive API for AI required an AI capable of cognition. That's a circular dependency that only resolves once the capability threshold is crossed. It was crossed sometime in 2023-2024. We're now in the window where the enabling technology exists but the infrastructure pattern hasn't been named.

Until now.

## The Unix Precedent

The design philosophy behind OAP isn't new. It's fifty-seven years old.

In 1969, Ken Thompson and Dennis Ritchie created Unix around a set of principles that turned out to be the natural architecture for AI-native computing. They just didn't know who the user would be.

**"Everything is a file"** was never about filesystems. It was about *everything is text that any process can reason about.* For fifty years, "any process" meant programs that could parse structured text. LLMs are the first process that can reason about *unstructured* text — natural language descriptions, ambiguous queries, contextual intent. Unix was waiting for a user that could read.

**"Do one thing well"** was violated by every generation of software because humans couldn't manage hundreds of small tools. We built monolithic applications because the operator — the human — needed integrated interfaces. AI agents don't. They can compose hundreds of small capabilities as easily as a shell script chains commands. The constraint was never the philosophy. It was the operator.

**"Expect the output of every program to become the input to another"** described composability before the word existed. We buried it under REST APIs, GraphQL schemas, and 47-page OpenAPI specifications because machine-to-machine communication required rigid contracts. But an LLM doesn't need a rigid contract to understand output. It reads the output, reasons about its shape and meaning, and decides how to use it. The pipe is back — and this time, the pipe can think.

The Unix philosophy was the right architecture deployed fifty years too early. OAP doesn't reinvent it. OAP is what happens when you take Unix seriously in an era where the operator is an intelligence that can read.

## The Web Precedent

The web provides the second precedent — for how discovery works at scale without centralized infrastructure.

In 1991, Tim Berners-Lee didn't build a directory of web pages. He built HTML — a format for describing content — and HTTP — a protocol for fetching it. That was the standardization layer. Everything above it — crawling, indexing, search, ranking — was left to the ecosystem. Multiple competing search engines emerged. None of them required the content publishers to register or categorize their pages. You published HTML. Crawlers found it. Search engines matched it to queries.

This architecture scaled to billions of pages without a single registry, taxonomy, or governing body for content organization. It worked because the standardization was at the *format* layer, not the *discovery* layer. Publishers controlled what they said. The ecosystem competed on how well it found things.

OAP applies exactly this pattern to capabilities. The manifest is the format. The well-known path is the convention. Everything above — crawling, indexing, matching intent to capability — is the ecosystem's job. Many crawlers, many indexes, many search engines, all competing on quality. No single point of failure. No gatekeeper. No committee deciding what categories exist.

## The Protocol Designer's Perspective

Full disclosure: the human half of this conversation spent thirteen years at Apple working on protocols, then built one of the first Silicon Valley ISPs — including hosting, colocation, and accelerated dial-up services. This isn't someone who stumbled into protocol design from the AI side. This is someone who has watched protocols succeed and fail for three decades and recognizes the pattern.

The protocols that win share three properties:

1. **They're simple enough that a single developer can implement them in an afternoon.** HTTP was simple. SOAP was not. HTTP won. HTML was simple. SGML was not. HTML won. TCP/IP was simple. The OSI model was not. TCP/IP won. Every successful protocol was accused of being "too simple" by the people building the complex alternative.

2. **They standardize the minimum viable surface and leave everything else to the ecosystem.** HTML standardized document structure. It didn't standardize design, layout, interactivity, or discovery. Those were left to CSS, JavaScript, and search engines — all of which emerged from the ecosystem, not from the spec. The spec stayed small. The ecosystem grew large.

3. **They don't require permission.** You didn't register your website with the W3C. You didn't submit your HTTP server for certification. You published a page and the network found it. Any protocol that requires registration, approval, or certification to participate is optimizing for control, not adoption.

OAP has all three properties. The spec is one page. It standardizes only the manifest format and the publishing convention. It requires no registration, no approval, and no fees. It is, by design, the minimum viable protocol that enables the maximum possible ecosystem.

Thirty years of watching protocols teaches you one thing: the spec that wins is always the one that looks too simple at first. It's too simple. It doesn't handle enough edge cases. It leaves too much undefined. And then it conquers the world because a million developers can actually use it, while the "proper" specification is still in committee review.

## What We're Actually Claiming

Let's be precise about what OAP asserts, because the claim is genuinely new:

**1. A manifest is not metadata. It's a cognitive interface.** The `description` field in an OAP manifest is not a label for a search index. It's the text an LLM reads to understand a capability it has never encountered before. This is functionally identical to how an LLM reads `--help` output, `man` pages, or documentation. The manifest is the interface between AI and an unknown capability — a cognitive API.

**2. Discovery is not an infrastructure problem. It's a publishing problem.** Thirty years of internet history show that discovery works when formats are standardized and discovery mechanisms are left to the ecosystem. Registries centralize. Taxonomies ossify. Search engines compete and improve. OAP bets on the web model, not the registry model.

**3. The agent/app distinction is worth questioning.** It's artificial because the thing being invoked doesn't change based on who invokes it. It's worth questioning because it drives the industry to build redundant infrastructure for "agents" that already exists for "applications." Collapsing the distinction simplifies everything.

**4. Composability is the agent's job, not the protocol's.** Unix didn't build composability metadata into its tools. It gave tools stdio and let the operator figure out the piping. OAP gives capabilities a manifest and lets the agent figure out the composition. The intelligence required to compose capabilities now exists in the invoker — the LLM — not in the specification.

**5. The enabling technology only just arrived.** A format designed for AI comprehension requires AI capable of comprehension. LLMs crossed that threshold recently. OAP is the first protocol designed specifically for the post-threshold world — where the consumer of the specification is an artificial intelligence, not a parser or a registry.

## The Reference Architecture

OAP is deliberately silent on how discovery should work. But it's worth illustrating what the ecosystem enables — not as prescription, but as proof that the web model works for capabilities the same way it works for documents.

The architecture has three components, and anyone can run all of them:

**Manifests live where they belong.** On publishers' domains. The capability owner writes the description, controls the truth, and updates it when things change. No central repository of "all capabilities." Just like HTML lives on web servers, not inside Google's database. Google has a *cache*. The source of truth is always the publisher.

**A local vector database is the agent's personal search engine.** Crawlers index manifests from across the internet, embed them as vectors, and store them locally. When an agent needs a capability, it does a similarity search against its own index — millisecond latency, no network call, no central service. This is `$PATH` for the AI era. Not "search the entire internet every time." Search your index of known capabilities. Re-crawl periodically to catch updates, the same way a search engine re-indexes web pages.

**A small, local LLM handles intent-to-manifest matching.** Reading a manifest and deciding "does this capability fit my task?" doesn't require a frontier model. A 7B or even 3B parameter model can do this reasoning. The expensive model handles the complex task — the actual work. The cheap model handles capability selection — the discovery. Two different cognitive jobs at two different cost points.

The runtime flow:

```
Agent receives a task
    ↓
Small LLM + vector DB: "what capabilities match this intent?"
    → millisecond similarity search across local manifest index
    → small LLM reads top candidates, picks the best fit
    ↓
Big LLM: "use this capability to complete the task"
    → invokes the capability via the manifest's invoke field
    → reasons about the output, composes next step if needed
    ↓
Done
```

This is exactly how web search works, layer by layer:

| Web Search | OAP Discovery |
|------------|---------------|
| Googlebot crawls HTML | Crawler indexes manifests |
| Search index stores pages | Vector DB stores embeddings |
| User types a query | Agent has an intent |
| Results ranked by relevance | Manifests ranked by similarity |
| User reads snippets, picks a result | Small LLM reads manifests, picks a capability |
| User clicks through to the page | Big LLM invokes the capability |

The critical property of this architecture: **anyone can run the whole stack.** A small LLM, a vector database, and a crawler. That's a weekend project for a developer with a laptop. No cloud dependency. No API costs for discovery. No central service to go down or get acquired or start charging rent. The expensive frontier model only gets called when there's actual work to do — never wasted on finding the work.

This is also why the `description` field is the most important field in the manifest. It's not just what the small LLM reads to match intent. It's what gets embedded as a vector for similarity search. A well-written description clusters near relevant queries in vector space. A vague description gets lost. The quality of discovery is directly proportional to the quality of the description — the same way the quality of web search results is proportional to the quality of the content on the page.

The manifest is simultaneously a cognitive interface (readable by LLMs), a search document (embeddable as a vector), and a machine contract (parseable by code). One format serving three functions. That's not an accident. That's what happens when you design for the simplest possible representation of truth.

## The One-Sentence Version

**OAP is how AI learns about capabilities that weren't in its training data.**

Everything else — the manifest format, the well-known path, the design principles — is implementation detail in service of that single idea.

If you accept that AI's knowledge is frozen at training time, and you accept that new capabilities are being created faster than any model can be retrained, then you need a mechanism for runtime knowledge extension. That mechanism needs to be readable by AI, publishable by anyone, and discoverable by the network.

That's a manifest. Published at a well-known path. On the open internet.

One file. One location. One page spec. Public infrastructure.

---

*This document was written to accompany the [Open Application Protocol specification](SPEC.md), [reference architecture](ARCHITECTURE.md), and [trust overlay](TRUST.md). OAP is released under CC0 1.0 Universal — no rights reserved.*
