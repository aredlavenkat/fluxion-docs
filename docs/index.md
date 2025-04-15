# 🚀 Fluxion Aggregator Documentation

Welcome to the official developer documentation for the **Fluxion Aggregator Framework** — a high-performance rule engine and aggregation system inspired by MongoDB, optimized for complex nested pipelines and ecommerce use cases.

---

## 📦 What is Fluxion?

Fluxion is a MongoDB-like aggregation engine built for structured JSON documents. It supports:

- 40+ Aggregation **Stages**
- 200+ Expression **Operators**
- System Variables (`$$ROOT`, `$$CURRENT`, `$$NOW`, `$$REMOVE`, etc.)
- Deeply **Nested Pipelines**
- Advanced support for `$facet`, `$group`, `$bucket`, `$map`, `$filter`, `$reduce`, `$switch`, `$expr`, `$lookup`, and more.

---

## 🧭 Quick Navigation

- 📘 [Usage Guide](usage.md)
- 📊 [Stages Overview](stages/)
- 🧮 [Operators Reference](operators/)
- 🛒 [Ecommerce Examples](examples/examples_gallery.md)
- 🧠 [Glossary of Terms](glossary.md)
- 🗺️ [Roadmap & Release Phases](roadmap.md)

---

## 🛠️ Example Pipeline

```json
[
  { "$match": { "status": "active" } },
  { "$unwind": "$items" },
  { "$group": {
      "_id": "$items.category",
      "total": { "$sum": "$items.price" }
  }},
  { "$sort": { "total": -1 } }
]
```

---

## 🎯 Why Use Fluxion?

✅ Familiar MongoDB-style syntax  
✅ Works on any Python backend  
✅ Supports system variables and expressions  
✅ Ideal for ecommerce & analytics pipelines  
✅ Fully unit-tested and modular

---

Start exploring from the sidebar or open the [Usage Guide](usage.md) to begin!
