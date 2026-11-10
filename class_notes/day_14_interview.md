# Day 14 — Pandas | Interview

**What is a DataFrame?** Two-dimensional labeled tabular structure with columns that can have different types.

**Why inspect before cleaning?** Understand schema, missing values, duplicates, distributions, and unexpected types before deciding transformations.

**loc vs iloc?** loc selects by labels/conditions; iloc selects by integer position.

**Why validate merge cardinality?** Unexpected many-to-many joins can multiply rows and inflate metrics. Use keys and merge validation.

**How handle missing values?** Understand cause and business meaning; fill, reject, preserve, or impute according to explicit rules.

**When not Pandas?** Data exceeds memory, distributed processing is needed, or database-native operations are more efficient.

Scenario: revenue doubles after join. Check duplicate dimension keys, join type, key nulls, and row counts before/after merge.