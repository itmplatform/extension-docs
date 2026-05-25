# Task Completion Notification

The Task Completion Notification is an ITM Platform extension that automatically sends email notifications when a task is completed, alerting the team responsible for successor tasks that a dependency has been fulfilled.

## Features

- **Automatic Notification**: When a task is completed, the extension automatically identifies successor tasks linked by Finish-to-Start dependencies and notifies the people responsible.
- **Predecessor Status Summary**: Each email includes a table showing all predecessor tasks and their current completion status, giving recipients a clear picture of what is done and what is still pending.
- **Smart Recipient Selection**: Emails are sent to the successor task's managers and team members. If the successor task has no assignees, the project manager receives the notification instead.
- **Works Across All Projects**: The extension monitors task completions across all projects in the account.

## How It Works

1. A user changes a task's status in ITM Platform.
2. The extension checks whether the status actually changed and whether the new status is a completed status (`IsCompleted = true`).
3. If the task is completed, the extension looks up all Finish-to-Start dependencies where this task is the predecessor.
4. For each successor task found, it retrieves the task's team, predecessors, and start date.
5. It sends an email to the successor task's managers and team members (or the project manager as fallback) with a summary of all predecessor statuses.

### Example Email

> **Predecessor task completed: UI Design**
>
> The task **Backend Development** (planned start: 15/03/2026) has the following dependencies:
>
> | Predecessor task | Status |
> |---|---|
> | UI Design | **Completed** |
> | Requirements Analysis | Pending |
> | Security Review | **Completed** |
>
> When all predecessors are completed, the task can start.

## Configuration

To set up the extension, configure the following parameters:

1. **Activate Extension** (`isactive`): Enable or disable the extension. *(Checkbox, Optional)*
2. **API Key** (`apikey`): An ITM Platform API Key used for authentication. Found in **My Profile > API Key** in your ITM Platform account. *(String, Required)*

## Setup Instructions

### 1. Get Your API Key

- In ITM Platform, go to **My Profile > API Key**.
- Copy the key value.

### 2. Install and Configure the Extension

- Install the "Task Completion Notification" extension from the ITM Platform extension marketplace.
- Check the **Activate Extension** checkbox.
- Paste the API Key into the **API Key** field.
- Save the configuration.

### 3. Set Up Task Dependencies

The extension works with Finish-to-Start task dependencies. Make sure your projects have dependencies defined between tasks:

- In your project's Gantt chart or task list, create dependencies between tasks.
- The extension only triggers for **Finish-to-Start** (FS) dependency types.

## Prerequisites

- **ITM Platform version**: Requires the `Predecessors.IsCompleted` field in the Tasks API (available from the platform update that adds `IsCompleted` to the Predecessor model).
- **Task dependencies**: Projects must have Finish-to-Start dependencies defined between tasks.
- **Team assignments**: For notifications to reach the right people, successor tasks should have managers or team members assigned. If not, the project manager will be notified as a fallback.

## Step-by-Step Explanation

This section explains how the extension script works internally.

### Trigger

```json
{
    "trigger": "event",
    "entity": "Task",
    "event": "updated",
    "condition": "input.diff != null && input.diff.StatusId != null && input.diff.StatusId.old.ToString() != input.diff.StatusId.new.ToString()",
    "async": true
}
```

- **trigger**: `"event"` -- runs whenever a task event occurs in ITM Platform.
- **entity**: `"Task"` -- listens to task-related events.
- **event**: `"updated"` -- fires when a task is updated.
- **condition**: Uses the `diff` object from the event payload to check that the `StatusId` actually changed. This avoids processing task updates that don't involve a status change (e.g., name edits, date changes). The condition is null-safe. Uses `ToString()` for comparison because the diff values are JToken objects, and `Convert.ToInt32()` does not work reliably with JToken in Dynamic LINQ expressions.
- **async**: `true` -- runs without blocking the ITM Platform UI.

### Actions

#### 1. Authenticate

```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/@@AccountName@@/login/{{ config.apikey }}",
    "method": "GET",
    "output": "logininfo"
}
```

Authenticates using the configured API Key and stores the token for subsequent API calls.

#### 2. Verify Task Is Completed

```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/v2/@@AccountName@@/Projects/{{ input.projectId }}/Tasks/{{ input.task.Id }}",
    "method": "POST",
    "payload": "{\"Columns\": {\"$in\": [\"Id\", \"Name\", \"Status\"]}}",
    "output": "completedTask"
}
```

Fetches the task detail and checks `Status.IsCompleted`. If the task is not in a completed status, all subsequent actions are skipped via their condition `Convert.ToBoolean(completedTask.Status.IsCompleted) == true`.

**Note**: The `Changes.Status.new.IsCompleted` field in the event payload is unreliable (it can show `false` for completed statuses). Always verify via the REST API.

#### 3. Fetch Project Detail

```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/v2/@@AccountName@@/Projects/{{ input.projectId }}",
    "method": "POST",
    "payload": "{\"Columns\": {\"$in\": [\"Id\", \"Name\", \"Team\"]}}",
    "output": "projectDetail"
}
```

Retrieves the project by ID to get its name (for the email subject/body) and the project's team managers (used as fallback recipients when the successor task has no assignees).

#### 4. Fetch Task Dependencies

```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/@@AccountName@@/Project/{{ input.projectId }}/TaskDependencies",
    "method": "GET",
    "output": "dependencies"
}
```

Retrieves all task dependencies for the project. This is a v1 API endpoint (not v2). Each dependency includes `From` (predecessor task ID), `To` (successor task ID), `Type` (dependency type), `Lag`, and `LagUnit`.

#### 5. Loop Over Successors

```json
{
    "action": "loop",
    "loop": {
        "var": "dependencies",
        "output": "singleDep",
        "condition": "Convert.ToInt32(singleDep.From) == Convert.ToInt32(input.task.Id) && Convert.ToString(singleDep.Type) == \"2\""
    }
}
```

Iterates over all dependencies, filtering for those where:
- `From` matches the completed task (this task is the predecessor)
- `Type` is `"2"` (Finish-to-Start dependency)

For each matching dependency, the nested actions fetch the successor task detail and send emails.

#### 6. Fetch Successor Task Detail

```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/v2/@@AccountName@@/Projects/{{ input.projectId }}/Tasks/{{ singleDep.To }}",
    "method": "POST",
    "payload": "{\"Columns\": {\"$in\": [\"Id\", \"Name\", \"Team\", \"Predecessors\", \"StartDate\"]}}",
    "output": "successorTask"
}
```

Fetches the successor task including:
- `Team.Managers` and `Team.TeamMembers` -- email recipients
- `Predecessors` -- array of all predecessor tasks with `IsCompleted` status, used to build the status summary table
- `StartDate` -- shown in the email body

#### 7. Send Emails

Three email loops handle the recipient logic:

1. **Managers loop**: Sends to all managers of the successor task who have a valid email address.
2. **Team members loop**: Sends to all team members of the successor task who have a valid email address.
3. **Fallback loop**: If the successor task has no managers AND no team members, sends to the project managers instead.

The email body uses `{{#each successorTask.Predecessors}}` to render the status table, and `{{#if IsCompleted}}` to show each predecessor as "Completed" or "Pending".

## Limitations

- **Finish-to-Start only**: The extension only processes Finish-to-Start (FS) dependencies. Other dependency types (Start-to-Start, Finish-to-Finish, Start-to-Finish) are ignored.
- **Direct successors only**: Only immediate successor tasks are notified, not tasks further down the dependency chain.
- **No "all completed" gate**: The email is sent each time any predecessor is completed. It does not wait until all predecessors are completed before notifying.
- **Re-completion**: If a task is completed, reopened, and completed again, the notification will fire again with no indication that it is a re-completion.
