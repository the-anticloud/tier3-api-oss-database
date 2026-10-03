# Developer Cookbook — api-oss-database
**Stack:** Python 3.11, SQLite (SQLCipher for encryption), FTS5, AIOSS_FORMAT
**Domain:** Sovereign database layer: encrypted SQLite with full-text search and AIOSS audit
**License:** Apache-2.0 | **IP:** USPTO pending 2026, Anticloud FZ LLE

## Core Usage

```python
from api_oss_database import SovereignDB

db = SovereignDB("./anticloud.db", password=vault.retrieve("db_key"),
                 aioss_chain="./db.aioss")

# Create table
db.execute('''CREATE TABLE IF NOT EXISTS inference_log
              (id INTEGER PRIMARY KEY, timestamp INTEGER, module TEXT,
               prompt_hash TEXT, response_hash TEXT, latency_ms REAL)''')

# Insert
db.execute("INSERT INTO inference_log VALUES (?,?,?,?,?,?)",
           (None, int(time.time()), "PAX_INFERENCE_CORE", prompt_hash, resp_hash, 508.3))

# Full-text search
results = db.fts_search("inference_log", "PAX_INFERENCE_CORE HIPAA")

# NL query via PAX
sql = db.nl_to_sql("Show entries where latency > 1000ms in the last hour",
                   table="inference_log", pax_model="./pax-27b-q4.gguf")
rows = db.execute(sql)
```

## AIOSS Chain Append

```python
import hashlib, time

def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()

# After every api-oss-database output:
chain_hash = aioss_append("./api_oss_database.aioss",
                           result_bytes, "api-oss-database")
```

## Performance & Integration

SQLCipher: AES-256-CBC, PBKDF2 key derivation (64000 iterations). WAL journal mode. FTS5 trigram tokenizer for substring search. Connection pool size = CPU cores. Integration: storage layer for KANTOR_K5, MIIRAI_CHAT, api-oss-logging, api-oss-analytics.
