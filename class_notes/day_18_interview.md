# Day 18 Interview Preparation: Git and Code Organization

**Mini-project:** Version-Controlled ETL Project

## 1. Core questions

1. **What is Git?** A distributed version-control system for tracking project history.
2. **What is a commit?** A recorded snapshot of staged changes.
3. **What is a branch?** An independent line of development.
4. **What is a remote?** A named reference to a hosted repository.
5. **Why use pull requests?** To review, discuss and validate changes before integration.
6. **What is .gitignore?** A file specifying untracked paths Git should ignore.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Git vs GitHub | Version-control tool vs hosted collaboration platform |
| add vs commit | Stage changes vs record staged snapshot |
| fetch vs pull | Download remote updates vs fetch and integrate |
| merge vs rebase | Combine histories vs replay commits onto another base |
| revert vs reset | New reversing commit vs move branch pointer/change working state |
| tracked vs ignored | Already in repository history vs untracked path excluded by rules |

## 3. Practical questions

7. **How do you create a feature branch?** `git switch -c feature/name`.
8. **How do you inspect changes before commit?** `git status` and `git diff`.
9. **How do you resolve a conflict?** Review both changes, edit the intended result, test, stage and complete merge.
10. **How do you protect secrets?** Keep them outside source control, use secret management and rotate any exposed credential.

## 4. Scenario questions

### Scenario A: Accidentally staged a .env file
Unstage it, ensure it is ignored, inspect whether it was committed or pushed, and rotate any exposed credentials. Removing it from the latest working tree does not erase history.

### Scenario B: Teammate changed the same function
Fetch latest changes, inspect conflict context, discuss intended behavior, resolve carefully and run tests.

### Scenario C: Large data files make repository huge
Remove them from tracking through an approved history/data strategy, use external object storage or Git LFS where appropriate, and document dataset access. Do not rewrite shared history without coordination.

## 5. Troubleshooting
- Push rejected: fetch/reconcile remote changes and check branch protection.
- Wrong identity: inspect Git user.name and user.email.
- Ignored file still tracked: remove it from index; ignore rules affect untracked files.
- Conflict after pull: inspect status and resolve each file.
- Detached HEAD: switch to a branch before continuing normal work.

## 6. Follow-up questions
- What is a pull request review?
- What is a protected branch?
- What is semantic versioning?
- Why write small commits?
- What is a release tag?
- How do you roll back a bad deployment?

## 7. Mini-project defence
Demonstrate repository structure, .gitignore, requirements, tests, feature branch, commit history and remote push. Explain how code review and CI checks would improve the workflow.
