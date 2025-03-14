# Help Scout Connector Developer Guide

The Help Scout Connector is an ITM Platform extension that integrates Help Scout with ITM Platform. It automatically creates a task in ITM Platform for each new Help Scout ticket and synchronizes statuses bidirectionally between the two systems.

## Features

- **Automatic Task Creation**: Each new Help Scout ticket triggers the creation of a corresponding task in ITM Platform.
- **Bidirectional Status Synchronization**: Updates to task statuses in ITM Platform or ticket statuses in Help Scout are reflected in both systems, keeping information consistent.

## Configuration

To set up the Help Scout Connector, configure the following parameters:

1. **Activate Extension** (`isactive`): Enable/disable the extension. *(Checkbox, Optional)*
2. **Help Scout URL** (`url`): Your Help Scout account URL. *(String, Required)*
3. **Support Email** (`front_desk_support_email`): Help Scout support email. *(String, Required)*
4. **Client ID** (`client_id`): Client ID generated by Help Scout. *(String, Required)*
5. **Client Secret** (`client_secret`): Client Secret generated by Help Scout. *(String, Required)*
6. **Project ID** (`projectId`): ITM Platform project ID (Agile projects only). *(String, Required)*
7. **Initial Status ID** (`initial_status_id`): Task's initial status in ITM Platform. *(Dropdown, Required)*
8. **Ready to Reply Status ID** (`ready_to_reply_status_id`): Status for completed tickets. *(Dropdown, Required)*

## Setup Instructions

### 1. Enable the Extension

- Check the "Activate Extension" option to start syncing tasks between ITM Platform and Help Scout.

### 2. Enter Help Scout Details

- Provide your Help Scout account URL, support email, Client ID, and Client Secret.

### 3. Configure ITM Platform Settings

- Enter the Project ID where tasks will be created. Ensure this is an Agile project.
- Select the initial status for new tasks.
- Choose the status that indicates a ticket is ready for reply or completed.

### 4. Save Configuration

- After entering all required information, save the configuration to activate the integration.

## Help Scout Configuration

Visit [Help Scout configuration](./helpscout-configuration.md) for detailed steps on setting up Help Scout's webhook and triggers.

# Step-by-Step Explanation
This guide explains how the **Features** section of the provided extension script works. Each Feature is defined by its trigger (an event, a webhook, or a scheduler) and includes one or more actions that execute in sequence.

## Feature 1: Task Update Event

### Feature Definition
```json
{
    "trigger": "event",
    "connector": "216470e2-73ac-4110-bf9c-9289d0cc65db",
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

### Actions in Feature 1

1. **Retrieve Custom Fields**
  
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

2. **Get Help Scout Authentication Token**
  
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

3. **Retrieve Ticketing User List**
  
    ```json
    {
    "action": "restcall",
    "url": "{{ config.url }}/v2/users?email={{ config.front_desk_support_email }}",
    "method": "GET",
    "description": "Get Ticketing User List",
    "output": "frontDeskSupportUsersOutput",
    "dataType": "application/json",
    "headers": "{\"Authorization\": \"Bearer {{authToken.access_token}}\"}"
    }
    ```

4. **Update Ticket in Help Scout**
  
    ```json
    {
    "action": "restcall",
    "url": "{{ config.url }}/v2/conversations/{{ customfields.AllCustomFields.TicketId }}",
    "method": "PATCH",
    "description": "Update Ticket",
    "output": "ticketResponse",
    "condition": "input.diff != null && input.diff.StatusId.new == {{config.ready_to_reply_status_id}}",
    "payload": "{\"op\":\"replace\",\"path\":\"/assignTo\",\"value\":\"{{ frontDeskSupportUsersOutput._embedded.users.0.id }}\"}",
    "dataType": "application/json",
    "headers": "{\"Authorization\": \"Bearer {{ authToken.access_token }}\"}"
    }
    ```

## Feature 2: Webhook to Create or Update Task

### Feature Definition

```json
{
    "trigger": "webhook",
    "connector": "216470e2-73ac-4110-bf9c-9289d0cc65db",
    "AccountId": "@@AccountId@@",
    "description": "Create or Update Task in ITM from Ticketing platform",
    "condition": "input.id != null && Convert.ToInt64(input.id) != 0",
    "event": "updated",
    "async": true,
    "actions": [ ... ]
}
```

- **trigger**: `"webhook"` means this feature is called from an external system (e.g., Help Scout) via a web request.
- **description**: Creates or updates a task in ITM Platform based on the incoming ticket data.
- **condition**: Only runs if the `id` exists and is a valid integer.

### Actions in Feature 2

- Searches for project team members matching the ticket assignee.
- Validates if the assignee is part of the project team.
- Retrieves task custom fields from ITM Platform.
- Creates or updates a task in ITM Platform.
- Updates Help Scout ticket fields to link the corresponding task.

## Summary

- **Feature 1** (triggered by `event` on Task update) updates Help Scout tickets when task statuses change in ITM Platform.
- **Feature 2** (triggered by `webhook` from Help Scout) creates or updates ITM Platform tasks based on Help Scout ticket data.

This ensures both systems remain synchronized efficiently.
