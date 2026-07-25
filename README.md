# siRNA Therapeutics Knowledge Base — MCP server

A [Model Context Protocol](https://modelcontextprotocol.io) server that gives
**your own AI** semantic search over the primary literature and regulatory
record for the approved siRNA therapeutics.

> **Your model does the thinking.** This server is *retrieval only* — it returns
> cited source passages from a hosted vector index. No LLM runs on our side; your
> Claude, ChatGPT or other MCP client reasons over what it gets back. Our API keys
> are never used by you, and yours are never seen by us.

**No account, no key, no sign-in.** Paste the URL and connect:

```
https://sirna-atlas.pkpdbuilder.com/api/mcp
```

## What's in the corpus

FDA drug labels + clinical-pharmacology reviews · EMA EPAR assessment reports ·
pivotal & secondary trial publications · foundational siRNA/GalNAc **ADME, PK/PD
& delivery** reviews — for all eight approved drugs:

| Drug | Brand | Indication |
|---|---|---|
| patisiran | Onpattro | hATTR amyloidosis polyneuropathy |
| givosiran | Givlaari | Acute hepatic porphyria |
| lumasiran | Oxlumo | Primary hyperoxaluria type 1 |
| inclisiran | Leqvio | Hypercholesterolemia / ASCVD |
| vutrisiran | Amvuttra | hATTR polyneuropathy; ATTR-CM |
| nedosiran | Rivfloza | Primary hyperoxaluria type 1 |
| fitusiran | Qfitlia | Hemophilia A/B |
| plozasiran | Redemplo (EU) | Familial chylomicronemia syndrome |

Tables in the FDA/EMA PDFs (PK parameters, efficacy endpoints, tox summaries)
were OCR-recovered and preserved as Markdown, so **numeric table values are
searchable** — not just the surrounding prose.

## Tools

| Tool | What it does |
|---|---|
| `search_corpus` | Semantic search → ranked passages with source metadata + chunk ids. Optional drug filter and `topK`. |
| `corpus_info` | The 8 drugs, source types, docType meanings, live vector count. |
| `get_citations` | Chunk ids → APA-formatted references (journal articles resolve fully; FDA/EMA reviews resolve to the agency document). |
| `fetch_document_section` | Widen a hit to its full page/section (e.g. a whole table) when a snippet is cut off. |

## Connect

### Claude Desktop · claude.ai

Settings → Connectors → **Add custom connector**, paste the URL above. Nothing
else to fill in.

### Claude Code

```bash
claude mcp add --transport http --scope user sirna-kb https://sirna-atlas.pkpdbuilder.com/api/mcp
```

`--scope user` makes it available in every project. Drop it to add only to the
current project. Verify with `/mcp` inside Claude Code, or `claude mcp list`.

### Any other MCP client

This repo ships a [`.mcp.json`](./.mcp.json) — clone it, or copy the file into
your project:

```json
{
  "mcpServers": {
    "sirna-kb": {
      "type": "http",
      "url": "https://sirna-atlas.pkpdbuilder.com/api/mcp"
    }
  }
}
```

### Try it

> "Using sirna-kb, what was the mean baseline mNIS+7 for the vutrisiran group in
> HELIOS-A, and cite the source?"

Your model will call `search_corpus`, then `get_citations`, and answer with the
value and an APA reference.

## Prompting your agent

Paste this into the system prompt, project instructions or `CLAUDE.md` of
whatever you connected — it fixes the call order and carries the two rules this
corpus has been burned by:

```
You have access to the "sirna-kb" MCP server: a retrieval-only knowledge base
covering the eight approved siRNA therapeutics. Call corpus_info once to scope
what is answerable. For any factual question about these drugs — mechanism,
ADME, PK/PD, dosing, efficacy, safety, nonclinical toxicology, regulatory
history — search the corpus before answering from your own knowledge, passing
the "drugs" filter when the question names specific drugs. When a snippet is
truncated or part of a table, call fetch_document_section before quoting numbers
from it. Call get_citations for the ids you used and cite them.

- A source about one drug is evidence about that drug only. Never attribute a
  study ID, dose level, NOAEL or finding from one siRNA to another.
- Tables from regulatory PDFs may arrive as flattened text. If you cannot tell
  which value belongs to which dose, species or arm, say so rather than guessing.
- If a search returns nothing for a drug, say the search found nothing — not that
  the data does not exist.
- Antisense oligonucleotides are out of scope; this corpus is siRNA only.
```

## Fair use

Open access is bounded by usage limits rather than credentials: a burst limit per
client, a monthly search allowance per IP, and a ceiling for the server as a
whole. Heavy or automated use can be issued a token (sent as an `Authorization:
Bearer` header, or `?k=` in the URL) for a much larger allowance — contact the
repo owner. Everyone else needs nothing.

## Security

- **Transport:** HTTPS only.
- **Read-only:** the server only searches and reads; there are no write or delete tools.
- **No LLM calls server-side:** this server never invokes Claude or any model — it
  only queries a vector index and returns text.
- **Nothing secret is exposed:** the corpus is public FDA, EMA and PubMed material.
  Access is open because there is nothing here to protect — the limits exist to
  bound hosting cost, not to restrict the content.

## How it works

```
Your MCP client ────HTTPS────▶ /api/mcp (Vercel)
                                  │  search_corpus / get_citations / …
                                  ▼
                          Pinecone vector index
                    (llama-text-embed-v2, integrated inference)
```

Source passages come back to your model, which composes the answer.

There is also a hosted chat UI over the same corpus at
**[sirna-atlas.pkpdbuilder.com](https://sirna-atlas.pkpdbuilder.com)** — answers
with inline citations and an APA reference list.

## License

MIT — see [LICENSE](./LICENSE). The corpus documents are public regulatory and
published-literature sources and remain under their own terms.
