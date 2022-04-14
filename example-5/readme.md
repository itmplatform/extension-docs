# ITM Platform's Plugin Script. Example 5
## Restricts actual revenue amount to the projected amount limit

This example illustrates how to [validate](https://github.com/itmplatform/plugin-documentation#action-validate) a user input to match business rules.

If you haven't done so, it is recommended that you read previous examples first.

The complete code is in the JSON file included in this same folder.

### Desired functionality

The plugin must validate that the `actual amount` entered does not exceed the `projected amount`, both when creating or updating revenues.

## Features and actions
## Triggers
In this example, we will need two features. One  will trigger based on the [event](https://github.com/itmplatform/plugin-documentation#event-reference) of inserting a revenue and another when updating the revenue.

But because we need the trigger *before* the actual execution of the event, we will use `preInsert` and `preUpdate`.

We also want to trigger these validations only when the actual and projected amounts are higher than zero.

Let's see the first case when the user wants to create a new revenue with actual and projected amounts
```json
{
"features": [
    {
        "trigger": "event",
        "entity": "Revenue",
        "event": "preInsert",
        "condition": "Convert.ToDecimal(input.ActualAmount) > 0 && Convert.ToDecimal(input.ProjectedAmount) > 0",
        "async": false,
        "actions": [...]
    }
```

Takeaways from the trigger definition:
- The trigger will be the `preInsert` event on the `Revenue` entity.
- It will only run when the [condition](https://github.com/itmplatform/plugin-documentation#action-conditionals) actual and projected amount are higher than zero. 
- :warning: `"async": false` indicates that the execution of a user operation on ITM Platform will be interrupted ([read more](https://github.com/itmplatform/plugin-documentation#features)) until the actions are executed. This is exactly what we need in this case since it is a matter of validating user input.

## Actions
We can now add the [validation action](https://github.com/itmplatform/plugin-documentation#action-validate) we need to run:
```json
{"features": [
{
    "trigger": "event",
    "condition": "Convert.ToDecimal(input.ActualAmount) > 0 && Convert.ToDecimal(input.ProjectedAmount) > 0",
    "entity": "Revenue",
    "event": "preInsert",
    "async": false,
    "actions": [
        {
            "action": "validate",
            "validateCondition": "Convert.ToDecimal(input.ActualAmount) > Convert.ToDecimal(input.ProjectedAmount)",
            "message": "Actual amount cannot exceed the projected amount."
        }
    ]
}
```
The two relevant entries of the validation action are:
- `"validateCondition": "Convert.ToDecimal(input.ActualAmount) > Convert.ToDecimal(input.ProjectedAmount)"`.
   - The `input` object holds the data coming from the trigger, `input.ActualAmount` and `input.ProjectedAmount` 
   - To be able to compare them, we need to [convert the type](https://github.com/itmplatform/plugin-documentation#type-modifiers) to decimal (amounts have decimals).
   - And the `message` will be displayed to the user if the validation doesn't pass.

If the condition is not met, the user operation (insert or update a revenue in this case) will be canceled.

## Update a revenue
All the above must be repeated to verify these conditions when the revenue is updated (rather than created).

The only difference will be the event, which will not be `preUpdate`, like so:
```json
{
"trigger": "event",
"condition": "Convert.ToDecimal(input.ActualAmount) > 0 && Convert.ToDecimal(input.ProjectedAmount) > 0",
"entity": "Revenue",
"event": "preUpdate",
"async": false,
"actions": [
    {
        "action": "validate",
        "validateCondition": "Convert.ToDecimal(input.ActualAmount) > Convert.ToDecimal(input.ProjectedAmount)",
        "message": "The actual amount cannot exceed the projected amount."
    }
]
}
```