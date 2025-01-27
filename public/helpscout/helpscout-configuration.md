##  Helpscout Configuration
Specification explain about step to configure webhook API and generate Secret key, ID for OAuth2.0 token in helpscout

### Configuration Steps
    - First step is to generate Client ID and Secret Key for Authentication
	    - Go to My Profile and Go to `My Apps` last item in menu
	    - Click on `Create My App` button
        - Enter any APP Name and Redirection URL and create
        - Upon creation it will provide `App ID` and `App Secret`, paste these in extension configuration in ITM Platform
    - Second step is to create a webhook.
        - Go to `Apps` an Apps Directory from `Manage` tab in header navigation
        - Search for Webhook
        - Open `Real-time notifications from Help Scout` option
        - Generate new Webhook with any name and screte key.
        - Add ITM Extension URL in Callback URL e.g `https://new-api.itmplatform.com/revamping/v2/testsmarter/webhooks/itm-helpscout`
        - Choose the event upon this webhook needed to be trigger and make it active.
    - Last step is to create a Custom field for ITM Task Id
        - Open `Inbox` from navigation and click on setting icon 
        - Open custom field from menu
        - Create a new custom field with name `ITMTaskId` and `singleline`  type
