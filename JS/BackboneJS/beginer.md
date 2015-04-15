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


Above, we also use the `unset()` method, which removes an attribute by deleting it from the internal model attribute hash.

Validation function can be as simple or complex as necessary. If the attributes provided are valid, nothing should be returned from `.validate()`. If they are invalid, and error value should be returned instead.

Should an error be returned:
  * An `invalid` event will be triggered, setting the `validationError` property on the model with the value which is returned by this method.
  * `.save()` will not continue and the attribute and the attributes of the model will not be modified on the server.

A more complete validation example can be seen below

```javascript
var Todo = Backbone.Model.extend({
      defaults: {
        completed: false
      },
      validate: function(attributes) {
        if(attributes.title === undefined){
          return "Remember to set a title for your todo.";
        }
      },
      initialize: function(){
        console.log('This model has been initialized.');
        this.on("invalid", function(model, error){
          console.log(error);
        });
      }
    });

    var myTodo = new Todo();
    myTodo.set('completed', true, {validate:true}); // logs: Remember to set a title for your todo.
    console.log('completed: ' + myTodo.get('completed')); // completed: false
```
Note: the attributes object passed to the validate function represents what the attributes would be after completing the current `set()` or `save()`. This object is distinct from the current attributes of the model and from the parameters passed to the operation. Since it is created by shallow copy, it is not possible to change any Number, String, or Boolean attribute of the input within the function, but it is possible to change attributes in nested objects.

An example of this (by @fivetanley) is available here.

Note also, that validation on initialization is possible but of limited use, as the object being constructed is internally marked invalid but nevertheless passed back to the caller (continuing the above example):

```javascript
var emptyTodo = new Todo(null, {validate: true});
console.log(emptyTodo.validationError);
```
## Views ##
Views in Backbone don't contain the HTML markup for your application; they contain the logic behind the presentation of the model's data to the user. This is usually achieved using JavaScript templating (e.g., Underscore, Microtemplates, Mustachem jQuery-templ, ect.). A view's `render()` method can be bound to a model's `change()` event, enabling the view to instantly reflect model changes without requiring a full page refresh.


### Creating new views ###
Creating a new view is relatively straightforward and similar to creating new models. To create a new View, simply extend `Backbone.View`. We introduced the sample TodoView below in the previous chapter; now let's take a closer look at how it works:

```javascript
var TodoView = Backbone.View.extend({

  tagName:  'li',

  // Cache the template function for a single item.
  todoTpl: _.template( "An example template" ),

  events: {
    'dblclick label': 'edit',
    'keypress .edit': 'updateOnEnter',
    'blur .edit':   'close'
  },

  initialize: function (options) {
    // In Backbone 1.1.0, if you want to access passed options in
    // your view, you will need to save them as follows:
    this.options = options || {};
  },

  // Re-render the title of the todo item.
  render: function() {
    this.$el.html( this.todoTpl( this.model.attributes ) );
    this.input = this.$('.edit');
    return this;
  },

  edit: function() {
    // executed when todo label is double clicked
  },

  close: function() {
    // executed when todo loses focus
  },

  updateOnEnter: function( e ) {
    // executed on each keypress when in todo edit mode,
    // but we'll wait for enter to get in action
  }
});

var todoView = new TodoView();

// log reference to a DOM element that corresponds to the view instance
console.log(todoView.el); // logs <li></li>
```
### What is el? ###
`el` is basically a reference to a DOM element and all views must have one. Views can use `el` to compose their element's content and then insert it into the DOM all at once, which makes for faster rendering because the browser performs the minimum required number of reflows and repaints.
