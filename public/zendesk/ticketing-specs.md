##  ITM Platform ticketing sync extension specs
The extension will synchronize ticketing system tickets with ITM Platform tasks. We will initially work on Zendesk, then likely Help Scout. The extension will be available for all clients and the code will be public in the [extension docs](https://github.com/itmplatform/extension-docs) 
#### Basic terminology
- **Ticket** is always in the ticketing system (Zendesk, Help Scout[^4], aka **Ticketing**) and **task** in ITM Platform
- **Agent** or **ticket assignee** is the equivalent to ITM Platform **user** (typically a developer)
### Extension MVP
 The MVP idea is to have ITM Platform synchronized so that when a ticket is assigned to an agent in Ticketing, it will create or update a task in a specific project in ITM Platform, and assign it to a user.  When the task status is updated in ITM Platform[^10], it will update the Ticketing ticket.
 Therefore it will have two triggers: `webhook` for *Zendesk> ITM Platform* and `event` for *ITM Platform > Zendesk*.
 
### General specs
 
- Each extension will synchronize with one project. Initially, only agile projects (no waterfall, no services) [^0]
- Each Ticketing agent should have an equivalent ITM Platform user (both can be identified by the email)
- There are only two relevant statuses for the task in ITM Platform:
	- An `initial` one (typically `to do`) in which the task is created and when the ticket is reassigned in Zendesk
	- A `ready-to-reply` one (typically `done`) when the agent solves or provides more information to the front desk support team
	 Therefore, any intermediate status in ITM Platform will be ignored in the synchronization.
- The extension requires a custom field in the ITM Platform task called `TicketId`, type string, to store the ticket Id[^3]. See note about types [^1] and about read-only/hidden [^2]
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
	* It must check whether the project tasks have the `TicketId` associated. 
* It will check if there is a project task with the custom field `TicketId` containing the ticket `Id`
	* If it does not exist, regardless of whether the ticket was created or updated, it will create the task with:
		* Name (`subject`), `TicketId`, set the `inital` status 
    * Once task is created or updated, it will automatically sync Task Id in Ticketing platform with a custom field that created in specific ticketing platform e.g `ITMTaskId`
	* Then, the recently created task or the existing task, will be updated with:
		* Name (overwrite existing to ensure match if changed in ITM Platform)
		* Description (see how[^6]), 
		* Status: 
			* If the ticket assignee is not `front-desk-support-email` (therefore is an agent/developer) set the status to `initial`
		* `TicketId` build from the ticket Id of ticketing system
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
                    | Check if a task with TicketId exists
                    |   - NO: create new task with 'name', 'TickertURL',  , set 'initial' status
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
* If the task doesn't have a filled-in `TicketId` custom filed, it will stop, no logging or errors to avoid log clogging.
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
                    | Identify the ticket via TicketId
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

## Footnotes

[^0]: Because waterfall task status are general, and activities too plus which services are v1. This also limits only one extension per client (one Zendesk account, one ITM Platform account) 

[^1]: To prevent all projects having tasks with that custom field, the project should be of a specific type (e.g., `Ticketing`). We currently don't have a way to assign custom fields to specific projects. We also don't have a way to make it read-only.

[^2]: Check whether a hidden custom field can be accessed via API

[^3]: The custom field will be of type string since we don yet have URL custom fields. See [# Standard third-party connection field #885](https://github.com/itmplatform/backend-master/issues/885) for more on this.

[^4]: In helpscout tickets are called conversations. See [API](https://developer.helpscout.com/mailbox-api/endpoints/conversations/get/) 

[^5]:  [Zendesk API](https://developer.zendesk.com/api-reference/webhooks/webhooks-api/webhooks/) seems to have a rich payload, where as int he [Helpscout API](https://developer.helpscout.com/mailbox-api/endpoints/webhooks/create/) the payload does not contain the changed entity but rather just a resource URI that can be used to fetch the entity 

[^6]: Zendesk's `description` field  contains only the initial comment of a ticket, while Help Scout's `threads` array encompasses all messages within a conversation. Therefore to have all Zendesk comments you must use [Ticket Comments API](https://developer.zendesk.com/api-reference/ticketing/tickets/ticket_comments/) 

[^7]: We don't prevent that there could be more than one task with the same `TicketId`, so we should take the first one.

[^8]: To do things well, when a ticket changes agents, we should remove the old one. But since we admit multiple task members, for now, we can leave it as is.

[^9]: See [Task update event changes](#task-update-event-changes) to determine if the status changed to the desired status `ready-to-reply`.

[^10]: Basic MVP. This should quickly include task Progress Reports, as they will be added as an internal note in Ticketing, which would 

[^11]: The project task status `ids` should later on come from a dropdown, ideally filtered by project. For a user is impossible to know the status `id`.

[^12]: Zendesk: Supports direct assignment of tickets using the` assignee_email` field. Help Scout: Requires retrieving the agent's user ID via the List Users API before assigning a conversation.