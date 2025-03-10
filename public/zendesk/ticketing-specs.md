##  ITM Platform ticketing sync extension specs
The extension will synchronize ticketing system tickets, such as Zendesk and Help Scout, with ITM Platform tasks. 

These are the generic specification. For the final code, refer to the [public folder](../../public/).

#### Basic terminology
- **Ticket** is always in the ticketing system (Zendesk, Help Scout[^4], aka **Ticketing**) and **task** in ITM Platform
- **Agent** or **ticket assignee** is the equivalent to ITM Platform **user** (typically a developer)
 
### General specs
 
- Each extension will synchronize with one project. Only agile projects
- Each Ticketing agent should have an equivalent ITM Platform user (both can be identified by the email)
- There are only two relevant statuses for the task in ITM Platform:
	- An `initial` one (typically `to do`) in which the task is created and when the ticket is reassigned in Zendesk
	- A `ready-to-reply` one (typically `done`) when the agent solves or provides more information to the front desk support team
	 Therefore, any intermediate status in ITM Platform will be ignored in the synchronization.
- The extension requires a custom field in the ITM Platform task called ` TicketId`, type string
- 
### Configuration
All fields are mandatory unless otherwise indicated.
- Zendesk to ITM Platform (webhook part):
	- Destination `Project Id`
	- `initial` project task status id [^11]
	- `ready-to-reply` project task status id [^11]
- ITM Platform to Zendesk (task `updated` event)
	- Zendesk required fields, such as URL, authentication (see `type: header` in the [Extension Configuration](https://github.com/itmplatform/extension-docs?tab=readme-ov-file#extension-configuration)) 
	- `front-desk-support-email` 

Zendesk extension configuration example:
```
+----------------------------------------------------+
|                  EXTENSION CONFIG                  |
+----------------------------------------------------+
| Zendesk -> ITM Configuration:                      |
|   + Project Id: [REQUIRED: numeric]        |
|   + initial_status_id: [REQUIRED: numeric]         |
|   + ready_to_reply_status_id: [REQUIRED: numeric]  |
|                                                    |
| ITM -> Zendesk Configuration:                      |
|   + Zendesk URL: [REQUIRED: string/URL]            |
|   + Zendesk Auth Header: [REQUIRED: key/token]     |
|   + front_desk_support_email: [REQUIRED: email]    |
|                                                    |
+----------------------------------------------------+

```
### Workflow

#### Ticketing >> ITM Platform (webhook)
* Webhook is called when a Ticketing ticket is created or updated.
* If the Ticketing webhook payload doesn't include all required fields, it will gather them[^5]
* Prechecks. These failing, it will throw a user-friendly error visible in the extension history and log, and stop
	* It must check if the ticket assignee agent  is present in the project team or is the  `front-desk-support-email`.
	* It must check whether the project tasks have the ` TicketId` associated. 
* It will check if there is a project task with the custom field ` TicketId` containing the ticket `Id`
	* If it does not exist, regardless of whether the ticket was created or updated, it will create the task with:
		* Name (`subject`), ` TicketId`, set the `inital` status 
	* Then, the recently created task or the existing task, will be updated with:
		* Name (overwrite existing to ensure match if changed in ITM Platform)
		* Description 
		* Status: 
			* If the ticket assignee is not `front-desk-support-email` (therefore is an agent/developer) set the status to `initial`
		* ` TicketId` build from the ticket Id and the ticketing system URL
		* If the ticket assignee is not  `front-desk-support-email`, assign the agent to the task team if it wasn't already[^8]
```
        +-----------------------+
        |       Ticketing       |
        | (e.g., Zendesk/HS)    |
        +----------+------------+
                   |
          Webhook fired on ticket
           creation/update event
                   |
                   v
       +---------------------------+
       |    Extension (T->ITM)    |
       |  (Ticket->ITM Platform)  |
       +------------+-------------+
                    | Validate inputs (agent, project, fields)
                    | Check if a task with  TicketId exists
                    |   - NO: create new task with 'name', 'TicketId', set 'initial' status
                    |   - ALWAYS: update existing task (name, desc, status)
                    | If agent ≠ front-desk-support-email, 
                    | ensure agent is assigned to task team
                    v
       +---------------------------+
       |        ITM Platform       |
       |       (Agile Project)     |
       +---------------------------+

```
### ITM Platform >> Ticketing (task event)
* `Updated` event is triggered for a task[^9]
* If the task doesn't have a filled-in ` TicketId` custom filed, it will stop, no logging or errors to avoid log clogging.
* The code should only run if the status has changed[^9] to the `ready-to-reply` status[^10]
* The ticket must change the assigned agent to `front-desk-support-email` [^12]

```
       +---------------------------+
       |       ITM Platform       |
       |       (Agile Project)    |
       +------------+-------------+
                    |
            Trigger on task change event
             ('updated')
                    |
             If status changed to 
              'ready-to-reply' and 
              TicketId is set:
                    v
       +---------------------------+
       |    Extension (ITM->T)     |
       | (ITM Platform->Ticketing) |
       +------------+--------------+
                    | Identify the ticket via  TicketId
                    | Update the ticket’s assignee to front-desk-support-email
                    v
       +---------------------------+
       |         Ticketing         |
       |    Assign ticket back     |
       | to front-desk-support     |
       +---------------------------+

```

## Task update event changes
To assess whether the task status has changed, we need to modify the extension interpreter so that the task's `updated` event payload includes the changes. To do so, we will include a `diff` property in the payload/input that will contain all properties that have changed. For example:
```json
{
    "accountId": "accountId",
    "projectId": "projectId",
    "task": {
        "Id": "taskId",
        "JiraTaskId": "JiraTaskId",
        "KindId": "taskKindId",
        "Name": "taskName"
    },
    "userId": "userId",
	"diff": {
		"StatusName":{"old": "To Verify", "new": "Done"},
		"EndDate": {"old": "2017-04-20", "new": "2017-04-21" },
		"StartDate":{"old": "2017-02-14", "new": "2017-02-15"}
	}
}
```
As an additional feature we can exclude properties when the `old` and `new` values are the same (in case the user called the API overwriting the same values).

For the final code, refer to the [public folder](../../public/).
