## Zendesk Configuration

This specification explains the steps to configure a trigger and webhook API in Zendesk to use the ITM Platform - Zendesk custom extension.

### Configuration Steps

1. **Go to Admin Center from the Admin tab:**
   - Navigate to the `Apps and Integrations` tab.
   - Click on the `Webhooks` option under the `Actions and Webhooks` sub-tab.
   - Create a new webhook by clicking the `Create Webhook` button.
     - Choose `Trigger or automation` as the connection type.
     - Provide the required details:
       - **Endpoint URL:** Enter the ITM Webhook Endpoint URL with the extension name at the end, e.g.,  
         `https:/api.itmplatform.com/v2/{yourcompany}/webhooks/itm-zendesk`
       - **Request Method Type:** Select `POST`.
       - **Request Format:** Choose `JSON`.
       - **Authentication Type:** Select `Bearer token`.
       - **Token:** Enter the token in the `Token` textbox, which is generated from our platform, like the API Key from the user profile tab. Save the webhook.

2. **Create an API token for the extension to update tickets:**
   - Open `Zendesk API` under the `Apps and Integrations` tab.
   - Enable token access from the settings tab.
   - Generate a new access token and add it to the `API Token` field in the ITM Platform extension configuration.

3. **Create a trigger for creating or updating a task:**
   - Open the `Objects and Rules` tab and navigate to the `Triggers` sub-tab under `Business Rules`.
   - Create a new trigger by filling in the required fields:
     - Provide a name and select `Notifications` as the Trigger Category.
     - Add two `ANY` conditions:
       - Select `Ticket > Ticket` as the Category, `is` as the operator, and the value as `Created` and `Updated`.
     - In the `Actions` section:
       - Select `Notify by > Active Webhook` and choose the webhook created in step 1.
       - Add a payload, for example:

       ```json
       {
           "ticket": {
               "id": "{{ticket.id}}",
               "name": "{{ticket.title}}",
               "ticketURL": "{{ticket.url}}",
               "ticketRequesterEmail": "{{ticket.requester.email}}",
               "ticketAssigneeEmail": "{{ticket.assignee.email}}",
               "status": "{{ticket.status}}",
               "ITMTaskId": "{{ticket.ticket_field_24512485986333}}"
           }
       }
       ```

       - Replace `ticket_field_24512485986333` with your custom field ID created in Zendesk for the ITM Task ID.

4. **Create a custom field for the ITM Task ID:**
   - Open the `Fields` sub-tab under the `Objects and Rules` tab.
   - Add a new text field and replace the last part of the `ticket_field` object in the payload with its `Field ID`.
