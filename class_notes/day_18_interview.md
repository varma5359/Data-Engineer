# Day 18 — Git and Code Organization | Interview

**What is Git?** Distributed version control tracking source changes and history.

**Commit vs push?** Commit records changes locally; push sends commits to a remote repository.

**Why branch?** Isolate features/fixes and support review without destabilizing main.

**What belongs in Git?** Source, tests, documentation, safe configuration templates. Exclude secrets, virtual environments, sensitive/raw datasets, and generated artifacts per policy.

**Pull request?** Review workflow to discuss, validate, and approve changes before merge.

**Scenario: secret committed.** Revoke/rotate immediately, notify security, remove exposure from history as required, and add secret scanning/prevention. Deleting it in a later commit does not make the credential safe.