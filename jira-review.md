---
name: jira-review
description: Review Jira tickets and assign difficulty credits using a 1-3 credit scale.
---
## DIFFICULTY / CREDIT ASSESSMENT

Every Jira ticket MUST be assigned a difficulty credit.

Use ONLY these 3 values:

- Credit 1 — Easy
- Credit 2 — Medium
- Credit 3 — Hard

Do NOT use "Unknown".
Do NOT leave the credit blank.

### Credit 1 — Easy

Assign Credit 1 when the task is relatively straightforward and has low implementation complexity.

Typical characteristics:
- Simple configuration change
- Small code change
- Single service / single component
- No significant architectural change
- No complex business logic
- No database migration or complicated data transformation
- Few dependencies
- Easy to test and verify
- Low risk

Examples:
- Add/update a configuration value
- Simple API field mapping
- Small validation change
- Simple monitoring/alert change
- Minor UI or backend adjustment

### Credit 2 — Medium

Assign Credit 2 when the task requires moderate implementation effort or involves multiple components.

Typical characteristics:
- Changes multiple files/components
- Changes API behavior or data flow
- Requires moderate business logic
- Requires database/schema changes without major migration complexity
- Requires integration with another service
- Requires several test cases
- Has moderate technical risk
- Requires investigation before implementation

Examples:
- Modify an existing API and update its consumer
- Add a new business rule
- Add a new integration with an existing internal service
- Moderate database changes
- Changes involving multiple services

### Credit 3 — Hard

Assign Credit 3 when the task has high technical complexity, high risk, or requires significant investigation/design.

Typical characteristics:
- Multiple services or systems involved
- Significant architecture/design changes
- Complex business logic
- Major database migration
- Difficult integration
- Significant backward-compatibility concerns
- High production risk
- Requires extensive testing
- Requires investigation/spike before implementation
- Multiple dependencies or external systems
- Unclear technical approach that requires design decisions

Examples:
- Major changes across multiple microservices
- New payment/integration flow
- Complex data migration
- Significant architecture change
- Changes affecting critical production flows

### IMPORTANT

Do NOT determine Credit from:
- Ticket age
- Number of sprints
- Priority
- Status
- Number of comments
- Number of people involved
- How long the ticket has been open

Determine Credit primarily from the **actual implementation complexity described in the ticket**.

If the ticket description is incomplete, estimate the credit based only on the available information and explicitly mention what information is missing.

The credit is an assessment of **implementation difficulty**, NOT actual time spent.

---

## REQUIRED OUTPUT

Every ticket review MUST contain this section:

### DIFFICULTY

- **Credit:** 1 / 2 / 3
- **Level:** Easy / Medium / Hard
- **Reason:** <short explanation based on the ticket>
- **Complexity factors:**
  - <factor 1>
  - <factor 2>
  - <factor 3>

The `Credit` value MUST always be exactly `1`, `2`, or `3`.
