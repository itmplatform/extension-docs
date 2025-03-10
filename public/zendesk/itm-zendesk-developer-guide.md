# Zendesk Connector Developer Guide

The Zendesk Connector is an ITM Platform extension that integrates Zendesk with ITM Platform. It automatically creates a task in ITM Platform for each new Zendesk ticket and synchronizes statuses bidirectionally between the two systems. 

## Features

- **Automatic Task Creation**: Each new Zendesk ticket triggers the creation of a corresponding task in ITM Platform.
- **Bidirectional Status Synchronization**: Updates to task statuses in ITM Platform or ticket statuses in Zendesk are reflected in both systems, keeping information consistent.

## Configuration

To set up the Zendesk Connector, configure the following parameters:
1. **Activate Extension** (`isactive`): Enable/disable the extension. *(Checkbox, Optional)*
2. **Zendesk URL** (`url`): Your Zendesk account URL. *(String, Required)*
3. **Zendesk Email** (`email`): Email linked to Zendesk. *(String, Required)*
4. **Support Email** (`front_desk_support_email`): Zendesk support email. *(String, Required)*
5. **API Token** (`token`): Zendesk API token. *(String, Required)*
6. **Project ID** (`projectId`): ITM Platform project ID (Agile projects only). *(String, Required)*
7. **Initial Status ID** (`initial_status_id`): Task's initial status in ITM Platform. *(Dropdown, Required)*
8. **Ready to Reply Status ID** (`ready_to_reply_status_id`): Status for completed tickets. *(Dropdown, Required)*


## Setup Instructions

### 1. Enable the Extension

- Check the "Activate Extension" option to start syncing tasks between ITM Platform and Zendesk.

### 2. Enter Zendesk Details

- Provide your Zendesk account URL, email, and API token.
- Specify the support email used in Zendesk.

### 3. Configure ITM Platform Settings

- Enter the Project ID where tasks will be created. Ensure this is an Agile project.
- Select the initial status for new tasks.
- Choose the status that indicates a ticket is ready for reply or completed.

### 4. Save Configuration

- After entering all required information, save the configuration to activate the integration.

## Zendesk Configuration

Visit [Zendesk configuration](./zendesk-configuration.md) for detailed steps on setting up the Zendesk webhook and triggers.

# Step-by-Step Explanation
This guide explains how the **Features** section of the provided extension script works. Each Feature is defined by its trigger (an event, a webhook, or a scheduler) and includes one or more actions that execute in sequence.  Below is the `features` array from the JSON script. We'll break down each Feature and the actions it contains.  
## Feature 1: Task Update Event  
Feature Definition
```json 
{
    "trigger": "event",
    "connector": "76ee1244-77ea-454d-a5f6-c41bc98dd6a2",
    "AccountId": "@@AccountId@@",
    "entity": "Task",
    "description": "Task update Event, which will update 'Ticket' in Ticketing Platform",
    "condition": "input.diff != null && input.diff.StatusId != null && input.diff.StatusId.new != null && Convert.ToInt32(input.diff.StatusId.new) != 0 && Convert.ToInt32(input.userId) != -99",
    "event": "updated",
    "async": true,
    "actions": []
}
```

- **trigger**: `"event"` indicates this feature runs whenever a specific ITM Platform event occurs.
- **entity**: `"Task"` means it responds to task-related events.
- **event**: `"updated"` tells us it runs when a task is updated.
- **condition**: Ensures the actions only run if a task status was actually changed (`StatusId.new != null`) and if the updating user is not `-99`.
- **async**: `true` means the extension runs without blocking the main ITM Platform process.

Inside this Feature, there are several **actions**, each one defined as a `restcall` (an HTTP request to an external or internal API).
### Actions in Feature 1

1. **Action 1**: Retrieve Custom Fields
  
    ```json
    {
    "action": "restcall",
    "url": "@@TaskMS@@/v2/{{ input.accountId }}/Projects/{{ input.projectId }}/Tasks/{{ input.task.Id }}",
    "method": "POST",
    "description": "(1) Getting CustomFields values for project of ITM.",
    "output": "customfields",
    "payload": "{\"columns\":{\"$in\":[\"AllCustomFields\"]}}",
    "dataType": "application/json"
    }
    ```
  - **action**: `"restcall"`
  - **url**: Sends a POST request to ITM Platform’s Task service endpoint.
  - **payload**: Requests columns that include `"AllCustomFields"`.
  - **output**: `"customfields"` makes the retrieved data available for subsequent actions.
2. **Action 2**: Get Zendesk User List
    ```json
    {
    "action": "restcall",
    "url": "{{ config.url }}/api/v2/users/search.json?query={{ config.front_desk_support_email }}",
    "method": "GET",
    "payload": "",
    "description": "Get Ticketing User List",
    "output": "frontDeskSupportUsersOutput",
    "dataType": "application/json",
    "authentication": "basic",
    "username": "{{ config.email }}/token",
    "password": "{{ config.token }}"
    }
    ```  
  - **url**: Calls the Zendesk API to search for users with `front_desk_support_email`.
  - **authentication**: Uses Basic Authentication with the Zendesk token.
  - **output**: `"frontDeskSupportUsersOutput"` holds the list of found users.
3. **Action 3**: Update Ticket in Zendesk

    ```json
    {
    "action": "restcall",
    "url": "{{ config.url }}/api/v2/tickets/{{ int customfields.AllCustomFields.TicketId }}.json",
    "method": "PATCH",
    "description": "Update Ticket",
    "output": "ticketResponse",
    "condition": "input.diff != null && input.diff.StatusId != null && input.diff.StatusId.new != null && Convert.ToInt32(input.diff.StatusId.new) == {{config.ready_to_reply_status_id}} && frontDeskSupportUsersOutput != null && Convert.ToInt32(frontDeskSupportUsersOutput.count) > 0 && customfields.AllCustomFields != null && Convert.ToString(customfields.AllCustomFields.TicketId) != null",
    "payload": "{\"ticket\":{\"assignee_id\":\"{{ frontDeskSupportUsersOutput.users.0.id }}\",\"public\":false}}}",
    "dataType": "application/json",
    "authentication": "basic",
    "username": "{{ config.email }}/token",
    "password": "{{ config.token }}"
    }
    ```
  
  - **condition**: Only runs if the new Task status matches `ready_to_reply_status_id` and the user + ticket data is valid.
  - **payload**: A Zendesk `PATCH` call that assigns the ticket to the user found in Action 2.


## Feature 2: Webhook to Create or Update Task

**Feature Definition**

```json
{
    "trigger": "webhook",
    "connector": "76ee1244-77ea-454d-a5f6-c41bc98dd6a2",
    "AccountId": "@@AccountId@@",
    "description": "Create or Update Task in ITM from Ticketing platform",
    "condition": "input.ticket != null && Convert.ToInt32(input.ticket.id) != 0",
    "event": "updated",
    "async": true,
    "actions": [ ...]
}
```

- **trigger**: `"webhook"` means this feature is called from an external system (e.g., Zendesk) via a web request.
- **description**: Explains that it creates or updates a task in ITM Platform based on the incoming ticket data.
- **condition**: Only runs if the `ticket` object exists and `ticket.id` is a valid integer.


### Actions in Feature 2

1. **Action 1**: Search Project Team for the Ticket Assignee
    ```json
   {
    "action": "restcall",
    "url": "@@TaskMS@@/v2/{{ input.accountId }}/Projects/Search",
    "method": "POST",
    "condition": "input.ticket != null && Convert.ToInt32(input.ticket.id) != 0 && Convert.ToString(input.ticket.ticketAssigneeEmail) != \"\"",
    "description": "Getting Project Team Members",
    "output": "teamMembers",
    "payload": "{\"columns\":{\"$in\":[\"Team.TeamMembers\",\"Team.Managers\"]},\"Filter\":{\"id\":{{  config.projectId }},\"Team.TeamMembers.EmailAddress\":\"{{ input.ticket.ticketAssigneeEmail }}\"}}",
    "dataType": "application/json"
    }
    ```

  - **payload**: Searches the project specified by `config.projectId` for any team member matching the `ticketAssigneeEmail`.
  - **output**: `"teamMembers"` for subsequent use.
2. **Action 2**: Fallback Search if Action 1 Yields No Results
	 ```json
   {
    "action": "restcall",
    "url": "@@TaskMS@@/v2/{{ input.accountId }}/Projects/Search",
    "method": "POST",
    "condition": "input.ticket != null && Convert.ToInt32(input.ticket.id) != 0 && Convert.ToString(input.ticket.ticketAssigneeEmail) != \"\" && Convert.ToInt32(teamMembers.total) == 0",
    "description": "Getting Project Team Members",
    "output": "teamMembers",
    "payload": "{\"columns\":{\"$in\":[\"Team.TeamMembers\",\"Team.Managers\"]},\"Filter\":{\"id\":{{  config.projectId }},\"Team.Managers.EmailAddress\":\"{{ input.ticket.ticketAssigneeEmail }}\"}}",
    "dataType": "application/json"
    }
    ```
  - **condition**: Only runs if the team member search from Action 1 returned zero results.
  - **payload**: Looks for managers with the same email address instead.
3. **Action 3**: Validate Assignee Is on the Team
	```json
	{
    "action": "validate",
    "validateCondition": "input.ticket != null && Convert.ToInt32(input.ticket.id) != 0 && Convert.ToString(input.ticket.ticketAssigneeEmail) != \"\" && Convert.ToString(input.ticket.ticketAssigneeEmail) != Convert.ToString(config.front_desk_support_email) && (Convert.ToInt32(teamMembers.total) == 0)",
    "message": "The ticket assignee must be a member of the project team or the front-desk support email. Please update the assignee before proceeding."
    }
    ```
  - **action**: `"validate"` stops the process if the user is neither a regular team member nor a manager, nor the designated front-desk email.
  - **message**: Displays an error for the user.
4. **Action 4**: Get Project Task Custom Fields
    ```json
    {
    "action": "restcall",
    "url": "@@TaskMS@@/v2/{{ input.accountId }}/tasks/CustomFields",
    "method": "GET",
    "description": "Get Project Task Custom Fields",
    "output": "AllCustomFieldsOutput",
    "condition": "input.ticket != null && Convert.ToInt32(input.ticket.id) != 0 && Convert.ToString(input.ticket.ticketAssigneeEmail) != \"\"",
    "payload": "",
    "dataType": "application/json"
    }
    ```
  
  - **url**: Retrieves custom fields for tasks in ITM Platform.
  - **output**: `"AllCustomFieldsOutput"` for the next steps.
5. **Action 5**: Get Ticket Description
	```json
    {
        "action": "restcall",
        "url": "{{ config.url }}/api/v2/tickets/{{ input.ticket.id }}/comments.json?sort_order=desc",
        "method": "GET",
        "payload": "",
        "description": "Get All Custom Fields",
        "output": "ticketDescription",
        "dataType": "application/json",
        "authentication": "basic",
        "username": "{{ config.email }}/token",
        "password": "{{ config.token }}"
    }
    ```
  - **url**: Calls Zendesk to fetch the latest comments (which might include ticket details).
  - **output**: `"ticketDescription"` is the result.
6. **Action 6**: Update Existing ITM Platform Task
  
    ```json
    {
    "action": "restcall",
    "url": "@@TaskMS@@/v2/{{ input.accountId }}/Projects/{{ config.projectId }}/Tasks/{{ input.ticket.ITMTaskId }}?UserId=-99&LanguageId=@@LanguageId@@",
    "method": "PATCH",
    "description": "Update Existing Task",
    "output": "taskOutput",
    "condition": "input.ticket != null && Convert.ToInt32(input.ticket.id) != 0 && Convert.ToString(input.ticket.ticketAssigneeEmail) != \"\" && input.ticket.ITMTaskId != null && Convert.ToString(input.ticket.ITMTaskId) != \"\"",
    "payload": "{\"Name\":\"{{input.ticket.name}}\",\"StatusId\":\"{{ config.initial_status_id }}\",\"Details\":\"{{EscapeDoubleQuotes ticketDescription.comments.0.html_body }}\",\"TaskMembers\":\"{{ input.ticket.ticketAssigneeEmail }}\"}",
    "dataType": "application/json"
    }
    ```
  - **condition**: Runs only if the incoming data already has a valid `ITMTaskId`.
  - **payload**: Updates the task’s name, status, details, and team members.
7. **Action 7**: Create New Task (if no `ITMTaskId` found)
	```json
	{
    "action": "restcall",
    "url": "@@TaskMS@@/v2/{{ input.accountId }}/Projects/{{ config.projectId }}/Tasks?UserId=-99&LanguageId=@@LanguageId@@",
    "method": "POST",
    "description": "Create New Task",
    "output": "taskOutput",
    "payload": "{\"Name\":\"{{input.ticket.name}}\",\"StatusId\":\"{{ config.initial_status_id }}\",\"Details\":\"{{EscapeDoubleQuotes ticketDescription.comments.0.html_body}}\",\"TaskMembers\":\"{{ input.ticket.ticketAssigneeEmail }}\",\"CustomField\":[ {{#ifEquals AllCustomFieldsOutput.Name 'TicketId'}}  {{else}} {\"CustomFieldBaseId\":{{ mapArray AllCustomFieldsOutput 'Name' 'TicketId' 'Id'}},\"CustomFieldValue\":\"{{ input.ticket.id }}\",\"CustomFieldTypeName\":\"Text\"} {{/ifEquals}}],\"KindId\":3}",
    "dataType": "application/json"
    }
    ```
  
  - Creates a new task in ITM Platform if there’s no existing task (`ITMTaskId`) to update.
  - Sets the `"TicketId"` custom field to link back to Zendesk.
8. **Action 8**: Get Custom Fields in Zendesk
  
	```json
	{
    "action": "restcall",
    "url": "{{ config.url }}/api/v2/ticket_fields",
    "method": "GET",
    "payload": "",
    "description": "Get All Custom Fields",
    "output": "zendeskCustomFieldList",
    "dataType": "application/json",
    "condition": "taskOutput != null && Convert.ToInt64(taskOutput.Id) > 0",
    "authentication": "basic",
    "username": "{{ config.email }}/token",
    "password": "{{ config.token }}"
    }
    ```
  - **condition**: Only runs once a task has been created or updated in ITM Platform (`taskOutput.Id > 0`).
  - **output**: Holds a list of all custom fields in Zendesk.
9. **Action 9**: Update Ticket’s Custom Field with Task ID
    ```json
    {
    "action": "restcall",
    "url": "{{ config.url }}/api/v2/tickets/{{ input.ticket.id }}.json",
    "method": "PUT",
    "description": "Update Ticket Custom field",
    "output": "ticketResponse",
    "condition": "taskOutput != null && Convert.ToInt64(taskOutput.Id) > 0",
    "payload": "{\"ticket\":{\"custom_fields\":[{\"id\":{{ mapArray zendeskCustomFieldList.ticket_fields 'title' 'ITMTaskId' 'id' }},\"value\":\"{{ taskOutput.Id }}\"}]}}",
    "dataType": "application/json",
    "authentication": "basic",
    "username": "{{ config.email }}/token",
    "password": "{{ config.token }}"
    }
    ```
  - **payload**: Updates the Zendesk ticket’s custom field `ITMTaskId` to match the newly created or updated task in ITM Platform.
  - **condition**: Similar check to ensure a valid task ID.

## Summary

- **Feature 1** (triggered by `event` on Task update) pulls custom fields from ITM Platform, searches for a matching user in Zendesk, and updates the Zendesk ticket if a certain Task status is reached.
- **Feature 2** (triggered by `webhook` from Zendesk) either updates or creates a task in ITM Platform using data from the incoming Zendesk ticket. It also updates the Zendesk ticket with the newly created task ID in ITM Platform.

Each action within these features is a `restcall` or `validate`, with optional conditions, outputs, and payloads. These components work together to keep Zendesk tickets and ITM Platform tasks in sync.