# BACKBONE JS #

## Listening for changes to your model ##
If you want to receive a notification when a Backbone model changes you can bind a listener to the model for its change event. A convenient place to add listeners is in the `initialize()` function.

You can also listen for change to individual attributes in a Backbone model. In the following example, we can a message whenever a specific attribute (the title of our Todo model) is altered.

## Validation
Backbone supports model validation through `model.validate()`, which allows checking the attribute values for a model prior to setting them. By default, validation occurs when the model is persisted using the `save()` method or when `set()` is called if `{validate:true}` is passed as an argument.  
