# siRNA Therapeutics Knowledge Base — MCP server

A [Model Context Protocol](https://modelcontextprotocol.io) server that gives
**your own Claude Code** semantic search over the primary literature and
regulatory record for the FDA-approved siRNA therapeutics.

> **Your Claude does the thinking.** This server is *retrieval only* — it returns
> cited source passages from a hosted vector index. No LLM runs on our side; your
> Claude Code (or any MCP client) reasons over what it gets back. Our API keys are
> never used by you, and yours are never seen by us.

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
| plozasiran | — | Familial chylomicronemia syndrome |

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

## Enable it in Claude Code

You need an **access token** — [request one](#getting-a-token) (the corpus data is
public, but access to the hosted index is gated to control cost/abuse).

### Option A — one command (quickest)

```bash
claude mcp add --transport http --scope user sirna-kb \
  https://sirna-adme-app.vercel.app/api/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"
```

`--scope user` makes it available in every project. Drop it to add only to the
current project.

### Option B — clone this repo (project-scoped, sharable)

This repo ships a [`.mcp.json`](./.mcp.json). Clone it (or copy that file into your
project), set your token as an env var, and open the folder in Claude Code — it
detects the config and prompts you to approve the server.

```bash
git clone https://github.com/husain-clintel/sirna-kb-mcp.git
cd sirna-kb-mcp
export SIRNA_KB_TOKEN="YOUR_TOKEN"   # Claude Code expands ${SIRNA_KB_TOKEN}
claude                                # approve "sirna-kb" when prompted
```

Verify with `/mcp` inside Claude Code, or `claude mcp list`.

### Try it

> "Using sirna-kb, what was the mean baseline mNIS+7 for the vutrisiran group in
> HELIOS-A, and cite the source?"

Claude will call `search_corpus`, then `get_citations`, and answer with the value
and an APA reference.

## Security

- **Transport:** HTTPS only. Your token rides in the `Authorization` header.
- **Your token is a secret.** Don't commit it. The committed `.mcp.json` reads it
  from the `${SIRNA_KB_TOKEN}` environment variable — never hard-code it there.
- **Revocable:** each token is issued per user and can be revoked server-side
  without affecting anyone else.
- **Rate-limited** per token.
- **Read-only:** the server only searches and reads; there are no write/delete tools.
- **No LLM calls server-side:** this server never invokes Claude or any model — it
  only queries a vector index and returns text.

## Getting a token

Contact the maintainer (see repo owner) to be issued a token. Tokens are minted
with `scripts/mint-mcp-token.ts` in the server project; only the hash is stored
server-side.

## How it works

```
Your Claude Code ──HTTPS+Bearer──▶ /api/mcp (Vercel)
                                      │  search_corpus / get_citations / …
                                      ▼
                              Pinecone vector index
                        (llama-text-embed-v2, integrated inference)
```

Source passages come back to your Claude, which composes the answer. The corpus is
public FDA/EMA/PubMed material; the token gates access to the hosted index only.

## License

MIT — see [LICENSE](./LICENSE). The corpus documents are public regulatory and
published-literature sources and remain under their own terms.
