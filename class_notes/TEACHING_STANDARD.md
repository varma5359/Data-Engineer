# TechPhile.ai — Self-Learning Class Notes Teaching Standard

**Purpose:** A student should be able to learn from these notes without a live mentor or an AI assistant. Notes are the teacher—not a summary of what a teacher might say.

## Non-negotiable learning outcome
A beginner can independently: understand the idea, follow setup, execute every step, interpret outputs, complete the project, diagnose common failures, verify correctness, and answer interview questions with reasoning.

## Authoring rules
1. Teach from zero. Define prerequisite terms before using them.
2. Explain *why* before *what/how*. Use a relatable business scenario and analogy, then connect analogy to actual engineering.
3. Introduce one new idea per step. Never jump from a definition to a large final script.
4. Use progressive examples: tiny input -> one operation -> variations -> realistic dataset -> project.
5. Every executable cell has: goal, where to run it, prerequisites, code, exact expected output, line-by-line explanation, output walkthrough, checkpoint question and likely error/fix.
6. Output must be derived from the displayed input. Never invent output or omit rows without explaining why. If output is environment/version dependent, label it and show how to inspect it.
7. Explain syntax, keywords, parameters, operators, clauses, data types and relevant return values in beginner language.
8. Explain project workflow explicitly: requirements -> input files/tables -> folder/environment setup -> each step/code cell -> output files/tables -> validation -> rerun/recovery -> final deliverable.
9. A project is not a code dump. Every milestone must state what the student has built, how to run it, what should appear, and how to verify it.
10. Teach failure: wrong code or input -> exact error/symptom -> cause -> correction -> prevention. Cover at least normal, empty, null, invalid, duplicate, boundary and replay/failure cases where relevant.
11. Include business decisions and trade-offs: what a junior engineer should choose, why, what alternatives exist, and what risks matter.
12. Include testing and evidence: row counts, key uniqueness, totals, schema, freshness, checksums or other topic-appropriate checks.
13. Interview guide must give complete model answers, reasoning, worked examples, follow-up questions, traps and a project-based answer—not generic hints.
14. Give practice tasks in graduated difficulty and provide solutions/explanations (or a clearly marked self-check answer section).
15. Use simple English, short paragraphs, meaningful tables and flow arrows. Avoid unexplained acronyms and formula-heavy formatting.
16. No arbitrary line count. Depth and learner independence are the acceptance criteria. Do not pad with repeated generic text.

## Required lesson architecture
- Lesson map: what, why, prerequisites, outcomes, project deliverable.
- Scenario: business request, source data, constraints and stakeholder.
- Concept taught in small pieces, each with analogy, actual definition and use case.
- Setup: software, folder, files, commands, expected setup verification.
- Guided cells: code/query, exact output, detailed explanation and checkpoint.
- Progressive practice and variations.
- Project: requirements, design, implementation milestones, full runnable artifact, run instructions, outputs and acceptance tests.
- Debugging clinic and common mistakes.
- Production perspective: safety, reliability, performance, security and operational decisions as appropriate.
- Exercises + worked solutions.
- Interview guide with full answers.
- Revision map, glossary and final competency checklist.

## Quality gate before calling a day complete
- [ ] A student can start with only stated prerequisites.
- [ ] All commands/code are runnable and setup is explicit.
- [ ] Outputs match code/data and are explained.
- [ ] Project is completed end-to-end with checkpoints.
- [ ] Errors and recovery are taught.
- [ ] Every interview question has a substantive answer.
- [ ] No filler, placeholders, repeated boilerplate or unexplained jumps.
- [ ] A reviewer can follow the lesson as a student and reproduce its evidence.

## Explicit rejection examples
- “Used to design and explain reliable pipelines” repeated as every glossary definition.
- A code block followed only by “this reads the data.”
- A project title with no actual build/run/verification instructions.
- “Explain X” as interview answer instead of a complete model answer.
- Marking a lesson complete because it exists or has a high line count.
