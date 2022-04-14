# ITM Platform's Plugin Script. Example 2
## When a purchase updates, send a message if the actual amount exceeds the projected amount

This example illustrates how to use conditionals. If you haven't done so, it is recommended that you read more basic examples first.

The full code is in the JSON file included in this same folder.

### Desired functionality

When any purchase updates, the plugin must verify whether the Actual Amount is greater than the Projected Amount. In that case, it will send an email alert.

For the sake of simplicity, we will use a static email address `your.friend@itmplatform.com`

We will solely focus on the `features` because we already covered the [Plugin General Definition](https://github.com/itmplatform/plugin-documentation#plugin-general-definition) in previous examples and the [plugin configuration](https://github.com/itmplatform/plugin-documentation#plugin-configuration) has no additional settings.

## Features and actions

The specification tells us we need to trigger the feature when a purchase is updated. 

We already know how to configure that, and how to configure an email action.


```json
{"features": [
		{
			"trigger": "event",
			"event": "updated",
			"entity": "Purchase",
			"condition": "Convert.ToDouble(input.purchase.ActualAmount) > Convert.ToDouble(input.purchase.ProjectedAmount)",
			"description": "Purchase update Event, which will send an email",
			"actions": [
				{
					"action": "email",
					"to": "your.friend@itmplatform.com",
					"subject": "Purchase actual value exceeds projected amount",
					"body": "<html><p>The purchase actual value ({{input.purchase.ActualAmount}}) exceeds projected amount ({{input.purchase.ProjectedAmount}})</p></html>"
				}
			],
			"async": true
		}
	],
```
We will now focus on the condition line:

```json
{"condition": "Convert.ToDouble(input.purchase.ActualAmount) > Convert.ToDouble(input.purchase.ProjectedAmount)"}
```
In the documentation we can find the description of [action conditionals](https://github.com/itmplatform/plugin-documentation#action-conditionals), that will determine whether the feature or actions will be executed based on the conditions specified.

In our case, we need to send the email when `ActualAmount` is greater than `ProjectedAmount`.

Now, as explained in the [type modifiers](https://github.com/itmplatform/plugin-documentation#type-modifiers) section, we need to specify what kind of value is being operated with to perform logical operations. In this case, we chose `Convert.ToDouble`, which is a "double-precision floating-point number"

This is all you need to create your first plugin managing conditions.

<a href="../example-3/">Continue to example 3</a>
