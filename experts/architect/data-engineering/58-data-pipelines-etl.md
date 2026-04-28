# Data Pipelines and ETL

## Profile

### What It Is
Data Pipelines are automated workflows that move and transform data from source systems to destinations (data warehouses, data lakes, analytics platforms). ETL (Extract, Transform, Load) and ELT (Extract, Load, Transform) are the two primary patterns for data integration and processing.

### Origin and Evolution
- Traditional ETL tools: Informatica, DataStage, Talend (2000s)
- ELT shift with cloud data warehouses (BigQuery, Snowflake, Redshift)
- Apache Airflow (2014, Airbnb) became the standard orchestrator
- Modern: dbt for transformation, Dagster/Prefect as next-gen orchestrators
- Current: streaming pipelines (Kafka + Flink), real-time analytics, data contracts

### Key Authors and References
- **Ralph Kimball** — *The Data Warehouse Toolkit* (dimensional modeling)
- **Maxime Beauchemin** — Apache Airflow creator
- **dbt Labs** — dbt (data build tool) and analytics engineering
- **Martin Kleppmann** — Stream processing in *DDIA*

### Key Concepts at a Glance
| Pattern | Description |
|---------|-------------|
| ETL | Extract → Transform → Load (transform before loading) |
| ELT | Extract → Load → Transform (transform in the warehouse) |
| Batch | Process data in scheduled intervals (hourly, daily) |
| Streaming | Process data continuously as it arrives |
| Orchestration | Managing dependencies, scheduling, retries (Airflow, Dagster) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Idempotent pipelines** — Running a pipeline twice with the same input produces the same output. Essential for retries and backfills.
2. **ELT over ETL for modern stacks** — Load raw data first, transform in the warehouse. Cheaper, more flexible, and preserves raw data.
3. **Observability for data** — Data pipelines need monitoring, alerting, and lineage tracking just like application services.
4. **Data quality is non-negotiable** — Validate data at ingestion and after transformation. Bad data in → bad decisions out.
5. **Incremental over full refresh** — Process only new/changed data when possible. Full refreshes are simple but don't scale.

### When to Use vs. When to Avoid

**Batch pipelines**: Scheduled reports, daily/hourly aggregations, data warehouse loading
**Streaming pipelines**: Real-time dashboards, fraud detection, event-driven analytics
**Simple queries**: If a SQL query against the source database suffices, don't build a pipeline

---

## Frameworks and Methodologies

### 1. ELT Pipeline Design — step-by-step
1. Extract: pull raw data from sources (APIs, databases, files)
2. Load: land raw data in staging area (data lake or warehouse staging schema)
3. Transform: use dbt or SQL to clean, model, and aggregate in the warehouse
4. Test: validate data quality (row counts, null checks, uniqueness)
5. Serve: expose transformed data to BI tools and consumers
6. Monitor: track freshness, volume, and quality metrics

### 2. Orchestration with Airflow — step-by-step
1. Define DAG (Directed Acyclic Graph) of tasks
2. Set schedule (cron expression or data-driven trigger)
3. Implement operators for each step (SQL, Python, API calls)
4. Configure retries and failure alerts
5. Implement idempotency (rerunnable without side effects)
6. Monitor via Airflow UI and custom metrics

---

## How to Apply

### Decision Checklist
- [ ] Is the pipeline idempotent (safe to re-run)?
- [ ] Are data quality checks in place?
- [ ] Is there monitoring for freshness and volume?
- [ ] Is incremental processing used where possible?
- [ ] Are transformations version-controlled (dbt, SQL in Git)?

### Implementation Patterns

**dbt Transformation:**
```sql
-- models/orders/order_summary.sql
{{ config(materialized='incremental', unique_key='order_id') }}

SELECT
    o.id AS order_id,
    o.customer_id,
    c.name AS customer_name,
    COUNT(oi.id) AS item_count,
    SUM(oi.quantity * oi.unit_price) AS total_amount,
    o.created_at
FROM {{ ref('stg_orders') }} o
JOIN {{ ref('stg_customers') }} c ON o.customer_id = c.id
JOIN {{ ref('stg_order_items') }} oi ON o.id = oi.order_id
{% if is_incremental() %}
WHERE o.created_at > (SELECT MAX(created_at) FROM {{ this }})
{% endif %}
GROUP BY o.id, o.customer_id, c.name, o.created_at
```

**Airflow DAG:**
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

with DAG("order_etl", schedule="0 6 * * *",
         start_date=datetime(2024, 1, 1),
         catchup=False, default_args={"retries": 2}) as dag:

    extract = PythonOperator(task_id="extract_orders",
                             python_callable=extract_from_api)
    load = PythonOperator(task_id="load_to_warehouse",
                          python_callable=load_raw_data)
    transform = PythonOperator(task_id="run_dbt_models",
                               python_callable=run_dbt)
    quality = PythonOperator(task_id="data_quality_checks",
                             python_callable=run_quality_tests)

    extract >> load >> transform >> quality
```

### Common Mistakes
1. **Non-idempotent pipelines** — Re-running creates duplicates or incorrect results
2. **No data quality checks** — Bad data silently flows to dashboards and decisions
3. **Over-engineering** — Building Kafka streaming when a nightly batch SQL query suffices
4. **No monitoring** — Pipeline fails silently; stakeholders discover stale data days later
5. **Full refresh at scale** — Reprocessing millions of rows daily when only thousands changed

### Integration with Other Topics
- **Event Streaming** — Kafka as source for streaming pipelines
- **Database Design** — Source schemas affect extraction complexity
- **Data Modeling** — Dimensional modeling for analytics (star schema, snowflake)
- **Observability** — Data pipeline monitoring and alerting
- **Cloud-Native** — Cloud data warehouses enable ELT pattern
- **CI/CD** — dbt models deployed via CI/CD pipelines

---

## Resources

### Essential Reading
- *Fundamentals of Data Engineering* — Joe Reis & Matt Housley
- *The Data Warehouse Toolkit* — Ralph Kimball
- dbt documentation (docs.getdbt.com)

### Online Resources
- Apache Airflow documentation
- Dagster documentation
- Analytics Engineering Guide (dbt Labs)

### Practice Exercises
1. Build an ELT pipeline: extract from API → load to warehouse → transform with dbt
2. Implement data quality checks (row counts, null percentages, uniqueness)
3. Make a pipeline idempotent and test by running it twice
4. Set up Airflow DAG with retry logic and failure alerts
5. Convert a full-refresh pipeline to incremental processing
