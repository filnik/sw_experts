# Search Engine Architecture

## Profile

### What It Is
Search Engine Architecture encompasses the design and implementation of systems that enable fast, relevant full-text search, faceted navigation, and complex queries over large datasets. It involves inverted indexes, text analysis pipelines, relevance scoring, and distributed search infrastructure.

### Origin and Evolution
- Apache Lucene (1999) — foundational full-text search library
- Apache Solr (2004) — enterprise search server built on Lucene
- Elasticsearch (2010) — distributed, REST-based search engine on Lucene
- Modern: Meilisearch, Typesense (simpler alternatives), OpenSearch (AWS fork)
- Vector search for AI/ML: semantic search, embeddings, hybrid search

### Key Authors and References
- **Doug Cutting** — Apache Lucene creator
- **Shay Banon** — Elasticsearch creator
- **Clinton Gormley & Zachary Tong** — *Elasticsearch: The Definitive Guide*
- **Grant Ingersoll** — *Taming Text*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Inverted Index | Maps terms to documents containing them (reverse lookup) |
| Analyzer | Pipeline: tokenizer + filters (lowercase, stemming, synonyms) |
| Relevance Scoring | TF-IDF, BM25 — ranking documents by query relevance |
| Faceted Search | Aggregations for filtering (category counts, price ranges) |
| Sharding | Distributing index across nodes for scale |
| Vector Search | Similarity search using embedding vectors (k-NN) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Search is a separate concern** — Don't force your primary database to do full-text search. Use a dedicated search engine alongside your database.
2. **Relevance is tunable, not automatic** — Default scoring works for simple cases. Real-world search requires boosting, synonyms, and domain-specific tuning.
3. **Indexing and querying are separate pipelines** — Data flows from source → analysis → index. Queries go through analysis → index lookup → scoring → results.
4. **Keep the search index in sync** — The source of truth is the database. The search index is a derived, eventually consistent view.
5. **Start simple, add complexity as needed** — Begin with basic full-text search. Add facets, autocomplete, synonyms, and machine-learned ranking as requirements demand.

### When to Use vs. When to Avoid

**Use a search engine when:**
- Full-text search across large text fields
- Faceted navigation (filter by category, price range, attributes)
- Autocomplete and search-as-you-type
- Log aggregation and analytics (ELK stack)
- Relevance-ranked results are needed

**Avoid when:**
- Simple LIKE queries on small datasets suffice
- The database has adequate full-text search (PostgreSQL's tsvector)
- Exact-match lookups only (use the primary database)
- The operational complexity isn't justified

---

## Frameworks and Methodologies

### 1. Search Implementation — step-by-step
1. Define search requirements: what fields, what queries, what facets
2. Design the index mapping (field types, analyzers)
3. Build the indexing pipeline (database → transformation → search engine)
4. Implement sync mechanism (event-driven or periodic)
5. Build the query layer (search API, autocomplete, facets)
6. Tune relevance (boosting, synonyms, custom scoring)
7. Monitor search performance and relevance quality

### 2. Index Design — step-by-step
1. Define one index per search context (products, articles, users)
2. Choose field types: text (analyzed for search), keyword (exact match), numeric
3. Configure analyzers per language and field type
4. Set up multi-field mappings (same field as text + keyword)
5. Define shard count based on data volume and query load
6. Plan index lifecycle (rolling indexes for time-series data)

---

## How to Apply

### Decision Checklist
- [ ] Is there a clear sync strategy between database and search index?
- [ ] Are index mappings designed for known query patterns?
- [ ] Is relevance tuning planned (not just default BM25)?
- [ ] Are facets/aggregations designed for the UI filter needs?
- [ ] Is the search infrastructure monitored (index size, query latency)?

### Implementation Patterns

**Elasticsearch Index Mapping:**
```json
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "english",
        "fields": {
          "keyword": { "type": "keyword" },
          "autocomplete": {
            "type": "text",
            "analyzer": "autocomplete_analyzer"
          }
        }
      },
      "description": { "type": "text", "analyzer": "english" },
      "category": { "type": "keyword" },
      "price": { "type": "float" },
      "tags": { "type": "keyword" },
      "created_at": { "type": "date" }
    }
  },
  "settings": {
    "analysis": {
      "analyzer": {
        "autocomplete_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "autocomplete_filter"]
        }
      },
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 15
        }
      }
    }
  }
}
```

**Search Query with Facets:**
```python
def search_products(query: str, filters: dict, page: int = 1) -> SearchResult:
    body = {
        "query": {
            "bool": {
                "must": [
                    {"multi_match": {
                        "query": query,
                        "fields": ["title^3", "description", "tags^2"],
                        "type": "best_fields",
                        "fuzziness": "AUTO"
                    }}
                ],
                "filter": build_filters(filters)
            }
        },
        "aggs": {
            "categories": {"terms": {"field": "category", "size": 20}},
            "price_ranges": {
                "range": {
                    "field": "price",
                    "ranges": [
                        {"to": 25}, {"from": 25, "to": 50},
                        {"from": 50, "to": 100}, {"from": 100}
                    ]
                }
            }
        },
        "from": (page - 1) * 20,
        "size": 20,
        "highlight": {"fields": {"title": {}, "description": {}}}
    }
    return es.search(index="products", body=body)
```

**Index Sync via Events:**
```python
class SearchIndexer:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def on_product_created(self, event: ProductCreated) -> None:
        self.es.index(index="products", id=event.product_id, document={
            "title": event.title,
            "description": event.description,
            "category": event.category,
            "price": event.price,
            "tags": event.tags,
            "created_at": event.created_at.isoformat(),
        })

    def on_product_updated(self, event: ProductUpdated) -> None:
        self.es.update(index="products", id=event.product_id, doc={
            "title": event.title,
            "price": event.price,
        })

    def on_product_deleted(self, event: ProductDeleted) -> None:
        self.es.delete(index="products", id=event.product_id)
```

### Common Mistakes
1. **Search engine as source of truth** — The database is the source of truth; the search index is a derived view
2. **No sync strategy** — Index gets out of date because there's no reliable sync mechanism
3. **Default relevance forever** — BM25 defaults work poorly for domain-specific search; invest in tuning
4. **Over-sharding** — Too many shards for small indexes adds overhead; start with fewer shards
5. **Mapping explosion** — Dynamic mapping creates too many fields; define mappings explicitly
6. **Not handling partial failures** — Search being unavailable should degrade gracefully, not crash the app

### Integration with Other Topics
- **NoSQL and Polyglot Persistence** — Search engine as part of polyglot strategy
- **Event-Driven Architecture** — Events keep search index in sync with database
- **Caching Strategies** — Cache frequent search results to reduce load
- **System Design** — Search is a core component in most user-facing systems
- **Data Modeling** — Search index schema is query-driven, not entity-driven
- **AI/ML Architecture** — Vector search enables semantic/AI-powered search

---

## Resources

### Essential Reading
- *Elasticsearch: The Definitive Guide* — Gormley & Tong
- *Relevant Search* — Doug Turnbull & John Berryman
- *Taming Text* — Grant Ingersoll

### Online Resources
- Elasticsearch official documentation
- Meilisearch documentation (simpler alternative)
- Lucene in Action concepts

### Practice Exercises
1. Set up Elasticsearch and index a product catalog with custom analyzers
2. Implement search with facets for category and price range filtering
3. Build an autocomplete feature using edge ngrams
4. Sync a PostgreSQL table to Elasticsearch via events
5. Tune relevance scoring with field boosting and custom analyzers
