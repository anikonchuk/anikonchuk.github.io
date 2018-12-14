---
layout: post
title:      "Adding a JavaScript Front End to My Rails Project"
date:       2018-12-14 16:57:13 +0000
permalink:  adding_a_javascript_front_end_to_my_rails_project
---


My fourth portfolio project for the Flatiron School’s Online Full Stack Web Development Program focused on adding JavaScript to my existing Rails project. My goal was to take my project, Quick and Easy Recipes (about which you can read more [here](http://andrewnikonchuk.com/quickandeasyrecipes-_a_rails_app)), and add a JavaScript front end that relied on AJAX (asynchronous JavaScript and XML) to provide for a smooth user experience. In terms of guidelines for the project, we were tasked with rendering at least one index page, one show page, and one has-many relationship via JavaScript and an ActiveModel Serialization JSON (JavaScript Object Notation) backend. We also needed to render a form for creating a resource that submits dynamically, and we had to translate the JSON responses into JavaScript Model Objects with at least one method on the prototype.

## ActiveModel Serializers
My first step in transitioning to a JavaScript front end was ensuring that the data being sent by the server was in the JSON format. To do this, I used ActiveModel Serializers (AMS). AMS takes care of transforming our Ruby model objects to JSON, and it provides a way to customize how the data is represented. The first step was to install the gem to my Gemfile by including gem ‘`active_model_serializers`’ and running `bundle install`. I then began to generate my serializers by running `rails g serializer [model_name]` from my terminal. This creates a file in my app/serializers folder that I could then customize to render the information that I wanted from each model. I was also able to include the relationships between the models so that the related data could be rendered alongside the model’s data. For example, my Recipes model has attributes of id, name, instructions, time, created_at, and updated_at, and it has many quantities and has many ingredients through quantities. However, I didn’t need the serializer to include the timestamps, so I did not include that in the serializer. The full code from my serializer is as follows: 

```
class RecipeSerializer < ActiveModel::Serializer
  attributes :id, :name, :instructions, :time
  has_many :quantities
  has_many :ingredients, through: :quantities
end
```

Once I had my serializer set up for my models, I needed to tell my controllers to render JSON in addition to (or instead of) HTML as responses. One way to do this is to include a `respond_to` block, as shown below:

  ```
def show
  @recipe = Recipe.find_by(id: params[:id])
  respond_to do |f|
    f.html {render :show}
    f.json {render json: @recipe}
  end
end
```

In this case, if I navigated to ‘/recipes/1’, the controller would render the HTML of the show page, but if I navigated to ‘/recipes/1.json’, the controller would render only the well-formatted JSON with the custom fields I described in my serializers. In some cases, instead of using a `respond_to` block, I simply used `render json: @recipe, status: 201`, like when my new Recipe was created. This would only let the controller send back JSON-formatted data. 


## AJAX & Adding Event Listeners
Now that I had my serializers and controllers set up to render JSON data, I could work on transitioning the front end of my app to utilize AJAX to provide a smooth and quick user experience. Instead of locking up the browser and refreshing the entire page each time a link was clicked, AJAX allows the browser to send requests for information in the background and then use the response to update the DOM without a full page refresh. I set up a new ‘landing page’ to which users would be redirected when they logged in. This page, while nearly blank at first, has a `div` with an ID I could target when manipulating the DOM through JavaScript and jQuery. 

The general flow of events when dealing with AJAX is to hijack a link or some other action on the DOM, prevent its default action, request or send information from or to the server (or an external API), and then display information on the DOM. I used my navigation bar on my application layout for links that the user could click to trigger the AJAX events. I added specific ID’s to each of the links for easy targeting with JavaScript. I then added an event listener to each of the links that would then trigger additional actions. For example:

```
function addListenerToAllRecipesLink () {
  const allRecipesLink = document.getElementById("all-recipes-link");
  allRecipesLink.addEventListener('click', function(e){
    e.preventDefault();
    fetchAllRecipeData();
  });
}
```

This function first prevents the default action of the click and then triggers my `fetchAllRecipeData()` function, which uses `fetch()` to get the JSON for all the recipes from the server and display them on the DOM. The `fetchAllRecipeData()` function takes the response from the server, translates it into JSON, and then uses JavaScript Model Objects (see below) to render the information on the DOM.

The event listener function is executed after the window fully loads by wrapping it in the jQuery document ready function:

```
$(function() {
  addListenerToAllRecipesLink();
})
```

I needed to wait until the window was actually fully loaded before attaching the event listener to the link because you can’t attach a listener to something that isn’t present on the DOM. 


## JavaScript Model Objects
According to the Mozilla Developer Network, “JavaScript is an object-based language based on prototypes, rather than being class-based”. However, JavaScript classes, introduced in ECMAScript 2015, provide a clean way to implement JavaScript’s prototype-based inheritance. We can define a class in JavaScript and create methods associated with those classes that can be inherited by any instances of that class. I used JavaScript Model Objects to create functions that returned custom HTML responses based on the class instance. For example, for recipes, I created a recipe class: 

```
class Recipe {
  constructor(attr) {
    this.id = attr.id
    this.name = attr.name
    this.time = attr.time
    this.ingredients = attr.ingredients
    this.quantities = attr.quantities
    this.instructions = attr.instructions
  }
}
```

I was then able to take each Recipe returned from the server and translate it into an instance of the Recipe class. I created a custom function on the class (like `Recipe.prototype.createRecipeDisplay`) that took in that instance and rendered it in HTML. All instances of the class inherited that function. 


## Rendering the Form
While most of the AJAX requests users make in my app return JSON, I approached rendering the form for a new Recipe differently. The form I had in my original Rails app is a double-nested form with a large number of fields. While I could theoretically build it using HTML in JavaScript, it made no sense to recreate the work that Rails was already doing for me, especially using the nested resource URL for the action. Therefore, I decided to pull in the form partial as HTML when users clicked the New Recipe link. I could then inject this partial directly into the DOM instead of manipulating it further. In order to render just the partial, my new action in my Recipes Controller used the following code: `render :partial => 'recipes/form', :layout => false`.


## Using jQuery to Submit Form Data
Once the form was loaded and users filled in data, I needed to hijack the form’s submit action and dynamically submit the form using AJAX. I used jQuery instead of vanilla JavaScript for this because it provided cleaner code and was easier to use to manipulate and send the data to the server. Within the `addEventListener` function for the form, I used `this.action` to extract the URL to which I needed to post the data, and then I used jQuery to extract the data and translate it to a JSON string by using `$(this).serialize()`. I then used `$.ajax()` to send a “POST” request to the server with the data. Upon successful creation of a new recipe, the server responds with the JSON of the new recipe. If the data does not meet the validations on the Recipe model, I display a generic message about what needs to be input on the form. I could not figure out a way to extract the specific error data on the recipe object, but will continue to investigate a way to find and display the specific errors. 

## Conclusion & Reflection
Overall, I am happy with how my application turned out. The AJAX front end is fast and responsive. I feel I solidified my understanding of JavaScript, jQuery, and AJAX, and was able to effectively implement them to provide a positive user experience. 

There are still improvements that can be made to the app going forward. I would like to be able to add edit and delete functionality for users. I could also create links to view the index pages organized by name, etc., using my scope methods. However, these functionalities were beyond the scope of this individual project, so I plan to tackle them at a future date. 

If you would like to view my source code for the project, it is available at https://github.com/anikonchuk/QuickAndEasyRecipes-JS.
