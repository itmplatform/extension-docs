# ITM Platform's Plugin Script. Example 1
## When a project updates, send a message to a static email address

This example is the most basic version of a plugin you can create, but it will help you understand the fundamental mechanics. 

The full code is in the JSON file included in this same folder.

### Desired functionality

When any project updates (such as its status or name), a recipient will receive an email.

For the sake of simplicity, we will use a static email address `your.friend@itmplatform.com`

Let's see it part by part.

## Plugin General Definition
When you click on the button "New Plugin," you can fill out the form or provide the plugin name in the form and fill in the other properties within the script editor.

More about the [Plugin General Definition](https://github.com/itmplatform/plugin-documentation#plugin-general-definition).

```json
{"name": "project-update-sends-email",
	"svgString": "<svg xmlns='http://www.w3.org/2000/svg' fill='#495d73' viewBox='0 0 640 512'><path d='M640 256c0 35.35-21.49 64-48 64c-32.43 0-31.72-32-55.64-32C522.9 288 512 298.9 512 312.4V416c0 17.67-14.33 32-32 32h-103.6C362.9 448 352 437.1 352 423.6C352 399.1 384 400.4 384 368c0-26.51-28.65-48-64-48s-64 21.49-64 48c0 32.43 32 31.72 32 55.64C288 437.1 277.1 448 263.6 448H160c-17.67 0-32-14.33-32-32V312.4C128 298.9 117.1 288 103.6 288C79.95 288 80.4 320 48 320c-26.51 0-47.1-28.65-47.1-64S21.49 191.1 48 191.1c32.43 0 31.72 32 55.64 32C117.1 223.1 128 213.1 128 199.6V95.1C128 78.33 142.3 63.1 160 63.1l103.6 0C277.1 63.1 288 74.9 288 88.36C288 112 256 111.6 256 143.1C256 170.5 284.7 192 320 192s64-21.49 64-48c0-32.43-32-31.72-32-55.64c0-13.45 10.91-24.36 24.36-24.36L480 63.1c17.67 0 32 14.33 32 32v103.6c0 13.45 10.91 24.36 24.36 24.36c23.69 0 23.24-32 55.64-32C618.5 191.1 640 220.7 640 256z'/></svg>",
	"details": {
		"title": {
			"en": "Sends an email upon project update.",
			"es": "Envía un email tras actualización de proyecto.",
			"pt": "Envia um e-mail após a atualização do projeto."
		},
		"version": "0.0.1",
		"shortDescription": {
			"en": "Example 1. Sends an email to a static address upon project update.",
			"es": "Ejemplo 1. Envía un correo electrónico a una dirección estática tras la actualización del proyecto.",
			"pt": "Exemplo 1. Envia um e-mail para um endereço estático após a atualização do projeto."
		}
	}
```
We have removed the `longDescription`, `version`, and other properties to make it smaller.

## Features and actions

The specification tells us we need to trigger the feature when the project is updated. 

So, we know that the trigger is an event, and from the [event reference](https://github.com/itmplatform/plugin-documentation#event-reference) we also know that the entity will be `Project` and the action `updated`.

Let's see what this would look like:
```json
{"features": [
		{
			"trigger": "event",
			"event": "updated",
			"entity": "Project",
			"description": "Project update event will send email.",

```
Now that we have a feature triggering when we need it, we have to tell it what actions to perform.

Because this is a simple feature, it will only require an [email type of action](https://github.com/itmplatform/plugin-documentation#action-email). 

Let's create one within the actions array. The following is the complete `features` property of this plugin.

```json
{"features": [
		{
			"trigger": "event",
			"event": "updated",
			"entity": "Project",
			"description": "Project update event will send email.",
			"actions": [
				{
					"action": "email",
					"to": "your.friend@itmplatform.com",
					"subject": "Project updated: {{input.project.Name}}",
					"body": "<html><h2>Project update notification</h2><p>{{input.project.Name}} was updated!</p></html>"
				}
			]
		}
	],
```
Notice we are using [templates](https://github.com/itmplatform/plugin-documentation#template-syntax) for the first time, both in the `subject` and `body`.

Let's focus on 
```js
"subject": "Project updated: {{input.project.Name}}"
```
This means that the email subject will be a string of text containing "Project updated:" followed by another text stored in `input.project.Name`.

Remember, `input` is the "context" object that any event will send to your plugin. To figure out what is in that object, look in the [event reference](https://github.com/itmplatform/plugin-documentation#event-reference) for your event. 

In our case, we find this:

> When a project is updated: ``` { { "accountId", accountId }, { "userId", userId }, { "project": { "Id", projectId }, { "Name", projectName }, { "TypeId", typeId },  { "Description", description }}} ```

Meaning that `input.project.Name` holds the project name. Placing that variable in curly braces will tell the template to render its value.

Therefore, `"Project updated: {{input.project.Name}}"` will render in an email subject "Project updated: My First Project"

The same goes for the body, into which we wanted to add some HTML.

## Configuration
As described in the [plugin configuration](https://github.com/itmplatform/plugin-documentation#plugin-configuration) section, we must add an [activation option](https://github.com/itmplatform/plugin-documentation#activate-the-plugin-name-isactive) so you can, later on, activate this plugin.

```json
"config": [
		{
			"name": "isactive",
			"label": {
				"en": "Activate plugin",
				"es": "Activar extensión",
				"pt": "Ativar o plugin"
			},
			"tooltip": {
				"en": "You must activate the plugin for it to run.",
				"es": "Debe activar el plugin para que se ejecute.",
				"pt": "Você deve ativar o plugin para que ele seja executado."
			},
			"type": "checkbox",
			"required": false
		}
	],
```

Don't forget to activate the plugin by clicking on the "Configuration" section of your plugin and clicking on the checkbox you created.

:point_right: If you make any changes in the code, you will have to deactivate it and reactivate back again for the changes to apply.

<a href="../example-2/">Continue to example 2</a>
