# ITM Platform's Plugin Script. Example 4
## Syncs Hubspot's companies into ITM Platform's clients

This example illustrates how to connect to third-party system Rest APIs and how to use [loops](https://github.com/itmplatform/plugin-documentation#action-loop)

If you haven't done so, it is recommended that you read more basic examples first.

The full code is in the JSON file included in this same folder.

### Desired functionality

The plugin has to connect to Hubspot, retrieve companies having the property `sync_itm_platform` set to `true` and create or update the client in ITM Platform, depending on whether it already existed.

We will focus on the `features` section, since the [plugin configuration](https://github.com/itmplatform/plugin-documentation#plugin-configuration) section has no significant changes compared to previous examples.

We also already covered the [Plugin General Definition](https://github.com/itmplatform/plugin-documentation#plugin-general-definition).


## Features and actions
### Basic generic scripting
We know  this is a scheduled set of actions, like so:

```json
"features": [
    {
        "trigger": "scheduler",
        "async": true,
        "actions": []
```
We also have covered before the need to [authenticate](https://github.com/itmplatform/plugin-documentation#obtaining-the-itm-platform-authentication-token) to obtain the token that will gives us access.

```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/@@AccountName@@/login/{{ config.apikey }}",
    "method": "POST",
    "description": "Login with apikey",
    "output": "loginInfo",
    "payload": "",
    "dataType": "application/json"
},
```
Notice we are using again the two [interpreter variables](https://github.com/itmplatform/plugin-documentation#variables): `@@ITMAPI@@` and `@@AccountName@@`. 

### Logic and requirements for this plugin

The logic of the following actions will be:

1. Retrieve companies from Hubspot that have the `sync_itm_platform: true`
1. For each one of them
   1. If the company doesn't exist in ITM Platform, create the client.
   1. Otherwise, update the information of the client.

> :bulb: For simplicity we are using the `FiscalId` field as a way to identify the client with the Hubspot company Id. We could have used any other property, such as the Fiscal Id in hubspot, if it existed.

In order to make this work you will need a couple of things from Hubspot
1. **Hubspot's API** Key, which has a similar format as ITM Platform's. You can read about where to get it [here](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key).
1. Create a company property in Hubspot with the *internal name* `sync_itm_platform`, as explained [here](https://knowledge.hubspot.com/crm-setup/manage-your-properties). The type should be _"single checkbox"_ which will be set to `true` when ticked. 
1. Go ahead and set a few companies in Hubspot with the property `sync_itm_platform` set to `true` so we actually get something synchronized.


### REST Calls
First thing will be to retrieve companies from Hubspot that have the `sync_itm_platform: true`
```json
{
    "action": "restcall",
    "url": "https://api.hubapi.com/crm/v3/objects/companies/search/?hapikey={{ config.apikeyHubspot }}",
    "method": "POST",
    "description": "Retrieve all companies that have the sync_itm_platform field set to true",
    "output": "companiesHS",
    "payload": "{\"properties\": [ \"name\",\"address\",\"city\",\"state\", \"phone\" ], \"filterGroups\": [ { \"filters\": [ { \"propertyName\": \"sync_itm_platform\", \"operator\": \"EQ\", \"value\": \"true\" } ] } ]}",
    "dataType": "application/json"
},
```
Notice we are using the [search](https://developers.hubspot.com/docs/api/crm/search) feature of Hubspot's API and using a [filter](https://developers.hubspot.com/docs/api/crm/search#filter-search-results) that will return the requested properties of all companies matching our criteria.

If all went well, the object `companiesHS` will now hold an object with all matching companies. The following is an example of just one.

``` json
{"total":1,"results":[{"id":"8374988365","properties":{"address":"The address","city":"The city","createdate":"2022-04-02T11:12:48.141Z","hs_lastmodifieddate":"2022-04-05T20:31:44.896Z","hs_object_id":"8374988365","name":"Company Name","phone":"351911717475","state":null},"createdAt":"2022-04-02T11:12:48.141Z","updatedAt":"2022-04-05T20:31:44.896Z","archived":false}]}
```

### Loop
Now, we need to look into each company that we have in `companiesHS`. For this we need to [loop](https://github.com/itmplatform/plugin-documentation#action-loop) through an array.
```json 
{
    "action": "loop",
    "description": "At this point, we have the object companiesHS, we need to loop through them.",
    "loop": {
        "var": "companiesHS.results",
        "output": "singleCompanyHS"
    },
    "actions": [...]
```
Because Hubspot returned all companies in an array called `results` within the object `companiesHS`, the loop "var" will be `"var": "companiesHS.results"`. The content of each iteration will be stored in `singleCompanyHS`

All further actions will be performed within the `actions` array of the loop.

### Loop actions: more REST Calls
According to the specs, for each company, we need to:
1. If the company doesn't exist in ITM Platform, create the client.
1. Otherwise, update the information of the client.

First step is verifying whether the company exists as a client in ITM Platform

```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/@@AccountName@@/clients/?FiscalId={{ singleCompanyHS.id }}",
    "method": "GET",
    "token": "{{ loginInfo.Token }}",
    "description": "Retrieves the client from ITM Platform that matches de Fiscal ID (Company Id on Hubspot)",
    "output": "clientsMatchingFiscalId",
    "payload": "",
    "dataType": "application/json"
}
```
We are using ITM Platform's [GET Clients](https://developers.itmplatform.com/documentation/#clients-clients-get) endpoint to filter by `FiscalId`. Because we established that the equivalent to the Fiscal Id in Hubspot would be the company Id, we can use `?FiscalId={{ singleCompanyHS.id }}`. 

Now we will have an object `clientsMatchingFiscalId` returned by ITM Platform containing zero, one or more clients matching the Fiscal Id. 

### If the company doesn't exist in ITM Platform, create the client
```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/v2/@@AccountName@@/clients/",
    "condition": "Convert.ToInt32(clientsMatchingFiscalId.Count) == 0",
    "method": "POST",
    "token": "{{ loginInfo.Token }}",
    "description": "If clientsMatchingFiscalId.length ==  0, that means we need to create the company on ITM Platform for the first time",
    "output": "",
    "payload": "{\"FiscalId\": \"{{ singleCompanyHS.id }}\", \"Name\": \"{{ singleCompanyHS.properties.name }}\" }",
    "dataType": "application/json"
},
``` 
If there are zero clients, then we know we have no clients with that fiscal ID and we can create it from our previous

We used `"condition": "Convert.ToInt32(clientsMatchingFiscalId.Count) == 0",` to verify that the array is empty. `Count` will determine the length of the array

If the conditions is met, the client is created in ITM Platform using the [POST client](https://developers.itmplatform.com/documentation/#clients-client-v2-post) endpoint.

### Otherwise, update the information of the client
Likewise, if the list of clients is not empty we need to update the existing client

```json
{
    "action": "restcall",
    "url": "@@ITMAPI@@/v2/@@AccountName@@/clients/{{clientsMatchingFiscalId.0.ClientId}}",
    "condition": "Convert.ToInt32(clientsMatchingFiscalId.Count) > 0",
    "method": "PATCH",
    "token": "{{ loginInfo.Token }}",
    "description": "If clientsMatchingFiscalId.length >  0, that means that we just need to update it",
    "output": "",
    "payload": "{\"FiscalId\": \"{{ singleCompanyHS.id }}\", \"Name\": \"{{ singleCompanyHS.properties.name }}\" }",
    "dataType": "application/json"
}
```
To do so, we will use the [PATCH client](https://developers.itmplatform.com/documentation/#clients-client-v2-patch) method, adding the ClientId at the end. For simplicity, we will update the first client, which in normal conditions should be the only one `clientsMatchingFiscalId.0.ClientId`

<a href="../example-5/">Continue to example 5</a>