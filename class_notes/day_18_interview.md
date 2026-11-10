# Day 18 Interview Preparation: Git and Code Organization

**Mini-project:** Version-Controlled ETL Project

## 1. Core questions

1. **What is Git?**  
   A distributed version control system used to track code changes and collaborate.

2. **What is GitHub?**  
   A hosting and collaboration platform for Git repositories.

3. **What is a repository?**  
   Project files together with their version history.

4. **What is the working tree?**  
   The files currently checked out in the local project.

5. **What is the staging area?**  
   The place where selected changes are prepared for the next commit.

6. **What is a commit?**  
   A recorded snapshot of staged changes with metadata and a message.

7. **What is a branch?**  
   A separate line of development that can be changed and merged independently.

8. **What is a merge?**  
   Integrating changes from another branch.

9. **What is a merge conflict?**  
   A situation where Git cannot automatically reconcile competing changes.

10. **What does git clone do?**  
    Copies a remote repository locally, including its history.

11. **What does git fetch do?**  
    Downloads remote updates without integrating them into the current branch.

12. **What does git pull do?**  
    Fetches and integrates remote changes.

13. **What does git push do?**  
    Publishes local commits to a remote repository.

14. **What is .gitignore?**  
    A file listing untracked paths Git should ignore.

15. **Does .gitignore remove a file already tracked by Git?**  
    No. It does not automatically untrack existing files or erase history.

16. **Why should secrets not be committed?**  
    Repository history may expose credentials. Rotate any credential that is accidentally published.

## 2. Difference questions

| Concept | Difference |
|---|---|
| Git / GitHub | Version-control tool / hosted collaboration service |
| add / commit | Stage selected changes / record staged snapshot |
| commit / push | Save locally / publish commits to remote |
| fetch / pull | Download remote history / download and integrate |
| branch / tag | Movable development reference / named reference to a specific point |
| merge / rebase | Combine histories / replay commits onto another base |
| tracked / untracked | Already known to Git / not yet tracked |

## 3. Practical command questions

**Q1. Initialize a repository.**

```bash
git init
```

**Q2. Check current changes.**

```bash
git status
```

**Q3. Stage one file.**

```bash
git add pipeline.py
```

**Q4. Commit staged changes.**

```bash
git commit -m "Add order transformation"
```

**Q5. Create and switch to a feature branch.**

```bash
git switch -c feature/order-summary
```

**Q6. Inspect history.**

```bash
git log --oneline
```

## 4. Scenario-based questions

1. **You accidentally staged a password file but have not committed. What do you do?**  
   Unstage it, remove the secret from the working copy, add the file pattern to .gitignore if appropriate, and verify the staged changes before committing.

2. **You committed and pushed an API key. What now?**  
   Revoke or rotate the key immediately, notify the owner, inspect usage and follow the repository's secret-remediation process. Deleting the key from the latest file does not remove it from history.

3. **A push is rejected because the remote has new commits. What do you do?**  
   Fetch and inspect remote changes, integrate them safely, resolve conflicts if any, run tests and then push.

4. **A teammate and you edit the same line. What is a merge conflict?**  
   Git cannot choose the correct version automatically. Review the business intent, edit the final content, remove markers, test and commit the resolution.

5. **A data folder contains large raw files. Should it be committed?**  
   Usually not unless the project explicitly requires small, permitted sample data. Use a data storage location and document how to obtain the data.

6. **Why create a feature branch?**  
   It isolates work for review and reduces the risk of unfinished changes landing directly on main.

## 5. Follow-up questions

- What is HEAD?
- What is a remote named origin?
- What does git diff show?
- What is the difference between git diff and git diff --staged?
- Why use meaningful commit messages?
- What is a pull request?
- What should be included in a Python .gitignore?
- How do you protect credentials in a pipeline project?

## 6. Strong revision points

- Check the branch and working tree before editing or committing.
- Stage only intended files.
- Commit small, meaningful changes.
- Review diffs before committing.
- Use feature branches and code review.
- Never commit secrets or large raw datasets without an approved reason.
- Test after resolving conflicts and before pushing.


---

## 7. Follow-up interview questions

16. What is Git? A distributed version-control system that records changes and repository history.
17. Git versus GitHub? Git is the version-control tool; GitHub is a hosting and collaboration platform.
18. What is a repository? Project files and their version history.
19. What is the working tree? The checked-out files currently being edited.
20. What is the staging area? The set of changes selected for the next commit.
21. What is a commit? A recorded snapshot of staged changes with metadata and a message.
22. What is a branch? A movable reference used to develop a line of work independently.
23. What is a merge conflict? A case where Git cannot automatically combine changes.
24. What does git diff show? Changes not yet staged by default.
25. What does git diff --staged show? Changes currently in the staging area compared with the latest commit.
26. What is a remote? A named reference to another repository, commonly hosted on GitHub.
27. Why use .gitignore? To prevent selected generated, local or sensitive files from being added accidentally.
28. Is deleting a leaked API key from the latest commit enough? No. Rotate the secret and follow the history cleanup and incident process.
29. Why use feature branches? They isolate changes for review and reduce interference with the main development line.
30. What makes a good commit message? It briefly describes the purpose of the change in clear, specific language.

## 8. Scenario questions

A virtual environment appears in git status. Add the appropriate environment directory to .gitignore and ensure no secrets or needed source files are hidden.

Two developers edit the same lines. Resolve the conflict by understanding both changes, combining the intended result and running relevant tests before committing.

An API key was pushed to a public repository. Revoke or rotate it immediately; then follow the organization's incident and repository-history cleanup process.

## 9. Mini-project defense

- What did you stage and why?
- What does the commit preserve?
- Why did you create a feature branch?
- How did you resolve or avoid conflicts?
- How did you verify secrets and generated files were excluded?

## 10. Readiness rubric

| Skill | Ready when the student can… |
|---|---|
| Git basics | Initialize, inspect and commit a repository |
| Staging | Explain add and staged differences |
| Branching | Create, switch and merge a branch |
| Conflict | Explain and resolve a merge conflict |
| Hygiene | Use .gitignore and protect secrets |
| Collaboration | Explain remote and push workflow |
