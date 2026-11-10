# Day 18: Git and Code Organization

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Version-Controlled ETL Project  
**Learning time:** 2–3 hours

## 1. Learning outcomes
- Explain version control and Git.
- Understand repository, commit, branch and remote.
- Use essential Git commands.
- Create feature branches and merge changes.
- Resolve simple merge conflicts.
- Organize a small ETL project.
- Protect secrets and generated files.

## 2. Why version control?
Git records code history and supports collaboration. Engineers can review changes, restore earlier versions, work independently and connect code changes to issues and deployments.

Git is distributed: each clone contains repository history, while GitHub or another hosting service commonly provides a shared remote.

## 3. Key concepts

| Term | Meaning |
|---|---|
| Repository | Project plus version history |
| Working tree | Current checked-out files |
| Staging area | Changes selected for next commit |
| Commit | Snapshot with message and metadata |
| Branch | Movable line of development |
| Merge | Combine histories/changes |
| Remote | Hosted repository reference |
| Clone | Create a local copy |
| Pull | Fetch and integrate remote changes |
| Push | Publish local commits |

## 4. Configure Git
```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Use the email associated with your Git identity. Do not put access tokens in commands that may be saved in shell history.

## 5. Initialize or clone
Create a repository:

```bash
git init
```

Clone an existing repository:

```bash
git clone https://github.com/owner/repository.git
cd repository
```

## 6. Inspect repository state
```bash
git status
git log --oneline
git remote -v
```

Check status before staging, committing, pulling or pushing. Understand which branch and remote are active.

## 7. Stage and commit
```bash
git add transformations.py
git status
git commit -m "Add order transformation logic"
```

Stage only intended files. Avoid blindly using `git add .` when notebooks, data exports or secrets may be present.

A useful commit message states the change, not a vague phrase such as "update".

## 8. Branch workflow
```bash
git switch -c feature/order-validation
```

Make and test changes, then commit. Switch back and merge:

```bash
git switch main
git pull
git merge feature/order-validation
```

Push branch:

```bash
git push -u origin feature/order-validation
```

Exact team workflow may use pull requests and protected branches. Do not push directly to a protected production branch unless policy allows it.

## 9. Merge conflicts
A conflict occurs when Git cannot automatically combine overlapping changes.

Workflow:
1. Run `git status`.
2. Open each conflicted file.
3. Review both versions and understand intent.
4. Edit the final desired content.
5. Remove conflict markers.
6. Run tests.
7. Stage resolved files.
8. Complete the merge commit if required.

Never resolve a conflict by blindly choosing "ours" or "theirs" without reviewing the logic.

## 10. Undo carefully
- `git restore file.py`: discard unstaged changes to a file.
- `git restore --staged file.py`: unstage a file.
- `git revert <commit>`: create a new commit that reverses a prior commit.
- `git reset`: move branch history; use carefully, especially if changes are shared.

Before destructive commands, inspect status and ensure important work is backed up.

## 11. .gitignore
Example:

```gitignore
.venv/
__pycache__/
*.pyc
.env
*.log
data/raw/
.ipynb_checkpoints/
```

Do not ignore source code or required small test fixtures. Raw datasets may be excluded due to size, licensing or privacy. Track a README describing how to obtain permitted data.

If a secret was committed, deleting it in a later commit is not enough. Revoke/rotate the credential and follow repository incident procedures.

## 12. Organize an ETL project
```text
version_controlled_etl/
    README.md
    requirements.txt
    .gitignore
    config/
    data/
        raw/
        processed/
    src/
        extract.py
        transform.py
        load.py
    tests/
        test_transform.py
    main.py
```

Keep responsibilities clear:
- extract: obtain source data.
- transform: apply deterministic business rules.
- load: write to destination.
- tests: verify transformations and validation.
- config: non-secret configuration.
- README: setup, run instructions and assumptions.

Avoid unnecessary abstractions. A junior project should be understandable and testable.

## 13. requirements.txt
Create a dependency file:

```text
pandas==2.2.3
requests==2.32.3
```

Install:

```bash
python -m pip install -r requirements.txt
```

Pin or constrain dependencies according to the project's reproducibility policy. Validate versions against the environment before committing.

## 14. Mini-project: Version-Controlled ETL Project
1. Create the project structure.
2. Add a README with purpose, setup and execution steps.
3. Add a small transformation module.
4. Add unit tests.
5. Add .gitignore.
6. Initialize Git and commit the baseline.
7. Create a feature branch.
8. Add a validation rule and tests.
9. Commit the feature.
10. Merge through the team's approved process.
11. Push and verify remote history.

Useful commands:

```bash
git status
git diff
git add src/transform.py tests/test_transform.py
git commit -m "Add order validation"
git log --oneline --graph
git push
```

## 15. Common mistakes
| Mistake | Better practice |
|---|---|
| Commit secrets | Use secret storage and .gitignore |
| Commit huge raw data | Store data externally and document retrieval |
| One giant commit | Make small logical commits |
| Work directly on shared main | Use feature branches and review |
| Ignore failing tests | Fix or explain before merge |
| Force push shared branch | Coordinate and follow team policy |
| Commit generated files | Ignore caches and build artifacts |

## 16. Day 18 revision
- Git tracks project history.
- Staging selects what enters a commit.
- Branches isolate changes.
- Pull requests support review.
- Conflicts require understanding both changes.
- .gitignore reduces accidental commits but does not remove already tracked files.
- Secrets must be revoked if exposed.
