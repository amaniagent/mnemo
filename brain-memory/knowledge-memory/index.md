# Knowledge Memory Index

| Path | Size | Description | Updated |
|------|------|-------------|---------|
| repo/ | — | Source code repositories (Gortex indexed) | — |
| wiki/ | — | LLM-maintained knowledge wiki | — |
| rag/ | — | Graph-RAG indices (LightRAG, RAG Anything) | — |

## Repositories (repo/)
Managed by **Gortex** — each repo gets:
- `index.json` — Gortex tree-index
- `<name>-xs.md` … `<name>-xl.md` — Doc-size tiered exports
- `raw/` — Original files (optional reference)

## Wiki (wiki/)
Managed by **LLM-Wiki (Karpathy Pattern)** — each topic gets:
- `index.md` — Wiki entry point
- `CLAUDE.md` — LLM maintenance schema
- `<topic>-xs.md` … `<topic>-xl.md` — Tiered summaries

## RAG (rag/)
Managed by **LightRAG** + **RAG Anything** — each domain gets:
- Graph index + vector store
- Query API endpoint