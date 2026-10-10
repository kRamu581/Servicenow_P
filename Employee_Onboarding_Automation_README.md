# ServiceNow Employee Onboarding Automation

A step-by-step ServiceNow project to automate employee onboarding, from HR request submission through task assignment, approvals, notifications, and onboarding completion.

> **Project type:** ServiceNow HR / ITSM workflow automation  
> **Platform:** ServiceNow Personal Developer Instance (PDI) or development instance  
> **Suggested implementation:** Service Catalog + Flow Designer + tasks + notifications  
> **Note:** This is a practical reference implementation. Table availability, HRSD licensing, plugins, and field names vary by ServiceNow release and instance. Use the standard tables and capabilities available in your environment, or create scoped custom tables where needed.

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Project Objectives](#3-project-objectives)
4. [Proposed Solution](#4-proposed-solution)
5. [Scope and Assumptions](#5-scope-and-assumptions)
6. [Technology Stack](#6-technology-stack)
7. [Workflow](#7-workflow)
8. [Data Model](#8-data-model)
9. [Phase 1 — Prepare the Instance](#9-phase-1--prepare-the-instance)
10. [Phase 2 — Create the Onboarding Catalog Item](#10-phase-2--create-the-onboarding-catalog-item)
11. [Phase 3 — Configure Variables and Validation](#11-phase-3--configure-variables-and-validation)
12. [Phase 4 — Build the Flow in Flow Designer](#12-phase-4--build-the-flow-in-flow-designer)
13. [Phase 5 — Create Onboarding Tasks](#13-phase-5--create-onboarding-tasks)
14. [Phase 6 — Configure Approvals](#14-phase-6--configure-approvals)
15. [Phase 7 — Configure Notifications](#15-phase-7--configure-notifications)
16. [Phase 8 — Track Status and Completion](#16-phase-8--track-status-and-completion)
17. [Phase 9 — Testing](#17-phase-9--testing)
18. [Example Scenarios](#18-example-scenarios)
19. [Troubleshooting](#19-troubleshooting)
20. [Security and Best Practices](#20-security-and-best-practices)
21. [Future Enhancements](#21-future-enhancements)
22. [Expected Outcome](#22-expected-outcome)

---

## 1. Project Overview

Employee Onboarding Automation is a ServiceNow workflow project that coordinates the activities required when a new employee joins an organization.

Instead of managing onboarding through emails, spreadsheets, and manual follow-ups, a requester submits one onboarding request. ServiceNow then routes approvals, creates tasks for the responsible teams, sends notifications, and tracks progress until onboarding is complete.

### Example activities

- HR verifies employee details.
- The hiring manager confirms the joining information.
- IT prepares a laptop, account, and required access.
- Facilities prepares a desk or access badge.
- The employee receives joining instructions.
- HR confirms that all required tasks are complete.

The exact activities should be adjusted to the organization's policies.

## 2. Problem Statement

Employee onboarding often involves several teams, including HR, IT, the hiring manager, and facilities. When each team manages its work separately, requests can be missed, tasks may be delayed, and HR may not have a single view of onboarding progress.

**Problem:** There is no single automated workflow to coordinate onboarding tasks, approvals, notifications, and completion tracking.

## 3. Project Objectives

- Collect employee and joining details through a standard form.
- Route the request for the required approval.
- Automatically create tasks for HR, IT, and facilities.
- Notify task owners and relevant stakeholders.
- Track each task and the overall onboarding status.
- Reduce manual follow-ups and improve visibility.
- Maintain a record of actions taken during onboarding.

## 4. Proposed Solution

Build a ServiceNow catalog item that starts a Flow Designer flow when submitted. The flow validates the request, obtains the required approval, creates or assigns onboarding tasks, sends notifications, and updates the overall request status.

### High-level architecture

```text
Requester submits onboarding form
              |
              v
Service Catalog request is created
              |
              v
Flow Designer starts
              |
              v
Validate request details
              |
              v
Manager / HR approval (as required)
              |
              v
Create onboarding tasks
      /          |           \
     v           v            v
    HR           IT        Facilities
     \           |            /
      \          v           /
       Track task completion
              |
              v
All required tasks complete?
          /          \
        No            Yes
        |              |
  Wait / follow-up     v
                 Mark onboarding
                   complete
```

## 5. Scope and Assumptions

### Included in this reference project

- An onboarding request form.
- Flow Designer automation.
- Approval routing.
- Task assignment to responsible groups.
- Notifications.
- Status tracking and testing.

### Assumptions

- You have access to a ServiceNow development instance.
- You can create catalog items and flows, subject to instance permissions.
- You have or can create appropriate HR, IT, and facilities groups.
- Your organization has defined who can approve onboarding requests.
- You will use only test employee data while developing in a PDI.

### Licensing note

Some employee lifecycle and HR case features require **HR Service Delivery (HRSD)** or other licensed capabilities. This guide uses a Service Catalog and Flow Designer approach where possible. If your instance has HRSD, you can adapt the design to its employee lifecycle events, HR services, and HR cases. Do not assume every HRSD table or action is available in a standard PDI.

## 6. Technology Stack

| Component | Purpose |
|---|---|
| ServiceNow Service Catalog | Collect onboarding details |
| Flow Designer | Automate the workflow |
| Approvals | Obtain manager or HR approval |
| Catalog tasks / task records | Assign work to teams |
| Assignment Groups | Route work to HR, IT, and facilities |
| Notifications | Send updates and reminders |
| ServiceNow tables and reports | Track requests and progress |
| Optional: HRSD | Use employee lifecycle and HR case capabilities if licensed |

**Primary implementation approach:** Low-code automation using Service Catalog and Flow Designer. Script steps should be added only when a requirement cannot be met cleanly with standard flow actions.

## 7. Workflow

1. An authorized requester submits an onboarding request.
2. ServiceNow records the new employee's details and joining date.
3. Flow Designer validates required fields.
4. The flow requests approval from the configured manager or HR approver.
5. If rejected, the request is marked rejected and the requester is notified.
6. If approved, the flow creates tasks for the relevant teams.
7. Each team completes its assigned task and updates its status.
8. The flow checks whether all mandatory tasks are complete.
9. If tasks remain open, the request stays in progress and can trigger reminders.
10. When all mandatory tasks are complete, the request is marked complete and HR is notified.

> A joining date should not automatically grant system access. Access must follow the organization's identity, security, and authorization processes.

## 8. Data Model

Start with standard Service Catalog request records where appropriate. Depending on the chosen design and available tables, the request may be represented by a Request (`sc_request`), Requested Item (`sc_req_item`), and Catalog Tasks (`sc_task`). Confirm the actual relationships and available fields in your instance before configuring the flow.

### Suggested request fields

The following are suggested catalog variables, not guaranteed field names on a standard table.

| Variable | Type | Required? | Purpose |
|---|---|---:|---|
| `employee_name` | Single-line text | Yes | New employee's name |
| `employee_email` | Email / text | Yes | Work or contact email, per policy |
| `employee_id` | Single-line text | No | Employee identifier, if already assigned |
| `joining_date` | Date | Yes | Planned joining date |
| `department` | Choice / reference | Yes | Employee department |
| `job_title` | Single-line text | Yes | Role or designation |
| `manager` | Reference to user | Yes | Hiring manager |
| `location` | Choice / reference | Yes | Work location |
| `employment_type` | Choice | Yes | Full-time, contract, intern, etc. |
| `laptop_required` | Yes/No | Yes | Whether IT must prepare a laptop |
| `software_access` | Multi-line text / list | No | Approved software requirements |
| `additional_notes` | Multi-line text | No | Non-sensitive supporting details |

Collect only the information needed for onboarding. Avoid collecting sensitive identity or financial information unless the organization has an approved, secure process for it.

### Suggested task categories

| Task | Responsible group | Example deliverable |
|---|---|---|
| Verify employee details | HR | Details checked |
| Prepare laptop and account | IT | Device and account ready under approved process |
| Configure approved software access | IT / application owners | Required access completed |
| Prepare desk or badge | Facilities | Workspace ready |
| Complete joining checklist | HR | Required onboarding steps confirmed |

You can implement these as separate catalog tasks or as dedicated task records linked to the request. Use the approach supported by your instance and make sure every task is traceable to the parent request.

## 9. Phase 1 — Prepare the Instance

### Step 1: Check access

Confirm you can access:

- Service Catalog / Catalog Definitions.
- Flow Designer.
- Groups and users.
- Notifications.
- Request and catalog task records.

Navigation labels may differ by ServiceNow release and role.

### Step 2: Prepare assignment groups

Create or confirm groups such as:

- `HR Onboarding`
- `IT Onboarding`
- `Facilities`

Use existing organizational groups if available. Add test users to these groups so tasks can be assigned during development.

### Step 3: Prepare test users

Use test users for the requester, manager, HR agent, IT agent, and facilities agent. Do not use real employee information in a public demo or repository.

### Step 4: Sketch the approval policy

Before building the flow, decide:

- Who can submit an onboarding request?
- Who approves it?
- Is HR approval also required?
- What happens when the request is rejected?
- Which tasks are mandatory?
- Who can mark onboarding complete?

Write these rules down so the flow matches the expected process.

## 10. Phase 2 — Create the Onboarding Catalog Item

### Step 1: Open the catalog item area

Navigate to **Service Catalog → Catalog Definitions → Maintain Items** (or the equivalent module available in your instance).

### Step 2: Create a new item

Suggested configuration:

| Setting | Suggested value |
|---|---|
| Name | `Employee Onboarding Request` |
| Short description | `Submit and track a new employee onboarding request` |
| Catalog | Your organization's appropriate catalog |
| Category | HR Services or an appropriate available category |
| Active | Checked |
| Available for | Restrict to authorized requester groups as needed |

Use the catalog and category names available in your instance.

### Step 3: Add a description

Suggested description:

> Use this form to initiate onboarding for a new employee. Provide the required joining information and requested resources. The request will be routed for approval, and tasks will be assigned to the relevant teams.

### Step 4: Save

Save the catalog item, then open its variable configuration.

## 11. Phase 3 — Configure Variables and Validation

Add the variables listed in [Data Model](#8-data-model).

Recommended controls:

- Make essential fields mandatory.
- Use a date variable for the joining date.
- Use a reference variable for the manager when possible.
- Use choices for department, location, and employment type if controlled values are required.
- Show `laptop_required` and software-access questions only when relevant.
- Add clear instructions beside fields that users may misunderstand.

### Validation rules to consider

- Employee name must not be blank.
- Employee email must use the expected format.
- Joining date must follow organizational policy.
- Manager must be selected.
- Department and location must be valid choices.
- Requested equipment and software should follow approval policy.

Use catalog variable settings and catalog client scripts only where appropriate. Do not rely only on client-side validation for security; enforce important rules on the server side or within the flow as well.

### Test the form

1. Preview the catalog item.
2. Submit a test request with all required details.
3. Confirm that the request is created.
4. Check that the variables are stored and can be accessed by Flow Designer.
5. Try omitting a required field and verify that submission is blocked.

## 12. Phase 4 — Build the Flow in Flow Designer

### Step 1: Create a flow

1. Open **Flow Designer**.
2. Select **New → Flow**.
3. Name it `Employee Onboarding Automation`.
4. Add a clear description.
5. Select a trigger that starts the flow when the relevant catalog item is requested. In many catalog implementations, this is a **Service Catalog** trigger. Confirm the available trigger and configuration in your instance.
6. Configure the trigger for the Employee Onboarding Request item if the trigger supports item filtering.
7. Save the flow.

### Step 2: Read the request variables

Use the trigger data pills to access the requested item and its catalog variables. The exact data pill labels depend on the trigger and instance.

Map the employee name, joining date, manager, department, location, and optional equipment requirements to the actions that need them.

### Step 3: Validate request data

Add decision or validation steps for required details. If the request is incomplete or invalid:

1. Update the request to the appropriate rejected / incomplete state supported by your design.
2. Notify the requester with a clear reason.
3. Stop further task creation.

Do not create duplicate task sets if the flow is retriggered. Consider idempotency and verify the trigger runs only at the intended point in the request lifecycle.

### Step 4: Request approval

Add the approval action available in your Flow Designer instance, such as **Ask for Approval** when available.

Configure the approver using the manager variable or the defined HR approval group.

Handle both outcomes:

- **Approved:** continue to task creation.
- **Rejected:** update the request state, notify the requester, and end the workflow.

Define what happens if approval times out or the approver is unavailable.

### Step 5: Create team tasks

Use standard Flow Designer actions to create catalog tasks or other approved task records linked to the requested item. See [Phase 5](#13-phase-5--create-onboarding-tasks).

Create only the tasks required by the employee's selections and policy.

### Step 6: Wait for task completion

Choose a supported pattern for monitoring tasks:

- Wait for each mandatory task to reach a terminal state, if your flow actions support it; or
- Use a child flow / task completion flow to update the parent request; or
- Recalculate task completion through a controlled, supported workflow.

Avoid a simple delay as the only completion check. A task can remain open longer than expected.

### Step 7: Mark the onboarding request complete

Once every mandatory task is complete:

1. Update the parent request / requested item using valid states for the chosen record type.
2. Record the completion date if your design includes one.
3. Notify HR and the requester.
4. End the flow.

Use the state values available for your specific table; do not copy state numbers from another instance without checking them.

### Flow summary

```text
Catalog trigger
  -> Validate variables
  -> Approval
      -> Rejected: update state + notify + stop
      -> Approved:
          -> Create HR task
          -> Create IT task if required
          -> Create Facilities task if required
          -> Wait for / track mandatory task completion
          -> All complete?
              -> No: keep in progress / follow-up
              -> Yes: mark complete + notify
```

## 13. Phase 5 — Create Onboarding Tasks

The project can use Catalog Tasks (`sc_task`) when they fit the request structure in your instance. If you use another task table, ensure the records are related to the parent request and visible to the assigned teams.

For each task, set the fields supported by the selected table, such as:

- Short description.
- Description / work instructions.
- Assignment group.
- Assigned to, if known.
- Parent request or requested item relationship.
- Due date, if your process requires it.
- Initial state.

### Example tasks

**HR task**
- Short description: `Verify employee onboarding details`
- Assignment group: `HR Onboarding`
- Instructions: Confirm the employee details and joining information using the approved HR process.

**IT task**
- Short description: `Prepare employee equipment and approved access`
- Assignment group: `IT Onboarding`
- Instructions: Prepare required equipment and configure only approved accounts and permissions.

**Facilities task**
- Short description: `Prepare workspace and access badge`
- Assignment group: `Facilities`
- Instructions: Prepare the workspace and badge according to site policy.

### Conditional task creation

Use Flow Designer If / Else logic:

- If `laptop_required` is true, create the laptop preparation task.
- If additional software access was requested, create a review / fulfilment task for the appropriate application owner.
- If the employee's location requires facilities setup, create the facilities task.
- Create HR verification for every onboarding request if it is mandatory in your policy.

Use actual variable values from the trigger. Test both true and false branches.

## 14. Phase 6 — Configure Approvals

Define the approval policy before enabling the workflow.

### Example approval rules

| Condition | Example approver |
|---|---|
| Standard new hire | Hiring manager |
| Additional privileged access requested | Authorized application / security approver |
| HR verification required by policy | HR approver or HR group |

These are examples, not universal rules. Your organization must define its own approval matrix.

### Approval behavior

- On approval, continue with task creation.
- On rejection, record the result and notify the requester.
- On timeout, follow a defined escalation or cancellation policy.
- Do not treat an unanswered approval as approval.
- Keep an audit trail of approval outcomes.

## 15. Phase 7 — Configure Notifications

Use ServiceNow notifications or the notification actions available in Flow Designer.

Suggested notifications:

| Event | Recipient | Purpose |
|---|---|---|
| Request submitted | Requester | Confirm receipt |
| Approval requested | Approver | Request a decision |
| Request rejected | Requester | Explain the outcome |
| Task assigned | Assignment group / assignee | Explain the required work |
| Joining date approaching | Relevant task owner / HR | Remind about pending work |
| All tasks completed | HR / requester as appropriate | Confirm onboarding completion |

### Example notification messages

**Request submitted**

Subject: `Employee onboarding request received`

Body:

```text
Your employee onboarding request has been received.

Employee: ${employee_name}
Joining date: ${joining_date}

The request will be reviewed and routed to the appropriate teams.
Please refer to the ServiceNow request for status updates.
```

Replace placeholders with actual notification template fields or supported data pills. The `${...}` values above are illustrative and may not be valid in every notification configuration.

**Task assigned**

Subject: `Onboarding task assigned`

Body:

```text
A new onboarding task has been assigned to your team.

Task: [Task short description]
Employee: [Use only the minimum identifying details allowed by policy]
Due date: [Task due date, if configured]

Open the task in ServiceNow and update its state when work is complete.
```

Keep sensitive employee information out of email bodies unless the organization's policy explicitly permits it.

## 16. Phase 8 — Track Status and Completion

Choose status values that are valid for the record type used in your implementation. Do not assume the Request, Requested Item, and Catalog Task tables share the same state choices.

Suggested business-level lifecycle:

1. **Submitted** — request created.
2. **Awaiting Approval** — waiting for an authorized decision.
3. **Rejected** — approval declined.
4. **In Progress** — approved and tasks are being completed.
5. **Completed** — all mandatory onboarding tasks are finished.
6. **Cancelled** — cancelled according to the defined policy.

These are conceptual labels. Map them to supported states and choice values in your instance.

### Completion criteria

Mark onboarding complete only when:

- Required approvals are complete.
- All mandatory tasks are complete.
- Any conditional tasks required by the request are complete.
- HR has performed any required final verification.

Optional reporting ideas:

- Number of active onboarding requests.
- Requests awaiting approval.
- Overdue onboarding tasks.
- Average time from submission to completion.
- Requests grouped by joining month, department, or location.

Ensure reports and dashboards follow employee-data access restrictions.

## 17. Phase 9 — Testing

Test the flow in a development instance before using real onboarding data.

### Test cases

| Test | Input / situation | Expected result |
|---|---|---|
| 1. Valid request | All mandatory fields filled | Request is created |
| 2. Missing field | Required variable omitted | Form validation prevents submission |
| 3. Approval granted | Approver approves | Task creation begins |
| 4. Approval rejected | Approver rejects | Request is rejected; requester is notified |
| 5. Laptop required | Laptop variable is true | IT equipment task is created |
| 6. Laptop not required | Laptop variable is false | Conditional laptop task is not created |
| 7. Software access requested | Additional access is requested | Appropriate review / fulfilment task is created |
| 8. One task remains open | Other tasks complete | Onboarding is not marked complete |
| 9. All mandatory tasks complete | All required tasks reach completion | Request is marked complete |
| 10. Duplicate / retrigger test | Flow starts again for same request | Duplicate task creation is prevented or safely handled |
| 11. Notification test | Submission, assignment, rejection, completion | Correct recipients receive the expected messages |
| 12. Access test | Unauthorized user attempts to access records | ACLs prevent unauthorized access |

### Testing checklist

- [ ] Catalog item is available only to intended users.
- [ ] Mandatory variables are validated.
- [ ] Flow starts for the correct catalog item.
- [ ] Approval goes to the correct approver.
- [ ] Rejection prevents fulfilment tasks from being created.
- [ ] Tasks are assigned to the correct groups.
- [ ] Conditional tasks are created correctly.
- [ ] The parent request is not completed while mandatory tasks remain open.
- [ ] Notifications contain correct data and recipients.
- [ ] A repeated trigger does not create duplicate tasks.
- [ ] Unauthorized users cannot access employee data.
- [ ] Error paths and flow execution details have been reviewed.

## 18. Example Scenarios

### Scenario A: Standard employee onboarding

1. HR submits a request with employee details.
2. The hiring manager approves.
3. ServiceNow creates HR, IT, and facilities tasks.
4. Teams complete their tasks.
5. The flow checks the mandatory tasks.
6. The request is marked complete and HR is notified.

### Scenario B: Laptop not required

1. The request indicates that no laptop is required.
2. The manager approves.
3. HR and facilities tasks are created as needed.
4. The conditional laptop task is skipped.
5. Onboarding completes after all remaining mandatory tasks are complete.

### Scenario C: Approval rejected

1. A requester submits the form.
2. The manager rejects the request.
3. The flow records the rejection.
4. The requester receives a notification.
5. Fulfilment tasks are not created.

### Scenario D: IT task is delayed

1. HR and facilities tasks are complete.
2. The IT task remains open.
3. The parent request stays in progress.
4. A reminder or escalation runs according to policy.
5. The request completes only after the mandatory IT task is completed.

## 19. Troubleshooting

| Problem | Checks |
|---|---|
| Catalog item is not visible | Check Active, catalog/category, user criteria, and roles |
| Flow does not trigger | Check the trigger type, catalog item filter, activation, and flow execution details |
| Variables are empty in the flow | Confirm the correct trigger data pills and variable names |
| Approval is not created | Check approver mapping, approval action configuration, permissions, and flow execution |
| Tasks are not created | Check the create-record action, required fields, reference values, and execution errors |
| Tasks are assigned to the wrong team | Check group references and the assignment-group data pill |
| Request completes too early | Review the task completion condition and ensure every mandatory task is checked |
| Duplicate tasks appear | Review trigger conditions, retries, and duplicate prevention logic |
| Notifications are missing | Check notification activation, recipients, event conditions, email configuration, and logs |
| State update fails | Confirm the correct table and allowed state choice values |
| Flow cannot access HR features | Check installed plugins, licensing, roles, scope, and cross-scope permissions |

## 20. Security and Best Practices

1. **Least privilege:** Restrict who can submit requests, approve them, and view employee information.
2. **Protect employee data:** Collect only necessary information and apply appropriate ACLs and retention rules.
3. **Use test data:** Do not place real employee personal information in screenshots, public repositories, or demos.
4. **Approval before fulfilment:** Require the correct approvals before creating tasks that provision equipment or access.
5. **Follow access policy:** Do not automatically grant privileged access from free-text requests. Use authorized identity and access-management processes.
6. **Avoid hardcoded identifiers:** Prefer references to users and groups rather than environment-specific sys_ids in reusable examples.
7. **Prevent duplicates:** Make the flow safe against retries and repeated triggers.
8. **Handle errors:** Review Flow Designer execution details and provide a clear recovery path.
9. **Test upgrades and scope access:** Confirm that the selected actions and tables are supported in your release.
10. **Document ownership:** Define who maintains the flow, catalog item, task templates, approval rules, and notifications.

## 21. Future Enhancements

Once the core project works, consider adding:

- SLA or target completion dates for onboarding tasks.
- Automatic reminders before the joining date.
- Escalations for overdue tasks.
- A dashboard for HR and onboarding coordinators.
- Reusable task templates by department or location.
- Conditional workflows for contractors, interns, and full-time employees.
- Integration with approved identity-management or asset-management systems.
- HRSD Employee Lifecycle Events, if licensed and available.
- A knowledge article with onboarding instructions.
- Reporting on onboarding duration and common bottlenecks.

Build and test enhancements one at a time, starting with a working baseline.

## 22. Expected Outcome

After successful implementation, ServiceNow provides a consistent onboarding process in which a request is submitted once, routed for approval, divided into trackable team tasks, and monitored through completion.

### Deliverables

- Employee Onboarding Request catalog item.
- Employee and joining-detail variables.
- `Employee Onboarding Automation` flow.
- Approval handling for approved and rejected requests.
- HR, IT, and facilities task creation.
- Notifications for key workflow events.
- Status and completion tracking.
- Test cases and troubleshooting notes.

### Project summary

**Problem:** Onboarding tasks are coordinated manually across multiple teams.

**Solution:** Use ServiceNow Service Catalog and Flow Designer to coordinate approvals, task assignment, notifications, and completion tracking.

**Result:** A more consistent and visible onboarding workflow, with less manual follow-up and a clear record of task progress.

---

## Before sharing this README

- Replace example group names with names available in your instance.
- Confirm the catalog trigger, task table, state values, and approval actions in your ServiceNow release.
- Remove any organization-specific information that should not be public.
- Do not claim the project is deployed or tested unless you have verified it in your own instance.
