##  Zendesk Configuration
Specification explain about step to configure trigger and webhook API in zendesk

### Configuration Steps
- Go to Admin Center from admin tab:
	- Go to `Apps and integerations` tab
	- Click on `Webhooks` option in `Actions and webhooks` sub tab
    - Create a new webhook from by clicking `Create Webhook` button
    	-Choose `Trigger or automation` as a way to connect
        - Provide required fields detail like Name, Endpoint URL, Request Method Type, Request format and authentication type
            - Enter ITM Webhook Endpoint URL with Extension Name at it end e.g `https://new-api.itmplatform.com/revamping/v2/testsmarter/webhooks/itm-zendesk` 
            - Choose `POST` as Request Method Type
            - Choose `JSON` as Request Format
            - Select `Bearer token` as Authentication Type
            - Provide token in `Token` textbox that is generated from our platform like API Key from user profile tab and save
    - Second step is to create API token that will use by Extension to update tickets
        - Open `Zendesk API` under `Apps and integerations` tab
        - Enable token access from setting tab
        - Generate new access token and add that token in Extension Config in ITM Platform in API Token field
    - Third step is to create a trigger for Create or Update Task.
        - Open `Objects and rules` tab and go to `Triggers` sub tab under Business rules
        - Create a new trigger with providing all the required fields
            - Provide any name and select `Notifications` as Trigger Category
            - Add two ANY conditions by selecting `Ticket>Ticket` as Category, `is` as operator and Value as `Created` and `Updated` each
            - In Actions section select `Notify by > Active Webhook` and Webhook that you created in First step as a value
            - add a payload for example
            ```json
            {
                "ticket": {
                    "id": {{ticket.id}} ,
                    "name":"{{ticket.title}}",
                    "ticketURL" : "{{ticket.url}}",
                    "ticketRequesterEmail":"{{ticket.requester.email}}",
                    "ticketAssigneeEmail":"{{ticket.assignee.email}}",
                    "status":"{{ticket.status}}",
                    "ITMTaskId": "{{ticket.ticket_field_24512485986333}}"
                }
            }
            ```
            - Replace `ticket_field_24512485986333` last id part with your custom field created in zendesk for ITM Task Id in following step and save trigger
    - Last step is to create a Custom field for ITM Task Id
        - Open `Fields` sub tab under `Object and rules` tab
        - Add a new text field and replace last part of ticket field object in payload with its `Field ID`