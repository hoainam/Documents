# BACKBONE JS #

## Listening for changes to your model ##
If you want to receive a notification when a Backbone model changes you can bind a listener to the model for its change event. A convenient place to add listeners is in the `initialize()` function.

You can also listen for change to individual attributes in a Backbone model. In the following example, we can a message whenever a specific attribute (the title of our Todo model) is altered.

## Validation
Backbone supports model validation through `model.validate()`, which allows checking the attribute values for a model prior to setting them. By default, validation occurs when the model is persisted using the `save()` method or when `set()` is called if `{validate:true}` is passed as an argument.  

```javascript
var Person = new Backbone.Model({name: 'Jeremy'});

// Validate the model name
Person.validate = function(attrs) {
  if (!attrs.name) {
    return 'I need your name';
  }
};

// Change the name
Person.set({name: 'Samuel'});
console.log(Person.get('name'));
// 'Samuel'

// Remove the name attribute, force validation
Person.unset('name', {validate: true});
// false
```


Above, we also use the unset() method, which removes and attribute by deleting it from the internal model attribute hash.
