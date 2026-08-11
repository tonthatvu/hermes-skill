---

name: jira-review
description: Review Jira tickets and sub-tasks for implementation difficulty, assign Credit 1/2/3, and add the review result as a comment to the reviewed Jira sub-task.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Jira Review

## DIFFICULTY / CREDIT ASSESSMENT

Every Jira ticket MUST be assigned a difficulty credit.

Use ONLY these 3 values:

* Credit 1 — Easy
* Credit 2 — Medium
* Credit 3 — Hard

Do NOT use "Unknown".
Do NOT leave the credit blank.

### Credit 1 — Easy

Assign Credit 1 when the task is relatively straightforward and has low implementation complexity.

Typical characteristics:

* Simple configuration change
* Small code change
* Single service / single component
* No significant architectural change
* No complex business logic
* No database migration or complicated data transformation
* Few dependencies
* Easy to test and verify
* Low risk

Examples:

* Add/update a configuration value
* Simple API field mapping
* Small validation change
* Simple monitoring/alert change
* Minor UI or backend adjustment

### Credit 2 — Medium

Assign Credit 2 when the task requires moderate implementation effort or involves multiple components.

Typical characteristics:

* Changes multiple files/components
* Changes API behavior or data flow
* Requires moderate business logic
* Requires database/schema changes without major migration complexity
* Requires integration with another service
* Requires several test cases
* Has moderate technical risk
* Requires investigation before implementation

Examples:

* Modify an existing API and update its consumer
* Add a new business rule
* Add a new integration with an existing internal service
* Moderate database changes
* Changes involving multiple services

### Credit 3 — Hard

Assign Credit 3 when the task has high technical complexity, high risk, or requires significant investigation/design.

Typical characteristics:

* Multiple services or systems involved
* Significant architecture/design changes
* Complex business logic
* Major database migration
* Difficult integration
* Significant backward-compatibility concerns
* High production risk
* Requires extensive testing
* Requires investigation/spike before implementation
* Multiple dependencies or external systems
* Unclear technical approach that requires design decisions

Examples:

* Major changes across multiple microservices
* New payment/integration flow
* Complex data migration
* Significant architecture change
* Changes affecting critical production flows

### IMPORTANT

Do NOT determine Credit from:

* Ticket age
* Number of sprints
* Priority
* Status
* Number of comments
* Number of people involved
* How long the ticket has been open

Determine Credit primarily from the **actual implementation complexity described in the ticket**.

If the ticket description is incomplete, estimate the credit based only on the available information and explicitly mention what information is missing.

The credit is an assessment of **implementation difficulty**, NOT actual time spent.

---

# JIRA SUB-TASK REVIEW

When the reviewed issue is a Jira Sub-task, the review MUST be performed against the actual Sub-task.

Consider, when available:

* Sub-task Summary
* Description
* Acceptance Criteria
* Parent Story / Parent issue context
* Existing comments
* Linked issues
* Dependencies
* Technical details
* Affected services/components
* Database or integration requirements

The parent Story may be used as additional context, but the difficulty Credit MUST represent the implementation complexity of the specific Sub-task being reviewed.

Do NOT automatically assign the same Credit as the parent Story.

---

# REQUIRED OUTPUT

Every ticket review MUST contain this section:

### DIFFICULTY

* **Credit:** 1 / 2 / 3
* **Level:** Easy / Medium / Hard
* **Reason:** <reason>
* **Complexity factors:**

  * <factor 1>
  * <factor 2>
  * <factor 3>

The `Credit` value MUST always be exactly `1`, `2`, or `3`.

---

# JIRA COMMENT

After reviewing a Jira Sub-task, the AI MUST add the review result as a comment to the **same Jira Sub-task**.

The review is NOT considered complete until the comment has been successfully created.

## Comment format

Use this format:

### 🤖 AI Jira Review

**Difficulty**

* **Credit:** 2
* **Level:** Medium

**Reason:** <short explanation of why this Credit was assigned>

**Complexity factors:**

* <factor 1>
* <factor 2>
* <factor 3>

**Missing information / assumptions:**

* <missing information or assumption>
* If none, write: `None`

The Credit and Level in the Jira comment MUST exactly match the final review result.

---

# COMMENTING RULES

When the reviewed issue is a Jira Sub-task:

1. Identify the exact Sub-task issue key.
2. Review the Sub-task.
3. Assign Credit 1, 2, or 3.
4. Generate the review using the required format.
5. Use the available Jira tool/API to create a comment on the **same Sub-task issue key**.
6. Wait for the Jira operation to complete.
7. Verify that the Jira tool/API confirms the comment was successfully created.
8. Only after successful confirmation, consider the review complete.

The comment MUST be added to the Sub-task itself.

Do NOT add the review comment to:

* The parent Story
* The parent Epic
* A linked issue
* Another Sub-task

unless the user explicitly requests it.

If multiple Sub-tasks are being reviewed, create a separate review comment on each Sub-task.

---

# TOOL USAGE

When a Jira write/comment tool is available, use it to create the comment.

Prefer the Jira tool/API operation specifically intended to create or add an issue comment.

The AI MUST NOT merely output the comment text and claim that it was added.

The AI MUST actually execute the Jira comment operation when the required Jira write tool is available.

If no Jira write/comment capability is available:

* Complete the difficulty assessment.
* Clearly state that the review could not be posted because no Jira comment/write capability is available.
* Provide the exact comment text that should be posted.
* Do NOT claim that the comment was successfully added.

If the Jira comment operation fails:

* Report that the comment operation failed.
* Include the error if available.
* Do NOT claim that the comment was added successfully.

---

# MULTIPLE SUB-TASKS

When reviewing multiple Jira Sub-tasks:

1. Review each Sub-task independently.
2. Assign a separate Credit to each Sub-task.
3. Add a separate comment to each Sub-task.
4. Verify each comment operation individually.
5. Do not use one comment on the parent Story as a replacement for individual Sub-task comments.

Final output should summarize the results in a table:

| Sub-task | Credit | Level  | Comment |
| -------- | -----: | ------ | ------- |
| ABC-101  |      1 | Easy   | Added   |
| ABC-102  |      2 | Medium | Added   |
| ABC-103  |      3 | Hard   | Added   |

Use `Added` only when the Jira API/tool confirms successful comment creation.

---

# COMPLETION CRITERIA

A Jira Sub-task review is complete only when:

* The Sub-task has been analyzed.
* Credit 1, 2, or 3 has been assigned.
* The reason has been determined.
* Complexity factors have been identified.
* The review comment has been created on the same Sub-task.
* The comment creation has been successfully verified.

If any of these steps cannot be completed, explicitly report which step failed.
