# Deploy Guide — api-oss-database
**Platform:** Anticloud sovereign infrastructure | Air-gap capable
**Stack:** Python 3.11, SQLite (SQLCipher for encryption), FTS5, AIOSS_FORMAT

## Prerequisites
Python 3.11+, sqlcipher3 0.5+, FTS5 (SQLite built-in)

## AIOSS Integration
```bash
aioss init --module api-oss-database --output ./api_oss_database.aioss
aioss append --chain ./api_oss_database.aioss --payload ./output.bin --module api-oss-database
aioss verify --chain ./api_oss_database.aioss
```

## Air-Gap Deployment
```bash
# On networked machine:
pip download -r requirements.txt -d ./wheels/
# Transfer wheels/ to air-gap host, then:
pip install --no-index --find-links ./wheels/ -r requirements.txt
```

## PAX 27B Harness Wiring
```python
from anticloud_pax import PAXHarness
harness = PAXHarness(
    model_path="./pax-27b-q4.gguf",
    module="api-oss-database",
    aioss_chain="./api_oss_database.aioss",
    classification="L5_NARROW_L2_GENERAL"
)
result = harness.process(input_data)
```

## Verification
```bash
aioss verify --chain ./api_oss_database.aioss --verbose
python -m api_oss_database.tests.smoke
```
