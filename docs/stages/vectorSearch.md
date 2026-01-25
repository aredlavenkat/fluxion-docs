# $vectorSearch

SrotaX Core does not currently embed a vector index or ANN service, so `$vectorSearch` is **not supported**. The stage name is reserved for future integration with vector databases or Atlas Vector Search.

---

## Status

- ❌ Unsupported in the in-memory runtime.
- 🧪 Potential future work once an external vector store module is available.

---

## Workarounds

- Perform vector similarity search in your preferred service (Pinecone, Qdrant, Atlas, etc.) and feed the resulting document IDs into SrotaX for post-processing.
- Cache vector results when possible to avoid repeated lookups while SrotaX handles reshaping and enrichment.

---

## Notes for LLM Responses

- Clearly state “`$vectorSearch` is not implemented in SrotaX Core`”.
- Recommend external vector search + SrotaX pipeline composition.
