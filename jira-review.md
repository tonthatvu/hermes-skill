---

name: jira-review
description: Review Jira tickets and Sub-tasks for implementation difficulty, assign Credit 1/2/3, update the Jira credit field on the reviewed Sub-task, and add the review result as a comment to the same Sub-task.
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Jira Review

## PURPOSE

Review Jira tickets and Sub-tasks for implementation difficulty.

Every reviewed ticket MUST receive exactly one difficulty Credit:

* Credit 1 — Easy
* Credit 2 — Medium
* Credit 3 — Hard

For Jira Sub-tasks, the AI MUST:

1. Review the actual Sub-task.
2. Assign Credit 1, 2, or 3.
3. Update the Jira field named `credit` on the SAME Sub-task.
4. Add the AI review comment to the SAME Sub-task.
5. Verify both Jira operations succeeded.

The review is NOT complete until the `credit` field has been updated successfully and the review comment has been created successfully.

---

# DIFFICULTY / CREDIT ASSESSMENT

Every Jira ticket MUST be assigned a difficulty credit.

Use ONLY these 3 values:

* Credit 1 — Easy
* Credit 2 — Medium
* Credit 3 — Hard

Do NOT use:

* Unknown
* Blank
* N/A
* Unassigned
* Any other Credit value

The final Credit MUST always be exactly `1`, `2`, or `3`.

---

## Credit 1 — Easy

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

---

## Credit 2 — Medium

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

---

## Credit 3 — Hard

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

---

# IMPORTANT CREDIT RULES

Do NOT determine Credit from:

* Ticket age
* Number of sprints
* Priority
* Status
* Number of comments
* Number of people involved
* Time spent
* Story points
* Original estimate
* Assignee

Determine Credit primarily from the actual implementation complexity described in the ticket.

Consider:

* Number of affected services/components
* Business logic complexity
* API/data-flow changes
* Database changes
* Integration complexity
* Technical dependencies
* Production risk
* Testing complexity
* Architectural impact

Credit represents implementation difficulty, NOT actual time spent.

If the ticket description is incomplete:

1. Assign the best Credit possible from the available information.
2. Do NOT use Unknown.
3. Explicitly mention missing information or assumptions in the review.

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
* Database requirements
* Integration requirements

The parent Story may be used as additional context.

However:

**The difficulty Credit MUST represent the implementation complexity of the specific Sub-task being reviewed.**

Do NOT automatically assign the same Credit as the parent Story.

---

# REQUIRED OUTPUT

Every ticket review MUST contain:

### DIFFICULTY

* **Credit:** 1 / 2 / 3
* **Level:** Easy / Medium / Hard
* **Reason:** <reason>
* **Complexity factors:**

  * <factor 1>
  * <factor 2>
  * <factor 3>

The Credit and Level MUST correspond exactly:

| Credit | Level  |
| -----: | ------ |
|      1 | Easy   |
|      2 | Medium |
|      3 | Hard   |

---

# JIRA CREDIT FIELD

The Jira field used to store the difficulty score is named:

`credit`

When reviewing a Jira Sub-task, the AI MUST update the `credit` field on the SAME Sub-task.

The value must be:

| Difficulty | `credit` value |
| ---------- | -------------: |
| Easy       |              1 |
| Medium     |              2 |
| Hard       |              3 |

The value stored in Jira MUST exactly match the final Credit assessment.

Do NOT store:

* `Easy`
* `Medium`
* `Hard`
* `Credit 1`
* `Credit 2`
* `Credit 3`

unless Jira field metadata explicitly requires a different representation.

---

# CREDIT FIELD UPDATE PROCESS

After determining the difficulty:

1. Identify the exact Sub-task issue key.
2. Identify the Jira field ID corresponding to the field named `credit`.
3. Update the `credit` field on the SAME Sub-task.
4. Verify that the Jira operation succeeded.
5. If necessary, retrieve the issue again and verify that the field contains the expected value.
6. Only after successful verification, add the review comment.

The AI MUST NOT guess the Jira custom field ID.

If Jira metadata exposes:

```text
credit → customfield_12345
```

then use:

```text
customfield_12345
```

for the update.

If the field ID is already known from the Jira environment, use the known field ID.

---

# JIRA TOOL USAGE

When Jira tools are available, the AI MUST actually execute the Jira operations.

Do NOT merely output a suggested API request.

## Step 1 — Read the Sub-task

Use:

```text
mcp__atlassian__getJiraIssue
```

to retrieve the actual Sub-task.

Review:

* Summary
* Description
* Parent
* Acceptance criteria
* Dependencies
* Linked issues
* Comments
* Technical information

---

## Step 2 — Determine Credit

Assign exactly one:

```text
1 = Easy
2 = Medium
3 = Hard
```

The Credit must be based on implementation difficulty.

---

## Step 3 — Identify the `credit` Field

The Jira UI field name is:

```text
credit
```

If the Jira API uses a custom field ID, identify the actual field ID from Jira metadata.

For example:

```text
credit → customfield_12345
```

Do NOT guess the ID.

---

## Step 4 — Update the Credit

Use:

```text
mcp__atlassian__editJiraIssue
```

to update the `credit` field on the SAME Sub-task.

Conceptually:

```json
{
  "issueKey": "GP2-1234",
  "fields": {
    "customfield_12345": 2
  }
}
```

The actual payload MUST follow the available Jira tool schema.

If the field is a numeric field, write:

```text
1
2
3
```

according to the assessment.

---

## Step 5 — Verify the Credit Update

After the update:

* Confirm Jira reports success.
* If necessary, retrieve the issue again.
* Confirm the `credit` field contains the expected value.

Do NOT continue as if successful when Jira reports an error.

---

## Step 6 — Add the Review Comment

After the Credit field has been successfully updated and verified, use:

```text
mcp__atlassian__addCommentToJiraIssue
```

to add the review comment to the SAME Sub-task.

---

## Step 7 — Verify the Comment

Confirm that Jira reports successful comment creation.

Only after both operations succeed is the Sub-task review complete.

---

# JIRA COMMENT

The review comment MUST be added to the SAME Jira Sub-task.

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

The Credit and Level in the comment MUST exactly match:

1. The final assessment.
2. The value written to the Jira `credit` field.

---

# COMMENTING RULES

When the reviewed issue is a Jira Sub-task:

1. Identify the exact Sub-task issue key.
2. Review the Sub-task.
3. Assign Credit 1, 2, or 3.
4. Update the `credit` field on the SAME Sub-task.
5. Verify the `credit` update.
6. Generate the review comment.
7. Add the comment to the SAME Sub-task.
8. Verify the comment creation.
9. Only then mark the review as complete.

The comment MUST be added to the Sub-task itself.

Do NOT add the review comment to:

* Parent Story
* Parent Epic
* Linked issue
* Another Sub-task

unless explicitly requested by the user.

---

# FAILURE HANDLING

## Credit field update fails

If the `credit` field update fails:

* Do NOT claim that the field was updated.
* Report that the Credit field update failed.
* Include the Jira/API error when available.
* The review is NOT complete.

Example:

```text
Credit: 2 — Medium

Credit field: FAILED
Reason: <Jira error>

Review comment: NOT POSTED
```

---

## Credit field cannot be identified

If the `credit` field cannot be identified:

* Do NOT guess the field ID.
* Inspect Jira metadata when possible.
* Report that the field could not be identified.
* Do NOT claim that the Credit was updated.
* The review is NOT complete.

---

## Comment creation fails

If the comment operation fails:

* Report that the comment operation failed.
* Include the error when available.
* Do NOT claim the comment was added.
* The review is NOT complete.

Example:

```text
Credit field: Updated
Review comment: FAILED
Reason: <Jira error>
```

---

## No Jira write capability

If no Jira write capability is available:

* Complete the difficulty assessment.
* Report that the `credit` field could not be updated.
* Report that the review comment could not be posted.
* Provide the exact Credit and comment text that should be posted.
* Do NOT claim anything was updated.

---

# MULTIPLE SUB-TASKS

When reviewing multiple Jira Sub-tasks:

1. Review each Sub-task independently.
2. Assign a separate Credit to each Sub-task.
3. Update the `credit` field on each Sub-task independently.
4. Verify each Credit update individually.
5. Add a separate review comment to each Sub-task.
6. Verify each comment operation individually.
7. Do not use one comment on the parent Story as a replacement for individual Sub-task comments.

Final output:

| Sub-task | Credit | Level  | Credit Field | Comment |
| -------- | -----: | ------ | ------------ | ------- |
| ABC-101  |      1 | Easy   | Updated      | Added   |
| ABC-102  |      2 | Medium | Updated      | Added   |
| ABC-103  |      3 | Hard   | Updated      | Added   |

Use `Updated` only when Jira confirms the `credit` field update succeeded.

Use `Added` only when Jira confirms successful comment creation.

If an operation fails, use `Failed` and explain why.

---

# COMPLETION CRITERIA

A Jira Sub-task review is complete ONLY when ALL of the following are true:

* The actual Sub-task has been analyzed.
* Credit 1, 2, or 3 has been assigned.
* The reason has been determined.
* Complexity factors have been identified.
* The Jira field named `credit` has been identified.
* The `credit` field has been updated on the SAME Sub-task.
* The `credit` field contains the correct value: 1, 2, or 3.
* The Credit field update has been successfully verified.
* The review comment has been created on the SAME Sub-task.
* The comment creation has been successfully verified.

If any step cannot be completed:

* Explicitly report which step failed.
* Do NOT claim the review is complete.
* Do NOT claim the Jira field or comment was updated unless Jira confirmed it.

---

# FINAL REVIEW RESPONSE

After completing the Jira operations, provide a concise result.

Example:

### Jira Review — GP2-1234

**Difficulty:** Credit 2 — Medium

**Reason:** Moderate backend logic change involving an existing API and consumer.

**Jira updates:**

* `credit`: `2` — Updated
* AI review comment: Added

**Status:** Review completed successfully.
