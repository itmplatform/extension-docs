# Example 6. Weekly AI Project Summary
## Send a weekly email with an AI-generated summary of a specific project

This example demonstrates how to use the `llm` action to generate content using Artificial Intelligence.

The full code is in the JSON file included in this same folder.

### Desired functionality

Every week, the extension will:
1.  Log in to ITM Platform.
2.  Retrieve data for a specific project (defined in configuration).
3.  Send the project data to an LLM to generate a summary.
4.  Email the summary to a configured address.

## Features and actions

The trigger is set to `scheduler`, meaning it will run based on a time interval.

```json
"trigger": "scheduler",
"description": "Weekly AI Project Summary",
```

The actions sequence is as follows:

1.  **Login**: Authenticates with ITM Platform API.
2.  **Search**: Retrieves the specific project using the `restcall` action.
    - We use the `filter` property in the payload to find the project by its ID: `"filter": {"Id": {{ int config.project_id }} }`.
3.  **LLM**: Generates the summary using the retrieved data.
    ```json
    {
        "action": "llm",
        "description": "Generate Project Summary",
        "input": "Summarize the project described in the provided JSON using PMI-style terminology and structure. {{json projectData.list.First}}. I want to send it in an email so don't suggest anything.",
        "output": "ai_response"
    }
    ```
    The `input` field contains the prompt for the AI. We use `{{json projectData.list.First}}` to pass the project data found in the previous step.
4.  **Email**: Sends the generated summary.
    ```json
    {
        "action": "email",
        "to": "{{ config.email_address }}",
        "subject": "Weekly Summary: {{ projectData.list.First.Name }}",
        "body": "<html><body><h2>AI-Generated Project Summary</h2><p><strong>Project:</strong> {{ projectData.list.First.Name }}</p><hr><pre>{{ ai_response }}</pre></body></html>"
    }
    ```

## Configuration

This example requires the following configuration options:
-   `isactive`: To enable the extension.
-   `synchronizationfrequency`: Required for scheduler triggers. Set this to `10080` for weekly execution.
-   `project_id`: The ID of the project you want to summarize.
-   `apikey`: For API authentication.
-   `email_address`: The recipient of the summaries.

```json
{
    "name": "project_id",
    "label": { "en": "Project ID" },
    "tooltip": { "en": "The ID of the project to summarize" },
    "type": "string",
    "required": true
}
```
