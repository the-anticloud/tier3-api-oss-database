# L5 Narrow / L2 General Classification — api-oss-database
**Platform:** Anticloud | **PAX:** 27B | **IP:** USPTO pending 2026, Anticloud FZ LLE
**Domain:** Sovereign database layer: encrypted SQLite with full-text search and AIOSS audit

## L5 Narrow
api-oss-database provides the encrypted persistent storage layer for all Anticloud state that needs relational structure. SQLCipher-encrypted SQLite means every database file is AES-256 encrypted at rest. FTS5 full-text search enables fast AIOSS chain queries and knowledge base lookups.

## L2 General
L2 General: all 9 tiers use api-oss-database for structured storage. Clinical records, robot mission logs, RF packet metadata, and conversation history all go through the same encrypted database API.

## PAX Integration
PAX 27B is invoked for natural language database queries: 'Show me all AIOSS chain entries from the last 24 hours where latency exceeded 1000ms' is translated to SQL by PAX.

## AIOSS Audit Relevance
Every database operation (query hash + affected rows hash + execution plan hash) is AIOSS-chained: H_n = SHA3-256(H_{n-1} || entry_hash_n || timestamp_n).
Tamper-evident, offline-verifiable, zero cloud dependency.

## Regulatory / Compliance
GDPR Art. 32 (storage encryption), HIPAA 45 CFR 164.312 (encryption at rest), FIPS 140-2
