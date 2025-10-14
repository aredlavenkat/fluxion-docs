
## 🚀 Live Site

You can browse the full documentation here:  
👉 [https://aredlavenkat.github.io/fluxion-docs/](https://aredlavenkat.github.io/fluxion-docs/)

---

## 📚 What's Inside?

- 200+ Operators documented with syntax and examples
- 40+ Aggregation Stages fully supported
- Deep nested pipeline examples with input/output JSON
- Ecommerce-specific test cases and visual guides
- MkDocs-ready structure for easy deployment

---

## 🛠️ Getting Started Locally

```bash
pip install mkdocs mkdocs-material
mkdocs serve
```

Open http://127.0.0.1:8000 to preview the site with live reload.

---

## 📁 Repository Layout

- `docs/index.md` – landing page
- `docs/core/` – aggregation engine and runtime details
- `docs/connect/` – connector SPI documentation and per-connector guides
- `docs/enrich/` – enrichment operators and integration recipes
- `docs/examples/` – end-to-end pipeline walkthroughs
- `mkdocs.yml` – site navigation configuration

---

## 🤝 Contributing

- Keep doc updates alongside the matching code changes from `fluxion-core-engine-java`.
- Run `mkdocs serve` (or `mkdocs build`) to catch broken links before opening a PR.
- When adding new operators, stages, or connectors, use the existing templates so navigation remains consistent.

Happy documenting! 🚀
