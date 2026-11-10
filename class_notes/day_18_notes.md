# Day 18 — Git, Branching and Code Organization
**Phase 2: Python for Data Engineering**

## Why Git?
Git tracks code changes, enables collaboration/review/rollback, and records reproducible history. Keep code/config templates in Git; raw/sensitive data and secrets usually do not belong there.

## Basic commands
```bash
git init
git status
git add .
git commit -m "Initial data processing utility"
git log --oneline
```
- status: working tree changes.
- add: stage selected files.
- commit: record staged snapshot.
- log: inspect history.

## Branch workflow
```bash
git switch -c feature/normalize-orders
# edit and test
git add transformations.py test_transformations.py
git commit -m "Add order normalization"
git switch main
git merge feature/normalize-orders
```
Team workflows commonly use a pull request and review before merge.

## Project organization
```
data_processing_utility/
├── src/ (extract.py, transform.py, validate.py)
├── tests/
├── data/          # local/raw data, often ignored
├── requirements.txt
├── README.md
└── .gitignore
```
Separate responsibilities and document setup/run instructions.

## .gitignore
```
.venv/
__pycache__/
.env
*.log
data/raw/
```
Review staged files before committing; never commit credentials.

## Mini-project — Version-Controlled ETL Project
Initialize repository, add source/tests/docs, create a feature branch, test, commit, and merge through review.

## Revision
Git tracks history; branches isolate work; commits should be focused/reviewable; ignore secrets and local artifacts.