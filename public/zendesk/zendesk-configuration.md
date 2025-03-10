## Zendesk Configuration

This document outlines the steps to configure a trigger and webhook API in Zendesk to integrate with the ITM Platform - Zendesk custom extension.

### Configuration Steps

1. **Access Admin Center**  
   - Navigate to the **Admin Center** from the **Admin** tab.
   - Go to the **Apps and Integrations** tab.
   - Click on **Webhooks** under the **Actions and Webhooks** sub-tab.

2. **Create a Webhook**
   - Click the **Create Webhook** button.
   - Select **Trigger or Automation** as the connection type.
   - Provide the required details:
     - **Endpoint URL**: Enter the ITM Webhook Endpoint URL with the extension name appended. Example:  
       `https://new-api.itmplatform.com/revamping/v2/testsmarter/webhooks/itm-zendesk`
     - **Request Method**: Choose `POST`.
     - **Request Format**: Choose `JSON`.
     - **Authentication Type**: Select `Bearer Token`.
     - **Token**: Enter the token generated from the ITM Platform user profile API key field and save.

3. **Generate an API Token for the Extension**
   - Open **Zendesk API** under the **Apps and Integrations** tab.
   - Enable **Token Access** in the settings tab.
   - Generate a new access token and add it to the **API Token** field in the **Extension Config** section of ITM Platform.

4. **Create a Trigger for Task Creation or Updates**
   - Open the **Objects and Rules** tab and go to the **Triggers** sub-tab under **Business Rules**.
   - Create a new trigger and provide the required details:
     - **Name**: Enter any descriptive name.
     - **Trigger Category**: Select `Notifications`.
     - **Conditions**:
       - Under **Meet ALL of the following conditions**, add:  
         - `Category`: `Ticket Update Via`  
         - `Operator`: `is not`  
         - `Value`: `Web Service (API)`  
         (This prevents infinite loops between Zendesk and ITM Platform synchronization.)
       - Under **Meet ANY of the following conditions**, add:
         - `Category`: `Ticket > Ticket`  
         - `Operator`: `is`  
         - `Value`: `Created`  
         - `Value`: `Updated`
     - **Actions**:
       - Select `Notify by > Active Webhook`, and choose the webhook created in Step 2.
       - Add the following JSON payload:
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
       - Replace `ticket_field_24512485986333` with the **custom field ID** for ITM Task ID created in Step 5.
       - Save the trigger.

5. **Create a Custom Field for ITM Task ID**
   - Open the **Fields** sub-tab under the **Objects and Rules** tab.
   - Add a new **Text Field**.
   - Replace the last part of the `ticket_field_24512485986333` field ID in the payload with the newly created **Field ID**.

Once these steps are completed, the ITM Platform - Zendesk custom extension will be properly configured.
