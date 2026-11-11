# TechPhile.ai — Data Engineer Class Notes

> 75-day Data Engineer pathway. The approved syllabus in [career_path.md](../career_path.md) is authoritative. Each day has separate class notes and interview preparation.

## Rebuild progress

The course is being rebuilt phase by phase to one consistent beginner-first teaching standard. A phase is marked **Complete** only after every day's class notes and interview file in that phase have been rebuilt, committed, and verified in GitHub. Completed phases include a phase-level summary and completion checklist.

| Phase | Days | Status | Rebuild progress |
|---|---:|---|---:|
| Phase 1 — Data Engineering Foundations | 1–5 | Complete | 5/5 days rebuilt and verified |
| Phase 2 — Python for Data Engineering | 6–18 | Complete | 13/13 days rebuilt and verified |
| Phase 3 — SQL and Database Engineering | 19–29 | Rebuilt; final technical review | 9/11 days rebuilt and verified; Days 19–20 retained for cross-check |
| Phase 4 — Data Processing and Pipelines | 30–39 | **Rebuild required** | Existing drafts are not student-ready; re-authoring from teaching standard |
| Phase 5 — Data Warehousing and Modeling | 40–49 | Authored; runtime validation pending | 10/10 lesson sets prepared |
| Phase 6 — Cloud and Distributed Processing | 50–59 | Pending | 0/10 days verified |
| Phase 7 — Orchestration, Quality and Monitoring | 60–67 | Pending | 0/8 days verified |
| Phase 8 — Data Engineering Capstone | 68–75 | Pending | 0/8 days verified |


## Phase 1 — Data Engineering Foundations (Days 1–5)

| Day | Syllabus topic | Mini-project | Class notes | Interview |
|---|---|---|---|---|
| 1 | Introduction to Data Engineering | Data Engineering Architecture Map | [Notes](day_01_notes.md) | [Interview](day_01_interview.md) |
| 2 | Data Processing Fundamentals | Data Processing Comparison Lab | [Notes](day_02_notes.md) | [Interview](day_02_interview.md) |
| 3 | Data Sources and Formats | Data Source Inventory | [Notes](day_03_notes.md) | [Interview](day_03_interview.md) |
| 4 | Data Architecture and Integration | ETL/ELT Architecture Designer | [Notes](day_04_notes.md) | [Interview](day_04_interview.md) |
| 5 | Data Pipeline Fundamentals | Pipeline Requirements Document | [Notes](day_05_notes.md) | [Interview](day_05_interview.md) |

## Phase 1 completion summary — Data Engineering Foundations (Days 1–5)\n\n**Phase outcome:** Students can explain the role of data engineering, distinguish common processing patterns, identify data sources and formats, understand ETL/ELT architecture, and plan a reliable and secure data pipeline.\n\n**Day-wise learning outcomes**\n\n| Day | What students should now be able to do | Mini-project evidence |\n|---|---|---|\n| 1 | Explain Data Engineering responsibilities, the data lifecycle and a high-level data architecture. | Data Engineering Architecture Map |\n| 2 | Compare OLTP/OLAP, batch/streaming, real-time/near-real-time and event-driven processing. | Data Processing Comparison Lab |\n| 3 | Classify structured, semi-structured and unstructured data and select appropriate source formats. | Data Source Inventory |\n| 4 | Explain ETL/ELT, lakes, warehouses, lakehouses and integration architecture; recognize Kafka and dbt roles. | ETL/ELT Architecture Designer |\n| 5 | Gather pipeline requirements and plan lifecycle, reliability, scalability, fault tolerance, governance, security and operations. | Pipeline Requirements Document |\n\n**Phase completion checklist**\n\n- [x] Day 1 class notes and interview guide rebuilt and verified.\n- [x] Day 2 class notes and interview guide rebuilt and verified.\n- [x] Day 3 class notes and interview guide rebuilt and verified.\n- [x] Day 4 class notes and interview guide rebuilt and verified.\n- [x] Day 5 class notes and interview guide rebuilt and verified.\n- [x] Day-wise mini-project names checked against the approved syllabus.\n- [x] Phase tracker updated after file verification.\n\n**Phase 1 is complete.** Students are ready to begin Python for Data Engineering.\n\n## Phase 2 — Python for Data Engineering (Days 6–18)

| Day | Syllabus topic | Mini-project | Class notes | Interview |
|---|---|---|---|---|
| 6 | Python Introduction and Environment | Data Processing Utility | [Notes](day_06_notes.md) | [Interview](day_06_interview.md) |
| 7 | Functions and Modules | Reusable Transformation Library | [Notes](day_07_notes.md) | [Interview](day_07_interview.md) |
| 8 | File Operations | File Processing Manager | [Notes](day_08_notes.md) | [Interview](day_08_interview.md) |
| 9 | Data Formats and File Processing | Multi-Format Data Reader | [Notes](day_09_notes.md) | [Interview](day_09_interview.md) |
| 10 | Exception Handling and Debugging | Resilient File Processor | [Notes](day_10_notes.md) | [Interview](day_10_interview.md) |
| 11 | Iterators and Generators | Large File Reader | [Notes](day_11_notes.md) | [Interview](day_11_interview.md) |
| 12 | API Integration | API Data Extractor | [Notes](day_12_notes.md) | [Interview](day_12_interview.md) |
| 13 | Python Database Connectivity | Database Record Extractor | [Notes](day_13_notes.md) | [Interview](day_13_interview.md) |
| 14 | Pandas for Data Engineering | Dataset Transformation Tool | [Notes](day_14_notes.md) | [Interview](day_14_interview.md) |
| 15 | NumPy and Numerical Processing | Numerical Transformation Lab | [Notes](day_15_notes.md) | [Interview](day_15_interview.md) |
| 16 | Data Validation and Schema Checking | Data Validation Engine | [Notes](day_16_notes.md) | [Interview](day_16_interview.md) |
| 17 | Testing Data Processing Code | Transformation Test Suite | [Notes](day_17_notes.md) | [Interview](day_17_interview.md) |
| 18 | Git and Code Organization | Version-Controlled ETL Project | [Notes](day_18_notes.md) | [Interview](day_18_interview.md) |


## Phase 2 completion summary — Python for Data Engineering (Days 6–18)

**Phase outcome:** Students can create and run Python programs, organize reusable code, work with files and common data formats, handle failures, process data efficiently, integrate APIs and databases, transform datasets with Pandas and NumPy, validate records, test transformations, and manage code with Git.

**Day-wise learning outcomes**

| Day | What students should now be able to do | Mini-project |
|---|---|---|
| 6 | Set up Python, run scripts, understand variables and collections, and calculate simple order summaries. | Data Processing Utility |
| 7 | Define and call functions, use parameters and return values, and organize reusable modules. | Reusable Transformation Library |
| 8 | Create, read, append, inspect, copy and organize files safely using paths and context managers. | File Processing Manager |
| 9 | Read and write CSV, JSON, XML and Parquet, and compare format trade-offs. | Multi-Format Data Reader |
| 10 | Diagnose exceptions, handle expected failures, log operational events and debug systematically. | Resilient File Processor |
| 11 | Use iterators and generators, understand lazy evaluation and process large inputs in batches. | Large File Reader |
| 12 | Call REST APIs, parse responses, check status, handle timeouts and understand pagination and rate limits. | API Data Extractor |
| 13 | Connect Python to a relational database, execute parameterized SQL, retrieve records and manage transactions. | Database Record Extractor |
| 14 | Inspect, clean, filter, aggregate, merge and export tabular data with Pandas. | Dataset Transformation Tool |
| 15 | Create NumPy arrays, index and filter values, use vectorized operations, axes and broadcasting. | Numerical Transformation Lab |
| 16 | Validate required fields, types, ranges, allowed values and duplicates; preserve rejected records and reasons. | Data Validation Engine |
| 17 | Write and run unit tests, cover boundary cases and distinguish unit, integration and end-to-end testing. | Transformation Test Suite |
| 18 | Use Git repositories, staging, commits, branches, merges, .gitignore and GitHub collaboration safely. | Version-Controlled ETL Project |

**Phase completion checklist**

- [x] Day 6 class notes and interview guide rebuilt and verified.
- [x] Day 7 class notes and interview guide rebuilt and verified.
- [x] Day 8 class notes and interview guide rebuilt and verified.
- [x] Day 9 class notes and interview guide rebuilt and verified.
- [x] Day 10 class notes and interview guide rebuilt and verified.
- [x] Day 11 class notes and interview guide rebuilt and verified.
- [x] Day 12 class notes and interview guide rebuilt and verified.
- [x] Day 13 class notes and interview guide rebuilt and verified.
- [x] Day 14 class notes and interview guide rebuilt and verified.
- [x] Day 15 class notes and interview guide rebuilt and verified.
- [x] Day 16 class notes and interview guide rebuilt and verified.
- [x] Day 17 class notes and interview guide rebuilt and verified.
- [x] Day 18 class notes and interview guide rebuilt and verified.
- [x] All mini-project names checked against the approved syllabus.
- [x] Phase tracker updated after file verification.

**Phase 2 is complete.** Students are ready to begin SQL and Database Engineering in Phase 3.

## Phase 3 — SQL and Database Engineering (Days 19–29)

| Day | Syllabus topic | Mini-project | Class notes | Interview |
|---|---|---|---|---|
| 19 | Database Fundamentals | Operational Database Design | [Notes](day_19_notes.md) | [Interview](day_19_interview.md) |
| 20 | SQL DDL, DML and Constraints | Database Creation Lab | [Notes](day_20_notes.md) | [Interview](day_20_interview.md) |
| 21 | SQL Data Extraction | Multi-Table Extraction Queries | [Notes](day_21_notes.md) | [Interview](day_21_interview.md) |
| 22 | SQL Aggregations and Advanced Queries | Business Data Aggregation | [Notes](day_22_notes.md) | [Interview](day_22_interview.md) |
| 23 | SQL Window Functions | Historical Trend Query Engine | [Notes](day_23_notes.md) | [Interview](day_23_interview.md) |
| 24 | Transactions and ACID Principles | Transaction Simulation | [Notes](day_24_notes.md) | [Interview](day_24_interview.md) |
| 25 | Database Indexing and Query Optimization | Query Performance Investigation | [Notes](day_25_notes.md) | [Interview](day_25_interview.md) |
| 26 | Database Normalization | Data Model Comparison | [Notes](day_26_notes.md) | [Interview](day_26_interview.md) |
| 27 | Database Objects and Automation | Database Automation Lab | [Notes](day_27_notes.md) | [Interview](day_27_interview.md) |
| 28 | Database Administration Fundamentals | Database Operations Checklist | [Notes](day_28_notes.md) | [Interview](day_28_interview.md) |
| 29 | Python and Database Integration | Database Loader | [Notes](day_29_notes.md) | [Interview](day_29_interview.md) |

## Phase 3 rebuild status — SQL and Database Engineering (Days 19–29)

**Current status:** Days 21–29 have been rebuilt with beginner-first class notes and complete interview answers. Days 19–20 are retained because their class notes already have substantial depth; they are included in the final cross-check for terminology, schema and sample-data consistency.

**Teaching standard applied to Days 21–29**
- Start from the student's prior knowledge and explain why the topic is needed.
- Introduce one concept at a time, then show runnable SQL and expected results.
- Explain query clauses, parameters, output rows and business meaning.
- Use the shared TechStore sample database and identify PostgreSQL-specific syntax.
- Include common mistakes, debugging guidance, practical mini-projects and validation.
- Provide a separate interview guide with written answers and scenario questions rather than generic answer prompts.

| Day | Rebuild status | Class notes | Interview guide |
|---|---|---:|---:|
| 19 | Retained; cross-check | 1,552 lines | 438 lines |
| 20 | Retained; cross-check | 1,288 lines | 385 lines |
| 21 | Rebuilt | 1,606 lines | 549 lines |
| 22 | Rebuilt | 1,379 lines | 374 lines |
| 23 | Rebuilt | 931 lines | 254 lines |
| 24 | Rebuilt | 798 lines | 221 lines |
| 25 | Rebuilt | 698 lines | 233 lines |
| 26 | Rebuilt | 718 lines | 202 lines |
| 27 | Rebuilt | 750 lines | 206 lines |
| 28 | Rebuilt | 706 lines | 231 lines |
| 29 | Rebuilt | 1,184 lines | 250 lines |

**Important:** Line count is a verification aid, not a quality score. SQL examples still require execution in the course's supported PostgreSQL/MySQL environments before the phase is marked fully complete. The Phase 3 status will be changed to Complete only after the Day 19–20 cross-check and final review.

## Future phases
- Phase 3: SQL and Database Engineering (Days 19–29)
- Phase 4: Data Processing and Pipelines (Days 30–39)
- Phase 5: Data Warehousing and Modeling (Days 40–49)
- Phase 6: Cloud and Distributed Processing (Days 50–59)
- Phase 7: Orchestration, Quality and Monitoring (Days 60–67)
- Phase 8: Data Engineering Capstone (Days 68–75)

**Learning format:** beginner-friendly explanations, practical code examples, expected outputs, operational considerations and a separate interview-preparation file for every day.
## Phase 4 — Data Processing and Pipelines (Days 30–39)

| Day | Syllabus topic | Mini-project | Class notes | Interview |
|---|---|---|---|---|
| 30 | ETL and Data Extraction Patterns | Incremental Extraction Pipeline | [Notes](day_30_notes.md) | [Interview](day_30_interview.md) |
| 31 | File-Based Data Ingestion | Multi-File Ingestion System | [Notes](day_31_notes.md) | [Interview](day_31_interview.md) |
| 32 | API-Based Data Ingestion | API Ingestion Pipeline | [Notes](day_32_notes.md) | [Interview](day_32_interview.md) |
| 33 | Database Ingestion | Database Extractor | [Notes](day_33_notes.md) | [Interview](day_33_interview.md) |
| 34 | Data Transformation and Business Rules | Transformation Pipeline | [Notes](day_34_notes.md) | [Interview](day_34_interview.md) |
| 35 | Data Cleaning and Schema Evolution | Data Cleaning Framework | [Notes](day_35_notes.md) | [Interview](day_35_interview.md) |
| 36 | Batch Processing and Partitioning | Partitioned Batch Processor | [Notes](day_36_notes.md) | [Interview](day_36_interview.md) |
| 37 | Incremental Loading and Upserts | Incremental Data Loader | [Notes](day_37_notes.md) | [Interview](day_37_interview.md) |
| 38 | Pipeline Reliability and Failure Handling | Reliable Pipeline Simulator | [Notes](day_38_notes.md) | [Interview](day_38_interview.md) |
| 39 | Pipeline Logging and Observability | Pipeline Observability Tool | [Notes](day_39_notes.md) | [Interview](day_39_interview.md) |

**Phase 4 content status:** Days 32–39 class notes and interview guides included. Each lesson includes a beginner explanation, sample code/output, progressive workflow, common mistakes, debugging, project acceptance checks, exercises and production context. Interview guides contain model answers, scenarios and project defence. Execute examples in the course environment before labelling them runtime-tested.

---



## Day 30 — ETL and Data Extraction (rebuild artifacts)

- [Day 30 Class Notebook](day_30_notes.md) — 697 lines, beginner lesson with SQL/Python walkthrough, output interpretation, validation and debugging.
- [Day 30 Project Workbook](day_30_project_workbook.md) — 305 lines, step-by-step project cells, exact expected output shapes, acceptance checks and failure drills.
- [Day 30 Interview Guide](day_30_interview.md) — complete model answers, follow-ups, project defence and common traps.

**Status:** Day 30 is the first rebuilt lesson under the independent-learning standard. Review against the real TechStore schema and execute all cells before marking fully classroom-verified.

## Day 31 — File-Based Data Ingestion (rebuild artifacts)

- [Day 31 Class Notebook](day_31_notes.md) — 802 lines; CSV/JSON from zero, file discovery, schema and record validation, accept/reject, reconciliation, reruns and debugging.
- [Day 31 Project Workbook](day_31_project_workbook.md) — 312 lines; input fixture, execution cells, expected results, reconciliation, failure drills and acceptance checklist.
- [Day 31 Interview Guide](day_31_interview.md) — complete model answers and project defence.

**Status:** Rebuilt under independent-learning standard. Execute cells and test against the student's environment before classroom verification.

## Day 32 — API-Based Data Ingestion (rebuild artifacts)

- [Day 32 Class Notebook](day_32_notes.md) — 643 lines; API fundamentals, request/response, status codes, JSON, parameters, pagination, retries, security and persistence.
- [Day 32 Project Workbook](day_32_project_workbook.md) — guided execution, mock pagination, expected outputs, failure drills and acceptance criteria.
- [Day 32 Interview Guide](day_32_interview.md) — full model answers, follow-up prompts and project defence.

**Status:** Rebuilt under independent-learning standard. Live API tests depend on network and provider availability; mock exercises remain executable offline.

## Day 33 — Database Ingestion (rebuild artifacts)

- [Day 33 Class Notebook](day_33_notes.md) — 522 lines; source safety, SQL parameters, fetch methods, batching, streaming, watermark limits, consistency and recovery.
- [Day 33 Project Workbook](day_33_project_workbook.md) — guided extraction cells, batch output, reconciliation, failure drills and acceptance criteria.
- [Day 33 Interview Guide](day_33_interview.md) — full model answers, limitations and project defence.

**Status:** Rebuilt under independent-learning standard. Execute with TechStore schema and verify driver/cursor behavior in the learner environment.

## Day 35 — Data Cleaning and Schema Evolution (rebuild artifacts)

- [Day 35 Class Notes](day_35_notes.md) — profiling, nulls, duplicates, safe conversions, schema drift/evolution and accepted/rejected reconciliation.
- [Day 35 Project Workbook](day_35_project_workbook.md) — step-by-step cleaning lab, multi-reason quarantine, schema drift drill, assertions and submission evidence.
- [Day 35 Interview Guide](day_35_interview.md) — complete model answers, production scenarios and project explanation.

**Status:** Rebuilt as a beginner-first draft. Expected results are reasoned from the documented fixture; run the notebook in the supported TechPhile environment before claiming runtime verification.

## Day 34 — Data Transformation and Business Rules (rebuild artifacts)

- [Day 34 Class Notebook](day_34_notes.md) — 547 lines; raw/curated, profiling, parsing, Decimal, mapping, derived values, cross-field rules and reconciliation.
- [Day 34 Project Workbook](day_34_project_workbook.md) — guided transformation fixture, expected outputs, accepted/rejected data and validation.
- [Day 34 Interview Guide](day_34_interview.md) — complete model answers and project defence.

**Status:** Rebuilt under independent-learning standard. Confirm all business rules with the dataset/owner; examples are explicitly teaching rules.


## Phase 4 — Data Processing and Pipelines (Days 30–39)

Each day has a class notebook, a guided project workbook and an interview guide. Complete the artifacts in order and retain your own execution evidence.

| Day | Topic | Class notes | Project workbook | Interview |
|---|---|---|---|---|
| 30 | ETL and Data Extraction | [Notes](day_30_notes.md) | [Workbook](day_30_project_workbook.md) | [Guide](day_30_interview.md) |
| 31 | File-Based Data Ingestion | [Notes](day_31_notes.md) | [Workbook](day_31_project_workbook.md) | [Guide](day_31_interview.md) |
| 32 | API-Based Data Ingestion | [Notes](day_32_notes.md) | [Workbook](day_32_project_workbook.md) | [Guide](day_32_interview.md) |
| 33 | Database Ingestion | [Notes](day_33_notes.md) | [Workbook](day_33_project_workbook.md) | [Guide](day_33_interview.md) |
| 34 | Data Transformation and Business Rules | [Notes](day_34_notes.md) | [Workbook](day_34_project_workbook.md) | [Guide](day_34_interview.md) |
| 35 | Data Cleaning and Schema Evolution | [Notes](day_35_notes.md) | [Workbook](day_35_project_workbook.md) | [Guide](day_35_interview.md) |
| 36 | Batch Processing and Partitioning | [Notes](day_36_notes.md) | [Workbook](day_36_project_workbook.md) | [Guide](day_36_interview.md) |
| 37 | Incremental Loading and Upserts | [Notes](day_37_notes.md) | [Workbook](day_37_project_workbook.md) | [Guide](day_37_interview.md) |
| 38 | Pipeline Reliability and Failure Handling | [Notes](day_38_notes.md) | [Workbook](day_38_project_workbook.md) | [Guide](day_38_interview.md) |
| 39 | Pipeline Logging and Observability | [Notes](day_39_notes.md) | [Workbook](day_39_project_workbook.md) | [Guide](day_39_interview.md) |

**Teaching-quality status:** content rebuild and artifact indexing completed for the phase. Runtime validation still belongs to the student environment/CI: execute every workbook, verify installed dependencies, compare outputs, and record environment-specific differences before marking classroom delivery fully validated.


## Phase 5 — Data Warehousing and Modeling (Days 40–49)

Each day has separate class notes, a guided project workbook and a complete interview guide. Class notes are written for independent beginner learning. SQL examples use PostgreSQL unless a platform difference is explicitly noted.

| Day | Syllabus topic | Mini-project | Class notes | Project workbook | Interview |
|---|---|---|---|---|---|
| 40 | Data Warehouse Architecture | Warehouse Architecture Plan | [Notes](day_40_notes.md) | [Workbook](day_40_project_workbook.md) | [Guide](day_40_interview.md) |
| 41 | Fact and Dimension Tables | Sales Star Schema | [Notes](day_41_notes.md) | [Workbook](day_41_project_workbook.md) | [Guide](day_41_interview.md) |
| 42 | Slowly Changing Dimensions | Customer History Model | [Notes](day_42_notes.md) | [Workbook](day_42_project_workbook.md) | [Guide](day_42_interview.md) |
| 43 | Keys and Data Relationships | Warehouse Key Management | [Notes](day_43_notes.md) | [Workbook](day_43_project_workbook.md) | [Guide](day_43_interview.md) |
| 44 | Data Marts and Subject-Oriented Modeling | Sales Data Mart | [Notes](day_44_notes.md) | [Workbook](day_44_project_workbook.md) | [Guide](day_44_interview.md) |
| 45 | Data Quality and Validation | Data Quality Framework | [Notes](day_45_notes.md) | [Workbook](day_45_project_workbook.md) | [Guide](day_45_interview.md) |
| 46 | Data Reconciliation and Lineage | Source-to-Target Reconciliation | [Notes](day_46_notes.md) | [Workbook](day_46_project_workbook.md) | [Guide](day_46_interview.md) |
| 47 | Metadata and Data Documentation | Data Dictionary Generator | [Notes](day_47_notes.md) | [Workbook](day_47_project_workbook.md) | [Guide](day_47_interview.md) |
| 48 | Warehouse Loading Strategies | Warehouse Load Simulator | [Notes](day_48_notes.md) | [Workbook](day_48_project_workbook.md) | [Guide](day_48_interview.md) |
| 49 | Warehouse Performance Optimization | Warehouse Performance Audit | [Notes](day_49_notes.md) | [Workbook](day_49_project_workbook.md) | [Guide](day_49_interview.md) |

**Phase 5 content status:** All ten lesson sets have been authored for this phase batch. Class notes, project workbooks and interview guides are separate. Runtime execution of SQL examples and environment-specific dbt/warehouse behavior remains to be validated before marking the phase classroom-verified.

**Teaching progression:** architecture -> dimensional model -> historical dimensions -> key management -> subject marts -> quality -> reconciliation and lineage -> metadata -> loading -> performance.
