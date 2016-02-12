# Introduction to Ember

## What is Ember?

Ember is a front-end framework for building web applications. It uses the MVC framework to serve data to views that update automatically as the underlying data changes. Ember makes building a responsive web application super easy. Using Ember controllers and components, allowing our app to respond to the user's actions on the page is fast and easy. Even complex interacts and changes to the DOM can be implemented with relative simplicity. Ember also makes data retrieval and serialization simple: you don't need to display JSON retrieved from your server. Ember does it for us!

Ember doesn't have its own persistence layer. It communicates with an external API or a database like Firebase to get and post data. But don't worry! Ember makes data retrieval and serialization simple: you don't need to display JSON retrieved from your server. Ember does it for us!

Let's take a closer look at how MVC architecture is implemented in Ember. This reading will include some code snippets. Don't worry about understanding every line of Javascript. Instead, focus on understanding the big picture: how do the different pieces of an Ember app fit together?

## Before You Begin

This reading will include some code snippets. Don't worry about understanding every line of Javascript. Instead, focus on understanding the big picture: how do the different pieces of an Ember app fit together?

Our code snippets will be based on a simple app in which Ember connects to a Rails API that serves data describing graduates of the Flatiron School.

# Ember MVC

In Ember, the MVC framework is *slightly* different from Rails. In Ember, we have the following:

* **Models:** Just like in Rails, Ember models provide objects with which to wrap the database records. 
* **Views:** Ember templates are the view component of the Ember app. These templates use the Handlebars templating language, which allows us to inject data dynamically into the view. This is the equivalent of ERB in our Rails views. 
* **Controllers:** In Ember, the job of the controller (getting data and passing it to the view) is actually handled not by controllers but by route handlers. We'll take a closer look at this in a minute. 

There are other aspects of an Ember app too:

* In Ember, the controller is actually responsible for decorating data with with display logic and changing that decoration in response to the users' actions (i.e. action handling). 
* The adapter is responsible for connecting our app to the external API or database from which it retrieves data. 
* Components are similar to partials in Rails––they allow us to bundle up and re-use portions of our templates. But, components are smart––they can have their own properties and actions. 

Let's take a closer look at the elements of an Ember app now. 

## The Router and Route Handlers

Routes in Ember function exactly as they do in Rails. Routes receive a web request and send that request to the appropriate part of your app. However, in Rails, a route sends a request to the appropriate controller action, which gets the requested data and passes it to the view. In Ember, the route passes the request to the *route handler*, which does the job of the Rails controller: get the requested data and pass it to a view, or template, as we say in Ember Land. 

Let's take a look at our router for our Flatiron Grad app:

```javascript
// app/router.js

import Ember from 'ember';  
import config from './config/environment';

var Router = Ember.Router.extend({  
  location: config.locationType
});

export default Router.map(function() {  
  this.route('graduates');
});
```

Now, when a user tries to visit `www.ourapp.com/graduates`, Ember will send that request to the route handler that corresponds to this URL:

```javascript
// app/routes/graduates.js

import Ember from 'ember';

export default Ember.Route.extend({  
  model: function() {
    return all of the graduates
  }
});
```

The route handler sets a property, `model`, equal to a function that returns the requested data, in this case all of the graduates. We'll come back to how to grab all of the graduates from the API later. For now, we'll just hard-code in some graduate data to return:

```javascript
// app/routes/graduates.js

import Ember from 'ember';

export default Ember.Route.extend({  
  model: function() {
    return [{firstName: "Sophie", github: "SophieDeBenedetto"},
    {firstName: "Alex", github: "kkomaz"},
    {firstName: "Jeremy", github: "jeremysklarsky"]
  }
});
```

So, to recap: 

* A user visits `/graduates` and the router passes that request to the route handler. 
* The route handler invokes the model function, the return value of which is a collection of graduate objects.
* Then, the graduates template is rendered, on which the `model` object can be accessed and its data rendered. Let's take a closer look at that right now.

## Templates

Ember.js uses the Handlebars templating library to render data for the client. If you're familiar with Rails, you can think of Handlbars + HTML like ERB + HTML. I think the Ember documentation says it best:

> It may help to think of your Handlebars templates as an HTML-like DSL for describing the user interface of your app. And, once you've told Ember.js to render a given template on the screen, you don't need to write any additional code to make sure it keeps up-to-date.

### Handlebars Expressions

What are Handlebars Expressions?

Handlebars expressions are essentially Ember Data objects attributes wrapped in `{{}}`, or double curly braces. These expressions are bindings aware. That means that if the values used by your templates ever change, your HTML will be updated automatically. 

Let's take a closer look at our graduates templates in order to get a better understanding of how this works.

```javascript
// app/templates/graduates.hbs

<ul>
  {{#each model as |graduate|}}
     <li>Name: {{graduate.firstName}}</li>
     <li>github username: {{graduate.github}}</li>
  {{/each}}
</ul>
```

Here, we use Handlebars expressions to inject the `model` object onto the page. We iterate over the collection of graduates in that `model` object (which was set by the route handler) use a Handlebars helper expression, `{{each}}`. Handlebars provides us with a number of helpers, like `{{each}}`, `{{if}}` and `{{link-to}}`. 

When we say that our Handlebars expressions are bindings aware, that means that we can define an action in a controller (more on that later) that changes or updates the data in the model, based on a user's action, like clicking on a certain element or button. Those changes will then automatically appear on the page, without the page refreshing and without us needing to use any AJAX or jQuery to update the DOM. This is what we mean when we say that Ember apps are responsive. 

Let's take a step back and look at one more important element of our Ember templates, the `application.hbs` layout. 

### The Application Template

The Ember docs describe the application template like this:

>The application template is the default template that is rendered when your application starts.

It is exactly equivalent to the `app/views/layouts/application.html.erb` view in Rails.

You should put your header, footer, and any other decorative content here. Additionally, you should have at least one `{{outlet}}`: a placeholder that the router will fill in with the appropriate template, based on the current URL. This is equivalent to the Rails view's use of `<%= yield%>`

Here's the application template for our Flatiron Grad app. It has a header and footer, and it also includes the Handlebars `{{outlet}} `expression for rendering the appropriate data, depending on the URL of the page.

```html
<nav class="navbar navbar-default my-nav">  
  <div class="container-fluid">
    <div class="navbar-header">
    {{#link-to 'index' class="my-logo navbar-brand"}}<h2>// Flatbook</h2>{{/link-to}}
    </div>

    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav">
        <li class="my-nav-item">{{link-to '// Graduates' 'graduates'}}</li>
      </ul>
    </div>
  </div>
</nav>


{{outlet}}

<footer class="hidden-xs">  
    <p class="pull-left">&copy; Copyright 2014. Rebound.</p>
    <p class="pull-right">made with <span class="glyphicon glyphicon-heart-empty"></span> <a href="http://www.sophiedebenedetto.nyc">Sophie DeBenedetto</a> @ <a href="http://flatironschool.com">The Flatiron School</a></p>
</footer>  
```

So far we have a working app that allows us to visit the `/graduates/` page and see a rendering of the collection of graduates that we hard-coded into the route handler. Let's step it up a bit and incorporate real data retrieved from an API, wrapped by an Ember Model. 

## Connecting to an API: The Ember Adapter

In order to tell our Ember application to retrieve data from our Rails API, we need to build an an application adapter: `app/adapters/application.js`.

```javascript
import DS from 'ember-data';

export default DS.ActiveModelAdapter.extend({  
  namespace: 'api/v1',
  host: 'https://flatbook-api.herokuapp.com'
});
```

Let's take a closer look at what's going on here. First things first:

### What is Ember Data?

Ember Data is a library for managing model data an Ember app. Ember Data is persistence agnostic––it doesn't care where the model data comes from (an API, a Firebase database, you name it). 

All ember data objects are namespaced under DS.

### Understanding our Adapter

Let's revise the flow of our Ember app now that we're using an adapter to request data from an API:

* A user visits '/graduates'.
* The router sends that request to the route handler in `app/routes/graduates.js`. 
* The route handler requests all the graduates from the data store (more on the code that actually sends that request in a moment). 
* The data store passes that request on to the adapter and the adapter sends the request to the appropriate API endpoint.
* The adapter receives the response from the API, puts it in the data store, and the route handler gets the data out of the store.
* The route handler sets the `model` property equal to that data and passes `model` down into the template. 

Phew! That is a lot of bullet points. 

So, with all of this in mind, we can define the adapter as an object that receives requests from a store and then makes the appropriate request to your app's persistence layer––usually an HTTP API (in this case, our Rails API).

The adapter for our Ember app is a REST adapter. The REST adapter allows our store to communicate with an HTTP server by transmitting JSON via XHR. Most Ember.js apps that consume a JSON API should use the REST adapter.

Sidenotes before continuing...

### What is the Ember store?

You Ember app's store contains all of the data for records loaded from the server. It is also responsible for creating instances of DS.Model that wrap the individual data for a record, so that they can be bound to in your Handlebars templates.

### What is XHR?

XMLHttpRequest is an API available to web browser scripting languages (like JavaScript). It is used to send HTTP or HTTPS requests to a web server and load the response data back into the script.

### Back to our adapter...

Our adapter is an extension of the ActiveModelAdapter. The ActiveModelAdapter is a subclass of the RESTAdapter designed to integrate with a JSON API that uses an underscored naming convention instead of camelCasing, which perfectly describes our Rails API. The ActiveModelAdapter subclass has been designed to work with the activemodelserializers Ruby gem, which is exactly what we used to serialize our data in our Rails API.

From the Ember docs: 

> this adapter extends the DS.RESTAdapter by making consistent use of the camelization, decamelization and pluralization methods to normalize the serialized JSON into a format that is compatible with a conventional Rails backend and Ember Data.

*In other words, the ActiveModelAdapter will help to pair the data that comes down to the Ember app from the Rails API with it's JS camelCase equivalent.*

For example, in our Rails API, we have a Graduate model that produces graduate objects. Each graduate has a first_name, among other attributes. Our API serializes these attributes and sends them to requesters (like our Ember app) as JSON that would represent a first_name attribute as "first_name". Our Ember Graduate model will have a corresponding attribute, firstName and our adapter will connect the two so that the Ember graduate object can take the firstName of a corresponding JSON "first_name" node. More on this in just a minute, but first...

Let's take a step-by-step look at our adapter:

* Import the Ember Data library for managing our models:

```javascript
// app/adapters/application.js

import DS from 'ember-data';

...
```
* Extend, or add functionality to, Ember's Adapter class:

```javascript
export default DS.ActiveModelAdapter.extend({  
  namespace: 'api/v1',
  host: 'https://flatbook-api.herokuapp.com'
});
```
 
The code inside of our extension of the adapter tells our adapter object to respond to requests for data by sending HTTP requests to our Rails API hosted at `https://flatbook-api.herokuapp.com.` 

Now, let's update our route handler so that it asks the store for the graduate records, instead of hard-coding the `model` to be a collection of three made-up graduates.

#### Updating the Route Handler to Request Data from the Store

Currently our route handler looks like this:

```javascript
// app/routes/graduates.js

import Ember from 'ember';

export default Ember.Route.extend({  
  model: function() {
    return [{name: "Sophie", github: "SophieDeBenedetto"},
    {name: "Alex", github: "kkomaz"},
    {name: "Jeremy", github: "jeremysklarsky"]
  }
});
```
Let's change that model function so that it requests graduates from the store:

```javascript
import Ember from 'ember';

export default Ember.Route.extend({  
  model: function() {
    return this.store.find('graduate');
  }
});
```

Now, when a user visits `/graduates`, the route handler will request graduate data from the store, which will pass that request on to the adapter. The adapter will get the data from the API, wrap that data into Javascript objects using the guidelines provided by the Ember model (which we haven't set up yet) and put in in the store. 

Let's set up our model. 

## The Ember Model

### What is an Ember Model?

A model is a class that defines the properties and behavior of the data that you present to the user. DS.Model is the model class that all Ember Data records descend from. 

### Our Adapter

In `app/models/graduate.js` we should have the following:

```javascript
import DS from 'ember-data';

export default DS.Model.extend({  
  firstName: DS.attr('string'),
  github: DS.attr('string'),
  createdAt: DS.attr('date'),
  updatedAt: DS.attr('date'),
});
```
As described above, our adapter is responsible for connecting the JSON that is served to the Ember app by our Rails API to the Ember Data objects that will be presented to our viewer. So, when we set each attribute via lines like this:

```javascript
firstName: DS.attr('string'),  
```

We are telling the first name attribute of a the Graduate model to be whatever is provided by the server. Meanwhile, the adapter takes over the job of matching up `firstName` attributes of Ember graduates with `first_name` attributes of Ruby objects, sent to our Ember app via our Rails API as JSON.

We're almost done! The last thing we need to do is update our router so that it knows we are routing requests for resources that will ultimately be retrieved from the data store and an external API:

```javascript
import Ember from 'ember';  
import config from './config/environment';

var Router = Ember.Router.extend({  
  location: config.locationType
});

export default Router.map(function() {  
  this.resource('graduates')
  });
});
```

Notice we changed `this.route` to `this.resource`.

## Conclusion

Now we see how the different components of our Ember application work together to implement the MVC framework. Remember that, in Ember, the controller's responsibilities are handled by the route handlers. The controller, in Ember, has a different job, and we'll learn more about that later. 

Once again, just to recap, here's the flow of data through our Ember app:

* A user visits '/graduates'.
* The router sends that request to the route handler in `app/routes/graduates.js`. 
* The route handler requests all the graduates from the data store. 
* The data store passes that request on to the adapter and the adapter sends the request to the appropriate API endpoint.
* The adapter receives the response from the API, puts it in the data store, and the route handler gets the data out of the store.
* The route handler sets the `model` property equal to that data and passes `model` down into the template to be rendered.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/ember-mvc-intro' title='ember-mvc-intro'>ember-mvc-intro</a> on Learn.co and start learning to code for free.</p>


<p data-visibility='hidden'>View <a href='https://learn.co/lessons/ember-mvc-intro'>Introduction to Ember MVC</a> on Learn.co and start learning to code for free.</p>
