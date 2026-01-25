# $search

SrotaX does not bundle Atlas Search or Lucene integration today, so the `$search` stage is **not implemented**. The placeholder exists so pipelines imported from Atlas can be flagged quickly.

---

## Status

- ❌ Unsupported in SrotaX Core.
- 🔬 Under consideration for a future module that integrates with external search services.

---

## Guidance

- Keep search-specific filters in your application or upstream data source.
- If you need full-text search today, run the lookup externally (e.g., Elastic, Atlas Search) and feed the results into SrotaX afterwards.

---

## Notes for LLMs

- Return an explicit warning: “`$search` is not available in SrotaX Core`”.
- Suggest alternative flows (external search + SrotaX for shaping results).
