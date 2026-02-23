# AI/ML Architecture

## Profile

### What It Is
AI/ML Architecture encompasses the patterns, infrastructure, and design decisions for building systems that incorporate machine learning — from model training pipelines and serving infrastructure to feature stores, experiment tracking, and MLOps practices that bridge data science and production engineering.

### Origin and Evolution
- Statistical models in production (2000s) — batch scoring, manual deployment
- Hadoop/Spark era (2010s) — large-scale data processing for ML
- Deep learning revolution (2012+) — GPU infrastructure, TensorFlow, PyTorch
- MLOps emergence (2018+) — CI/CD for ML, experiment tracking, model registries
- Current: LLMOps, foundation models, RAG architectures, vector databases, AI agents

### Key Authors and References
- **Martin Fowler & ThoughtWorks** — CD4ML (Continuous Delivery for Machine Learning)
- **Chip Huyen** — *Designing Machine Learning Systems* (2022)
- **Google** — MLOps whitepaper, TFX pipeline architecture
- **Andrej Karpathy** — Software 2.0 thesis

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| ML Pipeline | End-to-end flow: data → features → training → evaluation → deployment |
| Feature Store | Centralized repository of curated features for training and serving |
| Model Registry | Versioned storage of trained models with metadata and lineage |
| Model Serving | Infrastructure to serve predictions (batch, real-time, streaming) |
| Experiment Tracking | Recording hyperparameters, metrics, and artifacts for reproducibility |
| MLOps | DevOps practices applied to ML: CI/CD, monitoring, automation |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Data is the architecture** — ML systems are data systems first. The quality, freshness, and accessibility of data determine model quality more than algorithm choice.
2. **Reproducibility is non-negotiable** — Every experiment, training run, and prediction must be reproducible. Version data, code, config, and environment.
3. **Training-serving skew is the #1 bug** — The feature computation in training must match serving exactly. Feature stores and shared pipelines prevent this.
4. **Monitor models, not just infrastructure** — Model performance degrades over time (data drift, concept drift). Monitor prediction quality, not just latency and errors.
5. **Separate experimentation from production** — Data scientists need freedom to experiment. Production needs stability. Bridge the gap with MLOps automation.

### When to Use vs. When to Avoid

**Invest in ML architecture when:**
- Multiple models in production with regular retraining
- Team has both data scientists and ML engineers
- Feature reuse across models provides value
- Model performance directly impacts business metrics

**Keep it simple when:**
- One or two models, infrequently updated
- Small team where one person handles end-to-end
- Rule-based systems solve the problem adequately
- Using pre-trained models/APIs without custom training

---

## Frameworks and Methodologies

### 1. ML Platform Design — step-by-step
1. Define ML use cases and their serving requirements (batch vs. real-time)
2. Build or adopt a feature store (Feast, Tecton, or custom)
3. Set up experiment tracking (MLflow, Weights & Biases, Neptune)
4. Create training pipelines (Kubeflow, Airflow, Vertex AI)
5. Implement model registry with versioning and approval workflow
6. Deploy model serving infrastructure (TorchServe, Triton, Seldon, custom)
7. Add model monitoring for drift detection and performance tracking

### 2. RAG Architecture Design — step-by-step
1. Define knowledge sources (documents, databases, APIs)
2. Build ingestion pipeline: parse, chunk, embed documents
3. Choose vector database (Pinecone, Weaviate, pgvector, Qdrant)
4. Design retrieval strategy: semantic search, hybrid search, reranking
5. Implement prompt construction: system prompt + retrieved context + user query
6. Add evaluation pipeline: relevance, faithfulness, answer quality metrics
7. Monitor: retrieval quality, latency, token usage, hallucination rate

---

## How to Apply

### Decision Checklist
- [ ] Is there a clear separation between experimentation and production?
- [ ] Are features computed consistently between training and serving?
- [ ] Are models versioned with lineage tracking?
- [ ] Is model performance monitored for drift?
- [ ] Is the retraining pipeline automated and reproducible?

### Implementation Patterns

**Feature Store Pattern:**
```python
from feast import FeatureStore, Entity, FeatureView, Field
from feast.types import Float32, Int64

# Define entity
customer = Entity(name="customer", join_keys=["customer_id"])

# Define feature view
customer_features = FeatureView(
    name="customer_features",
    entities=[customer],
    schema=[
        Field(name="total_orders", dtype=Int64),
        Field(name="avg_order_value", dtype=Float32),
        Field(name="days_since_last_order", dtype=Int64),
    ],
    source=customer_feature_source,  # BigQuery, Redshift, etc.
    ttl=timedelta(days=1),
)

# Training: get historical features
training_df = store.get_historical_features(
    entity_df=entity_df,  # customer_id + event_timestamp
    features=["customer_features:total_orders",
              "customer_features:avg_order_value"],
).to_df()

# Serving: get online features (same computation, no skew)
features = store.get_online_features(
    features=["customer_features:total_orders",
              "customer_features:avg_order_value"],
    entity_rows=[{"customer_id": "C123"}],
).to_dict()
```

**Model Serving with FastAPI:**
```python
import mlflow
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()
model = mlflow.pyfunc.load_model("models:/fraud_detector/Production")

class PredictionRequest(BaseModel):
    customer_id: str
    transaction_amount: float
    merchant_category: str

class PredictionResponse(BaseModel):
    fraud_probability: float
    is_fraud: bool
    model_version: str

@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    features = feature_store.get_online_features(
        entity_rows=[{"customer_id": request.customer_id}],
    )
    input_df = prepare_input(request, features)
    prediction = model.predict(input_df)

    return PredictionResponse(
        fraud_probability=float(prediction[0]),
        is_fraud=prediction[0] > 0.5,
        model_version=model.metadata.run_id,
    )
```

**RAG Pipeline:**
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Qdrant

# Ingestion
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(documents)

vectorstore = Qdrant.from_documents(
    chunks,
    embedding=OpenAIEmbeddings(),
    collection_name="knowledge_base",
)

# Retrieval + Generation
def answer_question(query: str) -> str:
    # Retrieve relevant chunks
    docs = vectorstore.similarity_search(query, k=5)
    context = "\n\n".join(doc.page_content for doc in docs)

    # Generate answer with context
    response = llm.invoke(
        f"Context:\n{context}\n\nQuestion: {query}\n\n"
        f"Answer based on the context above. "
        f"If the context doesn't contain the answer, say so."
    )
    return response
```

### Common Mistakes
1. **No feature store** — Features computed differently in training notebooks vs. serving code; training-serving skew
2. **Manual model deployment** — Copy-pasting model files; no versioning, no rollback capability
3. **No model monitoring** — Model deployed and forgotten; performance degrades silently as data distribution shifts
4. **Overengineering for one model** — Building a full ML platform for a single model; start simple, add infrastructure as complexity grows
5. **Ignoring data quality** — Focusing on model architecture while feeding it noisy, biased, or stale data

### Integration with Other Topics
- **Data Pipelines** — ML pipelines are specialized data pipelines with training and evaluation stages
- **Event Streaming** — Real-time feature computation from event streams
- **API Design** — Model serving exposes predictions via APIs
- **Observability** — Model monitoring extends traditional observability with ML-specific metrics
- **CI/CD** — MLOps extends CI/CD with model validation, A/B testing, and gradual rollout
- **Cloud-Native** — ML workloads leverage GPU instances, managed services, and auto-scaling

---

## Resources

### Essential Reading
- *Designing Machine Learning Systems* — Chip Huyen
- *Machine Learning Engineering* — Andriy Burkov
- Google's MLOps whitepaper (cloud.google.com)

### Online Resources
- MLOps Community (mlops.community)
- Made With ML (madewithml.com) — MLOps course
- Full Stack Deep Learning (fullstackdeeplearning.com)

### Practice Exercises
1. Build an ML pipeline with experiment tracking (MLflow or W&B)
2. Implement a feature store with Feast for training and serving
3. Deploy a model with FastAPI and add monitoring for data drift
4. Build a RAG pipeline with vector database and evaluation metrics
5. Set up automated retraining triggered by model performance degradation
