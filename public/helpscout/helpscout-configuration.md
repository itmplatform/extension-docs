## Help Scout Configuration

This specification explains the steps to configure the webhook API and generate the Secret Key and ID for an OAuth 2.0 token in Help Scout.

### Configuration Steps

1. **Generate the Client ID and Secret Key for authentication:**
   - Go to `My Profile` and navigate to `My Apps` (last item in the menu).
   - Click on the `Create My App` button.
   - Enter any app name and a redirection URL, then create the app.
   - Upon creation, Help Scout will provide an `App ID` and `App Secret`.  
     - Copy and paste these into the ITM Platform extension configuration.

2. **Create a webhook:**
   - Go to `Apps` and open the Apps Directory from the `Manage` tab in the header navigation.
   - Search for `Webhook`.
   - Open the `Real-time notifications from Help Scout` option.
   - Generate a new webhook with any name and a secret key.
   - Add the ITM Extension URL in the `Callback URL`, e.g.,  
     `https://api.itmplatform.com/v2/{your_copmpany}/webhooks/itm-helpscout`
   - Choose the event that should trigger the webhook and activate it.

3. **Create a custom field for ITM Task ID:**
   - Open `Inbox` from the navigation and click on the settings icon.
   - Open `Custom Fields` from the menu.
   - Create a new custom field with the name `ITMTaskId` and type `Single Line`.
