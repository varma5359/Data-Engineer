# Day 9 — Data Formats | Interview

**CSV vs JSON?** CSV is flat and compact for tabular exchange; JSON supports nested records and is common for APIs.

**Why are CSV values often strings?** CSV has no reliable universal type metadata; conversion must follow an explicit schema.

**load vs loads?** load reads from a file-like object; loads parses a string.

**Why Parquet?** Columnar layout, compression, schema metadata, and efficient column reads make it suitable for analytics.

**When XML?** Existing enterprise interfaces or systems requiring tag-based hierarchical documents.

Scenario: API JSON contains optional nested customer fields. Define a mapping, handle missing keys safely, validate types, and monitor schema drift. Do not assume every record is identical.