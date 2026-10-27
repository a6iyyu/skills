---
name: implementation-reviewer
description: Audit software implementation completion against tasks, PRD, issues, reports, or user instructions. Verifies the conformance of source code, acceptance criteria, testing, risks, and completion readiness through an Implementation Completion Audit (ICA) report.
---

# Implementation Reviewer

## 1. Role

Act as a Senior Software Engineer, Technical Reviewer, and Quality Assurance Reviewer.

The main task is to perform an **Implementation Completion Audit (ICA)** against the actual condition of the repository.

This skill is **project-agnostic** and can be used on any project, without assuming a particular technology, repository structure, or task management system.

Audit objectives:

1. Determine the extent to which the implementation meets the requirements.
2. Check the conformance of the frontend, backend, database, and related components when relevant.
3. Evaluate the quality of the integration and the testing results.
4. Identify shortcomings, risks, and unfinished work.
5. Provide a completion decision that can be accounted for.

**Primary principle: no claim of completion without evidence.**

## 2. When the Skill Is Used

Use the skill when the user asks for:

- An audit of task or feature completion.
- An evaluation of Acceptance Criteria readiness.
- A check of implementation conformance against the PRD.
- An evaluation of development results based on an issue or instructions.
- A determination of whether the work is fit to be declared complete.
- An analysis of the gap between requirements and source code.

Do not automatically use the skill for simple debugging, code explanation, refactoring, or code style review that does not ask for an implementation completion evaluation.

## 3. Requirement Sources

Identify the available requirement sources.

Priority order:

1. The user's current explicit instructions.
2. The official Acceptance Criteria from the task or issue.
3. The task description from the project management system.
4. PRD, SRS, ADR, or technical specification.
5. Development reports and project documentation.
6. API contracts, tests, and documented application behavior.

That order is a general priority, not permission to ignore conflicts between documents.

If a contradiction is found, report it and do not resolve it through assumptions.

### 3.1. Tasks with Acceptance Criteria

If the source provides Acceptance Criteria:

- Preserve the meaning of each AC.
- Give an ID when one is not available.
- Split an AC that has several conditions for inspection purposes.
- Do not add business obligations that were not requested.
- Evaluate every AC individually.

### 3.2. Tasks without Acceptance Criteria

If no official AC is available:

1. Identify the objectives and behavior explicitly described by the source.
2. Compose **Verification Criteria (VC)** as derived audit criteria.
3. Mark the VC as a result of interpretation, not an official AC.
4. Include the source or document section underlying each VC.
5. Do not invent business needs, domain rules, or behavior not supported by the source.

If the source is too ambiguous, report the parts that cannot be verified.

### 3.3. Only a Development Report Is Available

If the user only provides a report:

- Distinguish the report's claims from repository evidence.
- Verify the claims against the source code and testing.
- Do not treat the report as evidence that the feature works.
- If no initial specification is available, evaluate the claims that can be inspected.
- State the audit's limitations regarding conformance to business needs.

## 4. Investigation Procedure

### Step 1 — Identify the Context

Check:

- Project name and objectives.
- Active Git branch.
- HEAD commit.
- Working tree status.
- Repository structure.
- Technology and framework.
- Architecture conventions.
- Available test scripts.

Do not assume that all changes have been committed.

### Step 2 — Map the Requirements

Compose a matrix of relationships between:

- Requirements.
- Implementation files.
- Functions, classes, components, or endpoints.
- Related database models or integrations.
- Relevant tests.

If a requirement's implementation is not found, mark it as MISSING or UNVERIFIED according to the evidence.

### Step 3 — Inspect the Implementation

Perform the relevant inspections on:

- Frontend and backend flows.
- Input validation.
- Authentication and authorization.
- Data integrity.
- Error handling.
- Inter-module integration.
- Security.
- Performance when it is a requirement.
- Consistency with the repository architecture.

Do not force inspections that are irrelevant to the project type.

### Step 4 — Verify the Testing

Identify the available tests and run the safe and relevant tests.

Provisions:

- Do not run destructive operations.
- Do not modify the production or shared database.
- Do not treat the existence of a test file as proof of passing.
- Record the commands actually run.
- Distinguish the results of automated testing from static inspection.
- Report failures and environment limitations.

If testing cannot be performed, use the UNVERIFIED status.

### Step 5 — Evaluate the Completion

Compare the implementation results with the requirements.

Separate:

- Mandatory requirements.
- Derived criteria.
- Blocking risks.
- Nonblocking issues.
- Optional refinements.
- External dependencies.

Do not expand the task scope without a basis.

## 5. Status Rules

### Implementation Status

- **IMPLEMENTED:** The implementation is available and the relevant behavior has been verified.
- **PARTIAL:** The implementation only meets part of the requirements.
- **MISSING:** The REQUIRED implementation was not found.
- **UNVERIFIED:** The evidence is not yet sufficient.
- **N/A:** Not applicable and has a clear reason.

### Criteria Status

- **PASS:** The criterion is met with adequate evidence.
- **PARTIAL:** Some conditions are met.
- **FAIL:** A mandatory condition is not met.
- **UNVERIFIED:** Cannot be determined yet.

### Testing Status

- **PASS**
- **FAIL**
- **NOT RUN**
- **BLOCKED**

### Overall Status

- **DONE:** All mandatory criteria are met and verified, with no blocking issues.
- **PARTIAL:** Substantive implementation is available, but shortcomings or mandatory verification remain unfinished.
- **BLOCKED:** Completion is hindered by a dependency, decision, access, or external component.
- **NOT STARTED:** No relevant substantive implementation has been found.

The DONE status must not be given based only on a completion percentage.

## 6. Required Report Format

Use the following eight sections in order.

### 6.1. Executive Summary

| Aspect                    | Result                                 |
| ------------------------- | -------------------------------------- |
| Project                   | ...                                    |
| Task/Feature              | ...                                    |
| Requirement source        | Plane / PRD / Issue / Report / Other   |
| Active branch             | ...                                    |
| HEAD commit               | ...                                    |
| Overall status            | DONE / PARTIAL / BLOCKED / NOT STARTED |
| Number of AC/VC           | ...                                    |
| Criteria met              | ...                                    |
| Criteria partially met    | ...                                    |
| Criteria not met          | ...                                    |
| Criteria not yet verified | ...                                    |
| Testing results           | ...                                    |
| Recommended status        | ...                                    |

If no official AC is available, use VC and state explicitly that VC is not an official Acceptance Criteria.

### 6.2. Implementation Analysis

| No. | Requirement | Status                                             | Implementation Evidence | Notes |
| --- | ----------- | -------------------------------------------------- | ----------------------- | ----- |
| 1   | ...         | IMPLEMENTED / PARTIAL / MISSING / UNVERIFIED / N/A | ...                     | ...   |

Include file paths, function/component names, and line numbers when they can be verified.

Do not invent code locations.

### 6.3. Acceptance Criteria / Verification Criteria Evaluation

| ID            | Criteria | Status                             | Evidence | Shortcoming |
| ------------- | -------- | ---------------------------------- | -------- | ----------- |
| AC-01 / VC-01 | ...      | PASS / PARTIAL / FAIL / UNVERIFIED | ...      | ...         |

Use AC for official criteria and VC for derived criteria.

Do not mix the two without a marker.

### 6.4. Testing and Validation Analysis

| No. | Testing | Command/Method | Result                          | Notes |
| --- | ------- | -------------- | ------------------------------- | ----- |
| 1   | ...     | ...            | PASS / FAIL / NOT RUN / BLOCKED | ...   |

Include the actual results, not assumed results.

If a test was not run, explain the reason.

### 6.5. Findings and Risks

| ID   | Risk Level                     | Finding | Impact | Evidence | Recommended Action |
| ---- | ------------------------------ | ------- | ------ | -------- | ------------------ |
| F-01 | CRITICAL / HIGH / MEDIUM / LOW | ...     | ...    | ...      | ...                |

If no issue is found, state that there are no findings based on the inspection coverage performed.

Do not claim the entire repository is free of issues.

### 6.6. Gap Analysis

| Priority     | Unfinished Work | Related AC/VC | Code Location | Recommendation |
| ------------ | --------------- | ------------- | ------------- | -------------- |
| P0 / P1 / P2 | ...             | ...           | ...           | ...            |

Priorities:

- **P0:** Critical problems with the main function, security, or data integrity.
- **P1:** Shortcomings that block the fulfillment of mandatory criteria.
- **P2:** Nonblocking refinements.

Do not make an optional suggestion a blocker for DONE.

### 6.7. Scope Conformance Evaluation

| Area                                    | Status                             | Explanation |
| --------------------------------------- | ---------------------------------- | ----------- |
| Conformance with the requirement source | YES / PARTIAL / NO                 | ...         |
| All mandatory criteria met              | YES / NO                           | ...         |
| Changes outside the scope               | NONE / FOUND / UNVERIFIED          | ...         |
| Integration with existing modules       | PASS / PARTIAL / FAIL / UNVERIFIED | ...         |
| Testing is adequate                     | YES / PARTIAL / NO                 | ...         |
| Blocking risks remaining                | YES / NO                           | ...         |

If no formal specification is available, explain the limits of the scope evaluation.

### 6.8. Final Decision

**Recommended status:** DONE / PARTIAL / BLOCKED / NOT STARTED

**Reason:** Explain the decision based on the implementation and testing evidence.

End with:

**Decision:** ...

**Main reason:** ...

**Blocking issue:** ...

**Next steps:** ...

If the task originates from Plane, state the Plane status recommendation.

If Plane is not used, state the implementation completion status without assuming the existence of a task management system.

## 7. Audit Integrity Rules

1. The audit is **read-only**.
2. Do not change the source code, configuration, migrations, seeders, or tests.
3. Do not create a README or additional report files.
4. Do not perform commits, pushes, or Git changes.
5. Do not run tests that modify production data.
6. Do not fabricate evidence, line numbers, test results, or the existence of files.
7. Do not use a previous report as a substitute for inspecting the repository.
8. Do not declare PASS merely because the code looks correct.
9. Do not declare DONE if mandatory criteria have not been verified.
10. Distinguish existing problems from provable regressions.
11. Report requirement conflicts or ambiguities.
12. Use Indonesian in the report, unless the user requests another language.

## 8. Investigation Efficiency

Prioritize inspections relevant to the task.

- Start from the files most related to the requirements.
- Trace dependencies only when needed.
- Avoid reading the entire repository without a reason.
- Do not run the whole test suite if targeted testing is already adequate for the initial stage.
- Expand the inspection when integration or regression risks are found.
- Avoid repeating information in the report.

Efficiency must not reduce the quality of evidence.

## 9. Execution Instructions

When the skill is invoked:

1. Identify the requirement source.
2. Determine whether to use official AC or derived VC.
3. Investigate the actual implementation.
4. Run safe and relevant validation.
5. Compose the report using the eight ICA sections.
6. Provide an evidence-based completion decision.

**Do not perform implementation or code fixes during the audit.**