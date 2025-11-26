# Example 6. Weekly AI Project Summary
## Send a weekly email with an AI-generated summary of all active projects

This example demonstrates how to use the `llm` action to generate content using Artificial Intelligence.

The full code is in the JSON file included in this same folder.

### Desired functionality

Every week, the extension will:
1.  Log in to ITM Platform.
2.  Search for all active projects.
3.  Iterate through each project.
4.  Send the project data to an LLM to generate a summary.
5.  Email the summary to a configured address.

## Features and actions

The trigger is set to `scheduler`, meaning it will run based on a time interval.

```json
"trigger": "scheduler",
"description": "Weekly AI Project Summary",
```

The actions sequence is as follows:

1.  **Login**: Authenticates with ITM Platform API.
2.  **Search**: Retrieves active projects using the `restcall` action.
3.  **Loop**: Iterates over the list of projects found.
    ```json
    "action": "loop",
    "loop": {
        "var": "projectData.list",
        "output": "project",
        "index": "index"
    }
    ```
4.  **LLM**: Inside the loop, we use the `llm` action to generate the summary.
    ```json
    {
        "action": "llm",
        "description": "Generate Project Summary",
        "input": "Summarize the project described in the provided JSON using PMI-style terminology and structure. {{json project}}. I want to send it in an email so don't suggest anything.",
        "output": "ai_response"
    }
    ```
    The `input` field contains the prompt for the AI. We use `{{json project}}` to pass the current project's data (from the loop) to the prompt.
5.  **Email**: Sends the generated summary.
    ```json
    {
        "action": "email",
        "to": "{{ config.email_address }}",
        "subject": "Weekly Summary: {{ project.Name }}",
        "body": "<html><body><h2>AI-Generated Project Summary</h2><p><strong>Project:</strong> {{ project.Name }}</p><hr><pre>{{ ai_response }}</pre></body></html>"
    }
    ```

## Configuration

This example requires the following configuration options:
-   `isactive`: To enable the extension.
-   `synchronizationfrequency`: Required for scheduler triggers. Set this to `10080` for weekly execution (60 minutes * 24 hours * 7 days).
-   `apikey`: For API authentication.
-   `email_address`: The recipient of the summaries.

```json
{
    "name": "synchronizationfrequency",
    "label": { "en": "Synchronization frequency (minutes)" },
    "tooltip": { "en": "Minutes required for weekly execution (10080)" },
    "type": "string",
    "required": true
}
```

