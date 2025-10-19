# What's New

This lightweight changelog tracks notable documentation and API updates that affect integrators and LLM assistants.

## 2025-02-11

- Reworked stage documentation with syntax + before/after examples (`$match`, `$project`, `$limit`, `$skip`, `$sortByCount`, `$sampleRate`, `$densify`, `$fill`, `$setWindowFields`).
- Documented unsupported stages (`$merge`, `$out`, `$search`, `$vectorSearch`) so assistants respond with the correct guidance.
- Expanded the usage and integration guides to cover caching parsed pipelines, error handling, and registry patterns.
- Added enrichment and connector status tables to flag beta/experimental APIs, plus concrete HTTP/SQL examples.
- Introduced LLM assistant notes and glossary clarifications to emphasise Fluxion's single-document focus.

---

Earlier history lives in commit messages while the project stabilises. Significant releases will be summarised here as they ship.
