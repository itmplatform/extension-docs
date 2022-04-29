<div style="text-align: center">
<img src="./extensions-mark-title-framed-blue-bckg.svg" width="300">
</div>

<!-- *** HTML version *** -->

<!-- Uncomment the line below before converting this file to HTML -->
<!-- <script src="https://kit.fontawesome.com/81ab11011e.js" crossorigin="anonymous"></script> -->

<!-- Comment the line below before converting this file to HTML-->
<a href="https://developers.itmplatform.com/extension-docs/" target> HTML version of this document.</a>

<!-- *** HTML version *** -->

# Overview
Extensions add a specific feature to ITM Platform. They can be used to extend functionality or as a connector between ITM Platform and a third-party system.

You can create your own custom extensions. This guide will help you get started and serve you as a reference when you become an expert.
 
Each section of this guide is organized into three blocks: <i class="far fa-code" title="Learn by example"></i> **Learn by example**, <i class="fad fa-book-open" title="Guide"></i> **Guide**, and <i class="fad fa-brackets" title="Reference"></i> **Reference**.

The <a href="#terminology">terminology</a> section will help you with the most commonly used terms.

## Basic concepts

- You can access the extension panel in ITM Platform on the CONFIGURATION <i class="fa-solid fa-caret-right"></i> CUSTOMIZATION menu. 
- You need a Full Access license to operate on existing global extensions and edit or create existing ones. 
- You can create and edit your own extensions in the "code editor" section, which will give you access to the extension script. Notice that you can't edit global extensions.
- A extension script is a JSON file that follows a structure of instructions that will perform the functionality you need.
- A extension can be triggered based on time (scheduler) or specific ITM Platform events. For example, every day or when a purchase is updated.
- Once triggered, the extension will initiate actions that can be chained to one another, forming a sequence of actions called features.
- Once a extension is ready, you can activate it.

# Extension script
A extension script is defined in a single JSON file, following this structure:

1. <a href="#general">Extension General Definition</a>
1. <a href="#actions">Features and Actions</a>
1. <a href="#configuration">Configuration</a>
1. [Field Mapping](#field-mapping)

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
    {
        "name": "unique-identifier",
        "svgString": "<svg></svg>",
        "details": {},
        "features": [{
                "trigger": "scheduler",
                "actions": [{}, {}]
            }
        ],
        "config": [{}, {}],
        "mapping": []
    }
```
<a name="general"></a>

## Examples
All these examples include the code and a tutorial. It is recommended to start with the first and go into more complex examples as you understand the simpler ones.

- [Example 1](https://github.com/itmplatform/extension-docs/tree/main/example-1). When a project updates, send a message to a static email address.
    - Event-based
    - Minimal configuration: activation only
- [Example 2](https://github.com/itmplatform/extension-docs/tree/main/example-2).	When a purchase's actual value updates, send an email to a static address if the amount exceeds the purchase budget.
    - Event-based
    - Using conditions
- [Example 3](https://github.com/itmplatform/extension-docs/tree/main/example-3).	Send a daily email to the project manager of a specific project containing all tasks whose end date is later than today.
    - Scheduler-based
    - Loop and conditions
- [Example 4](https://github.com/itmplatform/extension-docs/tree/main/example-4). Synchronize ITM Platform's clients with Hubspot's companies: only those having the Hubspot's property `sync_itm_platform` set to `true`.
    - Scheduler-based
    - Connector to a third-party system
- [Example 5](https://github.com/itmplatform/extension-docs/tree/main/example-5). Allow users to input revenue's Actual Amount if <= Projected Amount. 
    - Event-based, synchronous

## Extension General Definition

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
    {
        "name": "unique-name",
        "svgString": "<svg xmlns='http://www.w3.org/2000/svg' fill='#495d73' viewBox='0 0 640 512'><path d='M640 256c0 35.35-21.49 64-48 64c-32.43 0-31.72-32-55.64-32C522.9 288 512 298.9 512 312.4V416c0 17.67-14.33 32-32 32h-103.6C362.9 448 352 437.1 352 423.6C352 399.1 384 400.4 384 368c0-26.51-28.65-48-64-48s-64 21.49-64 48c0 32.43 32 31.72 32 55.64C288 437.1 277.1 448 263.6 448H160c-17.67 0-32-14.33-32-32V312.4C128 298.9 117.1 288 103.6 288C79.95 288 80.4 320 48 320c-26.51 0-47.1-28.65-47.1-64S21.49 191.1 48 191.1c32.43 0 31.72 32 55.64 32C117.1 223.1 128 213.1 128 199.6V95.1C128 78.33 142.3 63.1 160 63.1l103.6 0C277.1 63.1 288 74.9 288 88.36C288 112 256 111.6 256 143.1C256 170.5 284.7 192 320 192s64-21.49 64-48c0-32.43-32-31.72-32-55.64c0-13.45 10.91-24.36 24.36-24.36L480 63.1c17.67 0 32 14.33 32 32v103.6c0 13.45 10.91 24.36 24.36 24.36c23.69 0 23.24-32 55.64-32C618.5 191.1 640 220.7 640 256z'/></svg>",
        "details": {
            "title": {"en": "My Extension","es": "Mi extensión","pt": "Meu extençao"},
            "version": "0.8",
            "shortDescription": {"en": "desc","es": "desc","pt": "desc"},
            "longDescription":  {"en": "long desc","es": "long desc","pt": "long desc"},
            "updated": "2022-05-25",
            "developer": "developer-name"
        }
    }        
```
<i class="fad fa-book-open" title="Guide"></i> **Guide**

The general definition will provide basic information about the extension you are building. The title, the short description, and the icon will appear on the extension button within the extension panel.

<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `name`: It will be used as a identifier for the extension and must be unique. It can't have spaces or special characters.
- `svgString`: Inline SVG that will be used as the extension icon.
- `details`
    - `title`: Main identifier of the extension for the end-user. It accepts <a href="#multi-language">multi-language</a>
    - `version`: Version number. Take into account that the system will not store previous versions.
    - `shortDescription`It will appear in the extension panel. It accepts <a href="#multi-language">multi-language</a>
    - `longDescription`: It accepts <a href="#multi-language">multi-language</a>
    - `updated`: yyyy-mmm-dd date format 
    - `developer`: Who developed the extension

    ### Checkpoints

    - Extension `name` must exist and be unique

<a name="actions"></a>

## Features 

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{"features": [
    {
        "trigger": "event",
        "extension": "providers-demo",
        "entity": "Project",
        "description": "Retrieving providers",
        "event": "updated",
        "actions": []
    }
]}

```
<i class="fad fa-book-open" title="Guide"></i> **Guide**

The Features section defines the behavior of the extension. It determines what will trigger it, and the actions that will follow.

When a feature is triggered by an event, such as a task update, you must specify the `entity`(task, in our example) and the `event` (update, in the same example).

When the feature is triggered by the scheduler, you must create the [synchronization frequency](#synchronization-frequency-"name"-"synchronizationfrequency") configuration option, so the user can set the value.

<i class="fad fa-brackets" title="Reference"></i> **Reference**

(*) Denotes a mandatory field

- `trigger`: It can be `event` or `scheduler` *
- `entity`: [Entities](#terminology) * mandatory for the `event` trigger
- `event` [System Events](#Events) * mandatory for the `event` trigger
- `async`: `true` or `false` It applies to the `event` trigger and determines whether the execution in ITM Platform that triggered it will stop (`false`) or the extension will run in a parallel thread (`true`). The default value is `async`: `true` if not specified.
- `description`: Useful for extension developers. It will not be displayed to the end-user. 
- `actions`: An array of [actions](#actions).

    ⚠️ If you make the feature synchronous (`async: false`), the execution in ITM Platform will pause until the extension responds, resulting in a potential lousy user experience. Make features synchronous only when necessary, and keep the logic as simple as possible. For example, when validating user inputs.

 ### Checkpoints

 - Each extension should contain at least one feature.
 - Triggers must be either `scheduler` or `event`
 - Each feature will have actions array.

## Actions 

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
     {
		"action": "restcall",
		"url": "https://api.itmplatform.com/v2/mycompany/projects/{{ input.project.Id }}",
		"method": "GET",
		"token": "{{ logininfo.Token }}",
		"description": "Retrieve project",
		"output": "project",
		"payload": "",
        "headers": "{\"token\" : \"{{ logininfo.Token }}\", \"myheader\" : \"header value\"}"
		"dataType": "application/json"
	}
```

<i class="fad fa-book-open" title="Guide"></i> **Guide**

Actions are pieces of code executed when a feature is triggered. Actions are typically chained to one another by sending the `output` value from the parent action to the child action. 

Actions admit [loops](#loops) and [conditionals](#conditionals).

<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `action`:  specifies the action type from the different [action types]().
- `token`: Used to call the ITM Platform API <!--reference-->
- `description`: For developers
- `output`: <a href="#output-object">output object</a> used in further actins as the `input` object <!-- reference. -->
- `payload`: body of the request. It supports [templates](#template-syntax), used to replace text with variable content.
- `headers`: Optional. Valid for `restcall` only. You can pass additional required headers to the rest call
- `token`: Optional. Valid to call ITM Platform's api. Shortcut to adding it as a header, like so: `"headers": "{\"token\" : \"{{ logininfo.Token }}\"}"`
- `dataType`: Optional. It will parse the result as JSON or XML. Possible values are `application/json` and `application/xml`. When omitted, the result will be a string.


#### Checkpoints
When saving an extension, the following validations will run. If the script doesn't pass them, you will be able to save it but not activate the extension.
 - The action value can be one of: `restcall`, `soapcall`, `validate`, `map`, `email`, or `loop`
 - If the action value is `restcall`, it can contain `url`, `method` (defaults to `GET`) and `payload` (not mandatory).
- If the action value is `restcall`, it can have a `condition`.
- If the action value is `soapcall`, it must include a `soapaction`, `url` and `payload`.
- If the action value is `restcall` or `soapcall`, there is an authentication attribute and its value is basic, it must have username and password. <!--explain this ?-->
- If the action value is `validate`, it must have `validationCondition` and `message`.
- If the action value is `map`, it must have `template`.
- If the action value is `loop`, it must have `var`, `condition`, `actions` and `output`.


### Action Types

The `action` value can be one of: `restcall`, `soapcall`, `validate`, `map`, `email`, or `loop`

#### "action": "restcall"

Performs a <a href="https://en.wikipedia.org/wiki/Representational_state_transfer" target="_blank">REST call</a> to any third-party API or ITM Platform's.

<i class="far fa-code" title="Learn by example"></i> **Learn by example**
<!-- We already explained all this. See a way to not repeat ourselves, but keep it understandable with requiring to read the whole document-->

```json
	{
		"action": "restcall",
		"url": "https://jsonplaceholder.typicode.com/todos/",
		"method": "GET",
		"description": "Retrieves a to-do list",
		"output": "todos",
		"payload": "",
		"dataType": "application/json"
	}
```
<i class="fad fa-book-open" title="Guide"></i> **Guide**

A REST call is the most common way to get or post data in any API, including ITM Platform's. Use it as you would with any tool, such as Postman, `curl`, or your favorite programming language.


<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `action`: Must be `restcall` 
- `url`: See the <a href="#url">url</a> section.
- `method`: specifies the HTTP method (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`)
- `description`: Reference for the developer
- `output`: <a href="#output-object">output object</a> used in further actions
- `payload`: body of the request. 
- `dataType`: is optional and it will parse the result as json so it can be send to any next step, if not passed then the result will be a string

#### "action": "soapcall"

Performs a <a href="https://en.wikipedia.org/wiki/SOAP" target="_blank">SOAP call</a> to any third-party system.

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{
    "action": "soapcall",
    "url": "http://testurl.com/testservice",
    "soapaction": "testmethod",
    "payload": "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:tem=\"http://tempuri.org/\"><soapenv:Header/><soapenv:Body><tem:CallPost><tem:name>{{ input.SomeInt }}</tem:name></tem:CallPost></soapenv:Body></soapenv:Envelope>",
    "dataType":  "application/json",
    "authentication": "basic",
    "user": "testuser",
    "pass": "testpass",
    "output": "jsonoutput"
}
```

<i class="fad fa-book-open" title="Guide"></i> **Guide**

A SOAP action will require a `url`, a `soapaction`, a `payload` and an `output`. The `dataType` attribute must be `"application/json"`. It can also include an authentication attribute, along with `user` and `pass`. 

**Data Types**

The following example parses the `output` as xml, which can be used later on to retrieve values with mapping
```json
{
    "action": "soapcall",
    "url": "http://testurl.com/testservice",
    "soapaction": "testmethod",
    "payload": "{{ output1 }}",
    "dataType":  "application/xml",
    "output": "dataoutput"
}
{
    "action": "soapcall",
    "url": "http://testurl.com/testservice",
    "soapaction": "testmethod",
    "payload": "{{ output1 }}",
    "dataType":  "application/json",
    "output": "dataoutput"
}
```


<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `action`: Must be `soapcall`
- `soapaction`: SOAP action
- `url`: SOAP URL
- `payload`: The payload of the SOAP Action needs to be a SOAP Envelop message like specified here https://www.tutorialspoint.com/soap/soap_message_structure.htm The easiest way to create an envelop that you can use later, is to use a tool like <a href="https://www.soapui.org/" tagert="_blank">SOAPUI</a>, and then replace the values.
- `authentication`: The type of authentication required, if any. The accepted values are.
   - `basic` which will require `user` and `pass`
   - `token` which will require `token` 
- `user`: User name when basic authentication is required.
- `pass`: Password when basic authentication is required.
- `token`: Bearer token when token authentication is required

#### action": "map"


<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
    {
        "action": "map",
        "template": "{ \"name\": \"{{ input.name }}\", \"value\": \"{{ input.value }}\" }",
        "dataType": "application/json",
        "output": "outputVar"
    }
```


<i class="fad fa-book-open" title="Guide"></i> **Guide**

The `map` actions allow mapping values from existing variables. In the example above, the  variables `name` and `value` coming from `input` will be transformed to JSON.

<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `template`: See [template syntax](#template-syntax)
- `output`: <a href="#output-object">output object</a> used in other actions.
- `dataType`: is optional, and it will parse the result as JSON so it can be sent to any next step. when omitted the result will be a string

#### "action": "validate"

Used in `synchronous` features, it performs a validation before proceeding. 

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
    {
        "action": "validate",
        "validateCondition": "Convert.ToInt32(input.Status.Id) == 10 && Convert.ToString(input.Description) == \"\"",
        "message": "When using the status {{ input.Status.Description }} you need to provide a description"
    }
```

<i class="fad fa-book-open" title="Guide"></i> **Guide**

The `validate` action will require a `validationCondition`, and a `message` that will be displayed to the final user if the condition is not met. It is generally used to perform custom validations that may need to change the way ITM platform works. For example, if you need to validate that a particular field has a value depending on other fields.

<!-- These type of actions should not use async as they are meant to stop the processing and rollback whatever action was undergoing, but in the future maybe we can use this to record messages in the jobs when they execute from the scheduler to stop the processing of repeat tasks in case a validation is happening, this would allow us to avoid executing tasks that would fail in the scheduler. -->

<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `action`: `validate` 
- `validateCondition`: It follows the same rules and syntax of [action conditionals](https://github.com/itmplatform/extension-documentation#action-conditionals)
- `message`: String displayed to the final user in the ITM Platform's interface in case the condition is not met.


#### "action": "email"
Sends an email to the given address/es

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
        {
          "action": "email",
          "to": "yourname@yourcompany.com",
          "subject": "This subject admits templates",
          "body": "<html><h1>This email body admits templates too!</h1></html>"
        }
```

<i class="fad fa-book-open" title="Guide"></i> **Guide**

Sends an email to the specified recipient(s). The sender will be notifier@itmplatorm.com, an address that doesn't admit replies.

<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `action`: Must be `email`
- `to`: Mandatory, indicates a comma-separated list of email addresses.
- `subject`: Contains the subject text. It allows templates.
- `body`: contains the message body. It allows templates.


#### "action": "loop"

The `loop`action will iterate over elements coming from previous calls.

There are different types of loops: simple, with variables, an with variables and conditions.

##### Simple loops

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
 { 
    "trigger": "event",
    "event": "updated",
    "entity": "Project",
    "actions": [
        {
            "action": "loop",
            "loop": {
                "var": "myarray",
                "output": "item",
                "index": "index"
            },
            "actions": [
                { 
                    "action": "restcall",
                    "url": "http://your-company/{{ item.name }}/{{ index }}",
                    "method": "post",
                }
            ]
        }
    ]
} 
```

<i class="fad fa-book-open" title="Guide"></i> **Guide**

A loop action will parse an object coming from a previous action and run an actions array over the outcome.

<i class="fad fa-brackets" title="Reference"></i> **Reference**


##### Loops with variables

We can also loop with variables inside an action. For example:

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{
    "trigger": "event",
    "event": "updated",
    "entity": "Project",
    "action": "loop",
    "loop": {
        "var": "myarray.values",
        "output": "singleElement",
    },
    "actions": [
        {
            "action": "restcall",
            "url": "http://testurl.com/testapi",
            "method": "POST",
            "payload": "{\"Columns\": \"Id,Name,JiraId,Type.Id,MethodTypeId\",\"filter\": { \"JiraId\" : {{ singleElement.id }} } }",
            "output": "outputVar",
            "dataType": "application/json"
        },
```
<i class="fad fa-book-open" title="Guide"></i> **Guide**

<i class="fad fa-brackets" title="Reference"></i> **Reference**

##### Loops with variables and conditions

We can also loop with variables and conditions inside an action. For an example:
<i class="far fa-code" title="Learn by example"></i>

```json
{
    "trigger": "event",
    "event": "updated",
    "entity": "Project",
    "action": "loop",
    "loop": {
        "var": "myarray.values",
        "output": "singleElement",
        "condition": "somevariable != null"
    },
    "actions": [
        {
            "action": "restcall",
            "url": "http://testurl.com/testapi",
            "method": "POST",
            "payload": "{\"Columns\": \"Id,Name,JiraId,Type.Id,MethodTypeId\",\"filter\": { \"JiraId\" : {{ singleElement.id }} } }",
            "output": "outputVar",
            "dataType": "application/json"
        },
        {
            "action": "restcall",
            "url": "http://testurl.com/testapi",
            "condition": "Convert.ToInt32(outputVar.total) > 0",
            "method": "PATCH",
            "payload": "{\"Name\": \"{{ singleElement.name }}\",\"Description\": \"{{ singleElement.description }}\" }",
            "output": "updatedoutputVar",
            "dataType": "application/json"
        }
    ]
}
```
<i class="fad fa-book-open" title="Guide"></i> **Guide**

<i class="fad fa-brackets" title="Reference"></i> **Reference**


### Action conditionals
Conditionals

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
    {
        "action": "restcall",
        "url": "https://api.hubapi.com/companies/v2/companies/recent/modified?hapikey={{config.apikeyHubspot }}&since={{ context.LastExecution }}", 
        "condition": "{{ context.LastExecution }} != null",
        "method": "GET",
        "description": "retrieve all clients filter lastExecutiondate",
        "output": "companiesHS",
        "payload": "",
        "dataType": "application/json"
    }
```

<i class="fad fa-book-open" title="Guide"></i> **Guide**
You can put a condition to the event to check whether the actions should be executed.

<!-- The conditions are executed using .net framework, but to make it easier to write also supports templates, therefore a complex condition could be written like this:    -->


```json
{
"condition": "{{input.SomeObject.Array.0.Array.1.SomeData}} == 3"
}
```

## Extension Configuration 

The `"config"` array will determine the content rendered in the extension's configuration tab on ITM Platform, displaying all fields required for your extension to operate.

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
  {
    "config": [

            {
            "name": "option-name",
            "label": {
                "en": "Label in English",
                "es": "Etiqueta en español",
                "pt": "Rótulo em inglês"
            },
            "tooltip": {
                "en": "A brief description for the tooltip",
                "es": "Una breve descripción para el tooltip",
                "pt": "Uma breve descrição para o tooltip"
            },
            "type": "date",
            "required": true
        }
    ]
}
}

```

<i class="fad fa-book-open" title="Guide"></i> **Guide**

These fields are represented by the *configuration options* within the `config` array.

Once the user fills out the ITM Platform's "Configuration" tab, the values will be available through the `config` object, available to be used in *features* and *actions*.

These are some examples of the usage of the `config` object throughout a script:

```json
"url": "{{ config.url }}/rest/api/3/project/search?maxResults=50&startAt=0"
```
```json
"username": "{{ config.user }}"
```
```json
"payload": "{\"Name\": \"{{ singlejiraproject.name }}\",\"Description\": \"{{ singlejiraproject.description }}\",\"InternalCode\": \"{{ singlejiraproject.key }}\",\"JiraURL\": \"{{ config.url }}\",\"SynchronizationSource\": \"Jira\"{{#ifempty singlejiraproject.projectCategory }}{{else}},\"TypeId\": \"{{ map config.mapping.projecttype.external singlejiraproject.projectCategory.id }}\"{{/ifempty}} }"
```
These *configuration options* can define any **custom value** that you need to store (eg. the API key required to log in), but it also interprets a few **hardcoded values**.

The identifier for each *configuration option* is the `name` property



<!-- In addition we've got
syncafterdate. -> defaulttaskdurationindays (Duration in days in for tasks in ITM Platform that are synchronized from Jira issues that do not have an end date)
But it is not clear how to use them -->

<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `name`: *Custom configuration options* accept any any valid unique name. *Hardcoded configuration options* are `"isactive"`, `"synchronizationfrequency"`, `"syncafterdate"` 
- `label`: Message that will show before the user input. For example `"Start Date"`
- `tooltip`: Message that will show when the user hovers over the information icon.
- `type`: Defines the type of the user input. The valid values are `string`, `password`, `checkbox`, `date`, `dropdown`, and `header`. 
- `required`: specifies if this configuration is required in order to execute the extension. It only applies to the types `string`, `password`, `date`, and `dropdown`.

The `header` type will render regular section header Section headers on ITM Platform's configuration tab, adding a adding a visual separation between fields.

```json
    {
        "config": [
        {
                    "name": "header-credentials",
                    "label": {
                        "en": "Credentials section",
                        "es": "Sección de credenciales",
                        "pt": "Seção de credenciais"
                    },
                    "type": "header"
                }
        ]
    }
```


### Hardcoded configuration options
The hardcoded *configuration options* will be identified by their name and define a specific behavior in the *extension interpreter*. These hardcoded options are:

- `"name": "isactive"` 
- `"name": "synchronizationfrequency"` 

#### Activate the extension (`"name": "isactive"`)
Provides the option for the user to activate the extension within the "Configuration" tab.

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{
    "config": [
            {
                "name": "isactive",
                "label": {
                    "en": "Activate connector",
                    "es": "Activar conector",
                    "pt": "Ativar o conector"
                },
                "tooltip": {
                    "en": "When selected, the connector will be active and will start synchronizing data",
                    "es": "Cuando esté seleccionado, el conector estará activo y comenzará a sincronizar datos",
                    "pt": "Quando selecionado, o conector estará ativo e começará a sincronizar dados"
                },
                "type": "checkbox",
            }
    ]
}
```
<i class="fad fa-book-open" title="Guide"></i> **Guide**
This will display a checkbox to activate the extension on the configuration tab in ITM Platform.

⚠️ If you don't add this configuration, you will not be able to activate the extension later on.

<i class="fad fa-brackets" title="Reference"></i> **Reference**

Properties with special considerations:
- `name`: Must be `"isactive"` 
- `type`: Must be `"checkbox"` 


#### Synchronization frequency (`"name": "synchronizationfrequency"`)

Provides the configuration option for the user to set the frequency at which the extension will be triggered. It is ***required*** when the feature is triggered by `scheduler`

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{
    "config": [
            {
                "name": "synchronizationfrequency",
                "label": {
                    "en": "Synchronization frequency (minutes)"
                },
                "tooltip": {
                    "en": "Frequency you wish to establish for the synchronization of the data from your account"
                },
                "type": "string",
                "required": true
            }
    ]
}
```

<i class="fad fa-book-open" title="Guide"></i> **Guide**
It will display in the "Configuration" tab a user input to introduce the synchronization frequency in minutes for the scheduler. 

The minimum frequency is 60 minutes. We recommend setting the frequency at the maximum your business case accepts.

It will apply to the feature(s) that have the `scheduler` trigger.

<i class="fad fa-brackets" title="Reference"></i> **Reference**

Properties with special considerations:
- `name`: Must be `"synchronizationfrequency"`
- `type`: Must be `"string"`
<!-- This should be 'number' -->


## Field Mapping
The mapping configuration allows you to establish an equivalence between two systems, typically ITM Platform and a third party. 

This is useful to create synchronizations based on the values of one or more fields.

As an extension developer, you don't set those relationships. You give the user the means to create relationships in the configuration tab in ITM Platform.

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{
 "mapping": [
        {
            "name": "projecttype",
            "label": {
                "en": "Project Type",
                "es": "Tipo de proyecto",
                "pt": "Tipo de Projeto"
            },
            "external": { 
                "method": "GET",
                "url": "@@url@@/rest/api/3/project/type",
                "authentication": "Basic",
                "user": "@@user@@",
                "pass": "@@pass@@",
                "id": "key",
                "name": "formattedKey",
                "columnName": {
                    "en": "Jira",
                    "es": "Jira",
                    "pt": "Jira"
                }
            },
            "internal": {
                "method": "GET",
                "url": "v2/@@companyId@@/GetProjectTypes",
                "id": "Id",
                "name": "Name",
                "columnName": {
                    "en": "ITM Platform",
                    "es": "ITM Platform",
                    "pt": "ITM Platform"
                }
            }
        }
 ]
}
```
<!-- v2/@@companyId@@/ won't work -->
<!-- Explain "@@user@@", "@@pass@@" -->
<i class="fad fa-book-open" title="Guide"></i> **Guide**

In the example above, the configuration panel will show two columns filled with the project types of ITM Platform and another with the project types of Jira. This will allow the user to establish that --for example- the project type "Software" in ITM Platform is equivalent to the project "Development" in Jira.

<!-- Insert image -->

<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `external`: The system A.
- `internal`: The system B.  
- `url`: Endpoint that will gather the values.
- `authentication`: The type of authentication required, if any. The accepted values are.
   - `basic` which will require `user` and `pass`
   - `token` which will require `token` 
- `user`: User name when basic authentication is required.
- `pass`: Password when basic authentication is required.
- `token`: Bearer token when token authentication is required
- `id`: ID or name of the property containing the value of interest.
- `name`: Name displayed
- `columnName`: Label for the column that will appear in the config panel


# Event Reference

The following are the events that ITM Platform triggers.

|Trigger|Entity|Action|When|Input|
|---|---|---|---|---|
|scheduler|||This executes from scheduler|context.LastExecution|
|event|Task|inserted|When a task is inserted| ``` { { "accountId", accountId }, { "projectId", projectId },  { "userId", userId }, { "task": { "Id", taskId }, { "Name", taskName }, { "JiraTaskId", JiraTaskId }, { "KindId", taskKindId }}} ```|
|event|Task|updated|When a task is updated|``` { { "accountId", accountId }, { "projectId", projectId },  { "userId", userId }, { "task": { "Id", taskId }, { "Name", taskName }, { "JiraTaskId", JiraTaskId }, { "KindId", taskKindId }}}``` |
|event|Project|inserted|When a project is created| ``` { { "accountId", accountId }, { "userId", userId }, { "project": { "Id", projectId }, { "Name", projectName }, { "TypeId", typeId },  { "Description", description }}}```|
|event|Project|updated|When a project is updated| ``` { { "accountId", accountId }, { "userId", userId }, { "project": { "Id", projectId }, { "Name", projectName }, { "TypeId", typeId },  { "Description", description }}} ```|
|event|Purchase|updated|When a Purchase is update| ``` { { "accountId", accountId }, { "projectId", projectId }, { "userId", userId }, { "purchase", new JObject() { { "Id", purchaseId }, { "Name", purchaseName }, { "ActualAmount", actualAmount }, { "ProjectedAmount", projectedAmount } } } }```|
|event|Revenue|pre insert|When a Revenue is going to be created| ``` { { "ProjectedAmount", projectedAmount }, { "ActualAmount", actualAmount }, { "ProjectId", projectId } } ```|
|event|Revenue|inserted|When a Revenue is created| ``` { { "Id", revenueId }, { "Name", revenueName }, { "DueDate", dueDate }, { "ProjectedAmount", projectedAmount }, { "ActualAmount", actualAmount }, { "Status", statusId }, { "ProjectId", projectId }, { "UserId", UserId }, { "AccountId", AccountId } } ```|
|event|Revenue|pre update|When a Revenue is going to be updated| ``` { { "ProjectedAmount", projectedAmount }, { "ActualAmount", actualAmount }, { "ProjectId", projectId } } ```|
|event|Revenue|updated|When a Revenue is updated| ``` { { "Id", revenueId }, { "Name", revenueName }, { "DueDate", dueDate }, { "ProjectedAmount", projectedAmount }, { "ActualAmount", actualAmount }, { "Status", statusId }, { "ProjectId", projectId }, { "UserId", UserId }, { "AccountId", AccountId } } ```|


## Event bubbling up
Events don't bubble up how you may be used in other environments. But if an entity event affects others, such as its parents, these parents will also trigger an event.

For example, you change the end date of a task. That will trigger the event `updated` on the entity `Task`. Now, consider these two scenarios:
- The task end date doesn't change the project's end date (or any other value, such as the cost). Then the project will not trigger any event
- The task end date pushes forward the project's end date. Then that will trigger the event `updated` on the entity `Project`


# Template Syntax

Wherever templates are supported, you can use curly braces to wrap an expression `{{ expression }}` to generate the text that you need.

The template system is based on [Handlebars](https://handlebarsjs.com/) and supports all the native [expressions](https://handlebarsjs.com/guide/expressions.html#basic-usage), plus a set of additional expressions that will come in handy when building an extension.

<!-- > caveat: Leave a space between curly craces and the expression {{ expression }} -->

> caveat: If you need to include quotes within the template (tipically in the `payload`), you need to escape internal quotes with a backlash.  `{{ "payload": "{\"Value\": 23 }}`

<!-- Mustache, when given a JSON structure, will try to read it and get all the values without the keys, so it is better not to pass JSON to the Mustache template -->

## Additional expressions

The following are Handlebars expressions created to specifically interact with the extension interpreter. 

### Data type converters

Because the extension script is written in plain text, you will need to tell the interpreter what data type you intend to pass.

- `date`: Transforms a string to a date type.
    - Parameters: `input` `format`
    - Example: `"payload": "{\"jql\" : \"updated >= '{{ date config.syncafterdate 'yyyy/MM/dd HH:mm' }} \" }"`
- `timespan`  Transforms a string to a time type. 
    - Parameters: `input`
    - Example: `"payload": "{\"timetracking\": {\"originalEstimate\": {{ timespan taskdetails.list.0.Efforts.Estimated }}\" }"`

- `timespanFromSeconds` Transforms seconds to a time type
    - Parameters: `input`
    - Example: `"payload": "{\"ReportedHours\": \"{{ timespanFromSeconds singleworklog.timeSpentSeconds 'hh\\:mm'}}\" }"`
- `int` Transforms a string to an integer.
<!-- cannot find any int example within extensions -->

### Value replacements

- `map` Replaces a mapped value with its equivalent, as defined in the [field mapping](#field-mapping)
    - Parameters: `map reference` `input value`
    - Example: `"payload": "{\"TypeId\": {{ map config.mapping.projecttype.external singlejiraproject.projectCategory.id }},\" }"`
- `mapArray` Get mapping from the array passed in parameter.
    - Parameters: `map reference` `input value`
    - Example: `"payload": "{\"CustomField\": [{ \"CustomFieldBaseId\": {{ mapArray customfields 'Name' 'Jira Link' 'Id'}} }] }"`
<!-- This is not clear -->
- `StringReplace` Replaces the second argument with the third argument in the first argument.
<!-- This is not clear. No examples in existing extensions -->
- `DoubleQuotesReplace` Encodes double Quotes(") with `&quot;`.
    - Parameters: `input`
    - Example: `"payload": "{\"Name\": \"{{ DoubleQuotesReplace singlejiraissue.fields.summary }}\" }"`
- `json` Returns serialized object from json
<!-- This is not clear. No examples in existing extensions -->

### Conditionals

- `ifempty` If the value given in the argument is empty then it executes the first statement otherwise second.
                  "payload": "{\"jql\" : \"project = {{ singlejiraproject.id }} AND updated >= '{{#ifempty context.LastExecution }}{{ date config.syncafterdate 'yyyy/MM/dd HH:mm' }}{{else}}-{{ config.synchronizationfrequency }}m{{/ifempty}}'\", \"startAt\" : 0, \"maxResults\" : 1000}",

- `ifext` If the condition given in the argument is satisfied then it executes the first statement otherwise second.
<!-- This is not clear. No examples in existing extensions -->

- `ifNull` Executes the first statement if the value passed is null otherwise executes the second statement.
<!-- This is not clear. No examples in existing extensions -->

- `ifEquals` If 2 values given in the argument are matched then it executes the first statement otherwise second.
                      "payload": "{\"Name\": \"{{ singlejiraissue.fields.summary }}\",\"Details\": \"{{ adfParserDescription singlejiraissue.fields.description }}\",\"JiraURL\": \"{{ config.url }}\",\"SynchronizationSource\": \"Jira\",\"StartDate\": \"{{ date singlejiraissue.fields.created 'yyyy-MM-dd'}}\",\"EndDate\": {{#ifEquals singlejiraissue.fields.duedate '' }}\"{{DateTime.Now \"yyyy-MM-dd\"}}\"{{else}}\"{{ date singlejiraissue.fields.duedate 'yyyy-MM-dd'}}\"{{/ifEquals}},\"TypeName\": \"{{ map config.mapping.issuetype.external singlejiraissue.fields.issuetype.id }}\" ,\"PriorityName\": \"{{ map config.mapping.issuepriority.external singlejiraissue.fields.priority.id }}\"{{#ifEquals itmproject.list.0.MethodTypeId '2' }},\"KanbanId\":{{ kanbantaskstatus.KanbanId }}{{else}},\"StatusName\":\"{{ map config.mapping.issuestatus.external singlejiraissue.fields.status.id }}\"{{/ifEquals}} }",

- `ifNotEquals` If 2 values given in the argument are not matched then it executes the first statement otherwise second.
<!-- No examples in existing extensions -->

<!-- Internal  

- `imagevariant` Inserts the third argument at the last index of the second argument in the first argument.
This is not clear. No examples in existing extensions 
- `adfParserDescription` Replaces Double Quotes(") in given argument with &quot; (Specially made for jira issue description)
This is not clear. 
                      "payload": "{\"Name\": \"{{ singlejiraissue.fields.summary }}\",\"Details\": \"{{ adfParserDescription singlejiraissue.fields.description }}\",\"JiraURL\": \"{{ config.url }}\",\"SynchronizationSource\": \"Jira\",\"StartDate\": \"{{ date singlejiraissue.fields.created 'yyyy-MM-dd'}}\",\"EndDate\": {{#ifEquals singlejiraissue.fields.duedate '' }}\"{{DateTime.Now \"yyyy-MM-dd\"}}\"{{else}}\"{{ date singlejiraissue.fields.duedate 'yyyy-MM-dd'}}\"{{/ifEquals}},\"TypeName\": \"{{ map config.mapping.issuetype.external singlejiraissue.fields.issuetype.id }}\" ,\"PriorityName\": \"{{ map config.mapping.issuepriority.external singlejiraissue.fields.priority.id }}\"{{#ifEquals itmproject.list.0.MethodTypeId '2' }},\"KanbanId\":{{ kanbantaskstatus.KanbanId }}{{else}},\"StatusName\":\"{{ map config.mapping.issuestatus.external singlejiraissue.fields.status.id }}\"{{/ifEquals}} }",
 --> -->

## Accessing array of items 
Mustache syntax won't work. To access the first item in a array, instead of something like `taskTeam.list.[0].Team`, it will have to be: `taskTeam.list.First.Team`
<!-- wht? -->

## Type modifiers
To compare or operate with values, you need to specify the type of value. 
<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{
"condition": "Convert.ToInt32(taskdetails.total) > 0 && Convert.ToInt32(taskdetails.list.0.KindId) == 3"
}
```
<i class="fad fa-book-open" title="Guide"></i> **Guide**

To perform logical operations, you need to specify the type of value. The script interpreter will throw an error otherwise:

There are the data converions you can use
- `Convert.ToBoolean()`: Converts a specified value to an equivalent Boolean value.
- `Convert.ToDateTime()`: Converts a specified value to a [DateTime](https://docs.microsoft.com/en-us/dotnet/api/system.datetime?view=net-6.0) value.
- `Convert.ToDecimal ()`: Converts a specified value to a decimal number.
- `Convert.ToDouble()`: Converts a specified value to a double-precision floating-point number.
- `Convert.ToInt16()`:Converts a specified value to a 16-bit signed integer.
- `Convert.ToInt32()`:Converts a specified value to a 32-bit signed integer.
- `Convert.ToInt64()`:Converts a specified value to a 64-bit signed integer.
- `Convert.ToSingle()`: Converts a specified value to a single-precision floating-point number.
- `Convert.ToString()`: Converts the specified value to its equivalent string representation.




<a name="variables"></a>

# Script Variables

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{
    "url": "@@ITMAPI@@/my-company/projects/",
}
```

<i class="fad fa-book-open" title="Guide"></i> **Guide** 

The extension interpreter provides you with variables that you can use. In the example above, we used `@@ITMAPI@@` to call ITM Platform's login endpoint. The outcome of this case would be: `https://api.itmplatform.com/my-company/projects`

<i class="fad fa-brackets" title="Reference"></i> **Reference**

- `@@AccountId@@` : The account Id, such as `18137`
- `@@AccountName@@` : The account name as it appears in the URL. For example `my-company`
- `@@LanguageId@@` :  The user's language code, as follows: en: `1`, es: `2`, pt:`3`
- `@@ITMAPI@@` : The host part of the URL. `https://api.itmplatform.com/`
<!-- - @@TaskMS@@
- @@AccountMS@@ -->

## URL

Let's take the URL as an example.

<i class="far fa-code" title="Learn by example"></i> **Learn by example**

```json
{
"url": "@@ITMAPI@@/@@AccountName@@/login/{{ config.apikey }}",
}
```

<i class="fad fa-book-open" title="Guide"></i> **Guide** 



In the example above, we are using @@<a href="#variables">variables</a>@@ and {{ <a href="#template-syntax">template syntax</a> }} to call ITM Platform's login endpoint. The outcome in this case will be this: `https://api.itmplatform.com/my-company/login/86d4bb2a-2129-40f5-8311-918c6da16823`


# Debugging

To debug an extension, you need to look in the logs to know what happened during its execution. Because the script gets interpreted in ITM Platform's servers, there is no "run and debug" action as you may be used to in other languages.

When an extension is triggered, either by the scheduler or an event, two different types of logs are fed.

- The "Event History" displays a registry of executions, their status and useful information for the end-user.  
- The "Script logs" are available for developers, and they show a registry of all action results.



# Deploying an Extension

To create and deploy a script, you must have the proper license and permissions within ITM Platform. 

1. From the Extension Panel, click on New Extension
1. Fill out the name and other metadata. The extension name will be used internally as the unique identifier and cannot be changed. For example, 'my-extension'. It is case insensitive. No special characters or spaces are allowed
1. Create the script in the "Code Editor" section
1. Activate the extension from the Configuration section


<a name="multi-language"></a>

# Multi-language
- Some fields admit different language versions that will the displayed to the end-user depending on their language configuration. The three accepted languages are English (en), Spanish (es), Portuguese  (pt). The default language is English.

<a name="terminology"></a>

# Terminology
- **Extension:** Apiece of code that adds a specific feature to ITM Platform
- **Connector:** An extension that connects ITM Platform with other applications
- **Extension Script:** Code that defines the behavior of the extension (represented within a JSON structure)
- **Extension Interpreter:** The module within ITM Platform that executes the extension script
- **Entities:** A singular object within ITM Platform. For example, a project or a task
- **System Events:** Signals emitted upon changes within ITM Platform. For example, a task update.
- **Scheduler Events:** Time-based system that specifies a moment in time. For example, every two hours.
- **Trigger:** Any occurrence, such as a system event or a scheduler event.
- **Extension Features:** Set of related actions that will be initiated upon a trigger.
- **Actions:** Pieces of code executed to implement a feature, typically chained to one another.

# Examples

## Obtaining the ITM Platform authentication token

```json
{"actions":[{
    "action": "restcall",
    "url": "https://api.itmplatform.com/myCompany/login/{{ config.apikey }}",
    "method": "POST",
    "description": "Login with apikey",
    "output": "logininfo",
    "payload": "",
    "dataType": "application/json"
},
{
    "action": "restcall",
    "url": "https://api.itmplatform.com/myCompany/projects/22333",
    "method": "GET",
    "token": "{{ logininfo.Token }}",
    "description": "Retrieve project",
    "output": "project",
    "payload": "",
    "dataType": "application/json"
}]}
```

Result object in the `logininfo` variable (`output` of the first action):
```json
{
    "Token": "5955520211116162058986",
    "UserID": "33989",
    "AccountID": "18137",
    "Result": "1",
    "ResultStatus": "Success"
}
```




