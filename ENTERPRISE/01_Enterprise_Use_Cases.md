# Enterprise Use Cases â€” api-oss-database

## Overview

api-oss-database delivers a complete local-first database platform combining PostgreSQL (relational), pgvector (vector similarity), and SQLite (embedded), replacing Supabase and similar cloud-hosted database services. All data remains on-premises.

---

## Use Case 1: RAG Knowledge Base for Enterprise Search

**Scenario:** A 1,000-employee manufacturing company builds an internal knowledge base over 500,000 technical documents. Semantic search uses pgvector; structured metadata queries use PostgreSQL.

```python
import psycopg2
from aioss import Ledger
import numpy as np

ledger = Ledger.open("./db_ledger.aioss")

conn = psycopg2.connect("postgresql://localhost:5432/enterprise_kb")

def hybrid_search(query_embedding: list, keyword: str, limit: int = 10):
    cur = conn.cursor()
    cur.execute("""
        SELECT doc_id, title, content,
               embedding <=> %s::vector AS vec_dist
        FROM documents
        WHERE to_tsvector('english', content) @@ plainto_tsquery(%s)
        ORDER BY vec_dist LIMIT %s
    """, (query_embedding, keyword, limit))
    results = cur.fetchall()
    ledger.append(
        entry_type="vector_search",
        actor="enterprise-kb",
        content={"tokens_in": len(query_embedding), "tokens_out": len(results),
                 "wall_time_ms": 12, "cost_if_cloud_microcents": 0}
    )
    return results
```

**ROI:** Supabase Pro plan: $25/month base + $0.09/GB storage. At 2TB enterprise scale: ~$200/month. Self-hosted on existing NAS hardware: **$2,400/year saved** plus no data-residency risk.

---

## Use Case 2: Multi-tenant SaaS Database Backend

**Scenario:** ISV replaces hosted Supabase with local api-oss-database for their B2B SaaS product, achieving row-level security isolation per tenant.

```python
# Row-level security per tenant â€” no cloud dependency
def setup_tenant_rls(tenant_id: str):
    with conn.cursor() as cur:
        cur.execute(f"""
            ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;
            CREATE POLICY tenant_isolation ON tenant_data
                USING (tenant_id = current_setting('app.tenant_id')::uuid);
        """)
        conn.commit()
```

**Deployment:**
```
[App Server] â”€â”€â–º [pgBouncer :5432] â”€â”€â–º [PostgreSQL 16 + pgvector]
                                              â”‚
                                    [AIOSS Ledger: DDL/DML audit]
```

**ROI:** Supabase Team plan for 100 tenants: ~$599/month. Self-hosted: $0/month in SaaS fees. **$7,188/year saved**.

---

## Use Case 3: LLM-Assisted Query Generation

**Scenario:** Business intelligence team generates SQL from natural language using a local LLM, with api-oss-database validating and executing queries safely.

```python
from api_oss_database import SafeQueryExecutor
from aioss import Ledger

ledger = Ledger.open("./bi_ledger.aioss")
executor = SafeQueryExecutor(
    model_endpoint="http://localhost:11434/v1",  # local Ollama
    db_conn="postgresql://localhost:5432/analytics",
    ledger=ledger
)

result = executor.natural_language_query(
    "Show me top 10 products by revenue last quarter"
)
# Query validated, parameterised, executed, and logged to AIOSS ledger
```

**ROI:** Cloud NL-to-SQL services: ~$0.002/query Ã— 50,000 queries/month = $100/month. Local inference: $0. **$1,200/year saved** on API fees alone.

---

## Deployment Architecture

```
[Enterprise Network]
â”œâ”€â”€ api-oss-database (PostgreSQL 16 + pgvector)
â”‚   â”œâ”€â”€ Port 5432 (internal only)
â”‚   â””â”€â”€ AIOSS Ledger: ./db_ledger.aioss
â”œâ”€â”€ pgBouncer (connection pool)
â”œâ”€â”€ api-oss-backup (nightly Parquet snapshots)
â””â”€â”€ api-oss-analytics (reads AIOSS ledger for query stats)
```
