# BACKBONE JS #

## Listening for changes to your model ##
If you want to receive a notification when a Backbone model changes you can bind a listener to the model for its change event. A convenient place to add listeners is in the `initialize()` function.
> Nếu bạn muốn nhận một thông báo khi một Backbone model thay đổi , bạn có thể rằng buộc một listener đến model cho sự kiện thay đổi  của nó. Địa điểm thích hợp nhất để thêm listener là trong function `initialize()`.

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

There are two ways to associate a DOM element with a view: a new element can be created for the view and subsequently added to the DOM or a reference can be made to an element which already exists in the page.

If you want to create a new element for your view, set any combination of the following properties on the view: `tagName`, `id`, and `className`. A new element will be created for you by the framework and a reference to it will be available at the `el` property. If nothing is specified `tagName` defaults to `div`.

In the example above, `tagName` is set to 'li', resulting in creation of an li element. The following example creates a ul element with id and class attributes
```javascript
var TodosView = Backbone.View.extend({
  tagName: 'ul', // required, but defaults to 'div' if not set
  className: 'container', // optional, you can assign multiple classes to
                          // this property like so: 'container homepage'
  id: 'todos' // optional
});

var todosView = new TodosView();
console.log(todosView.el); // logs <ul id="todos" class="container"></ul>
```
The above code creates the DOM element below but doesn't append it to the DOM.
```html
<ul id="todos" class="container"></ul>
```
If the element already exists in the page, you can set `el` as a CSS selector that matches the element.
```javascript
el: '#footer'
```
Alternatively, you can set `el` to an existing element when creating the view:
```javascript
var todosView = new TodosView({el: $('#footer')});
```
Note: When declaring a View, `options`, `el`, `tagName`, `id` and `ClassName` may be defined as functions, if you want their values to be determined at runtine.

### elennd() ###
View logic often needs to invoke jQuery or Zepto functions on the `el` element and elements nested within it. Backbone makes it easy todo so by defining the `$el` function. The `view.$el` property is equivalent to `$(view.el)` and `view.$(selector)` is equivalent to `$(view.el).find(selector)`. In our TodoView example's render method, we see `this.$el` used to set the HTML of the element and `this.$()` used to find subelements of class 'edit'.

### setElement ###
If you need to apply an existing Backbone view to a different DOM element `setElement` can be used for this purpose. Overriding this.el needs to both change the DOM reference and re-bind events to the new element(and unbind from the old).

`setElement` will create a cached `$el` reference for you, moving the delegated events for a view from the old element to the new one.
```javascript
// We create two DOM elements representing buttons
// which could easily be containers or something else
var button1 = $('<button></button>');
var button2 = $('<button></button>');

// Define a new view
var View = Backbone.View.extend({
      events: {
        click: function(e) {
          console.log(view.el === e.target);
        }
      }
    });

// Create a new instance of the view, applying it
// to button1
var view = new View({el: button1});

// Apply the view to button2 using setElement
view.setElement(button2);

button1.trigger('click');
button2.trigger('click'); // returns true
```
The `el` property represents the markup portion of the view that will be rendered; to get the view to actually render to the page, you need to add it as a new element or apprend it to an existing element.

```javascript
// We can also provide raw markup to setElement
// as follows (just to demonstrate it can be done):
var view = new Backbone.View;
view.setElement('<p><a><b>test</b></a></p>');
console.log(view.$('a b').html()); // outputs "test"
```
### Understanding `render()` ###
`render()` is an optional function that defines the logic for rendering a template. We'll use Underscore's micro-templating in these examples, but remember you can use other templating framerworks if you prefer. Our example will reference the following HTML markup:
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title></title>
  <meta name="description" content="">
</head>
<body>
  <div id="todo">
  </div>
  <script type="text/template" id="item-template">
    <div>
      <input id="todo_complete" type="checkbox" <%= completed ? 'checked="checked"' : '' %>>
      <%= title %>
    </div>
  </script>
  <script src="underscore-min.js"></script>
  <script src="backbone-min.js"></script>
  <script src="jquery-min.js"></script>
  <script src="example.js"></script>
</body>
</html>
```
The `_.template` method in Underscore compiles Javascript templates into function which can be evaluated for rendering. in the TodoView, I'm passing the marking from the template with id `item-template` to `_.template()` to be compiled and stored in the todoTpl property when the view is created.

The `render()` method uses this template by passing it the `toJSON()` encoding of the attributes of the model associated with the view. The template returns its markup after using the model's title and completed flag to evaluate the expressions containing them. I then set this markup as the HTML content of the `el` DOM element using the `$el`property.

This populates the template, giving you a data-complete set of markup in just a few short lines of code.


A common Backbone convention is to return `this` at the end of `render()`. This is useful for a number of reasons, including:
  * Making views easily reusable in other parent views.
  * Creating a list of elements without rendering and painting each of them individually, only to be drawn once the entire list is populated.
