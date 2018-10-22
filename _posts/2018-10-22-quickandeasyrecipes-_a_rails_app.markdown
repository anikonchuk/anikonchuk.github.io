---
layout: post
title:      "QuickAndEasyRecipes- A Rails App"
date:       2018-10-22 12:00:44 -0400
permalink:  quickandeasyrecipes-_a_rails_app
---


For my Rails project for the Flatiron School’s Online Full Stack Web Development Program, we were tasked with creating a Content Management System. We were tasked with building a complete Ruby on Rails application that manages related data through complex forms and RESTful routes. 

My application, QuickAndEasyRecipes, aims to provide a simple interface for storing and sharing, you guessed it, quick and easy recipes. Users are able to create a new recipe and edit that recipe through the web interface. They can also see recipes that other users have created. Recipes are presented in a table that also shows the time it takes to make the recipe and the number of ingredients required. They can be sorted by name, by date added, and by shortest time to make. Users can also browse by ingredients used in recipes. 

![](https://i.imgur.com/GODWh9W.png?1)


## Models and Relationships

I had planned four models for my app, with relationships between each model, detailed below.

* **User**- My user model initially had a username and a password_digest for fields. I used password_digest to work with the `bcrypt` gem and the `has_secure_password` macro to secure the passwords of my users.  The `bcrypt` gem hashes and salts the password so it cannot be easily unencrypted. Eventually, I changed username to email address to deal with third-party authentication (see “Authentication and OmniAuth” below). In terms of relationships, a User `has_many` Recipes. Validations ensure that a User has an email address and that it is unique, and the `has_secure_password` macro ensures that a User has a password.
* **Recipe**- My Recipe model has fields for name, time required, and instructions. It `belongs_to` a User, so it also has a field for user_id. The Recipe `has_many `Quantities, and it also `has_many` Ingredients through Quantities. Validations ensure that a recipe has a name, time, and instructions.
* **Quantity**- Quantity serves as my join table between Recipes and Ingredients. Therefore, it `belongs_to` a Recipe and `belongs_to` an Ingredient, and has fields for a recipe_id and ingredient_id. It also has a user-submittable attribute in amount, which is validated for presence.
* **Ingredient**- The Ingredient model has a name attribute. It `has_many` Quantities, and it `has_many` Recipes through Ingredients. While there is no model-level validation on the Ingredient name, an Ingredient is only added through a custom setter method in the Recipe model if the name is present. 

## Nested Resources

I used nested resources both to display a User’s own recipes and to ensure that only valid users could create and edit recipes. I nested certain Recipe resources under Users as shown in the code below:

```
resources :users, only: [:new, :create] do
  resources :recipes, except: [:show, :destroy]
end
```

In addition to the users/new and users/create routes, this code created routes consisting of `get /users/:user_id/recipes `for the User’s own index route, `get /users/:user_id/recipes/new` for Users to present a form for a new recipe, `post /users/:user_id/recipes` for the create route,` get /users/:user_id/recipes/:id/edit` to present a form for users to edit their own recipes, and `patch/put /users/:user_id/recipes/:id` for users to update their recipe. Controller actions checked to ensure that only the users who owned the recipes could see their own index and create/edit their own recipes. 

## Class-Level ActiveRecord Scope Methods

In order to sort and manipulate the ActiveRecord data for Recipes, I created a number of scope methods in the Recipe model. For example, to display the Recipes in order of shortest time to make, I created the scope method:

`scope :by_shortest_time, -> { order(time: :asc) }`

I could then call Recipe.by_shortest_time in the controller to pull up the list of all Recipes by shortest time. This method is equivalent to the class method:

```
def self.by_shortest_time
  self.order(time: :asc)
end
```

I also created scope methods that sorted the Recipes by name, by most recently added, and by fewest ingredients.

## Issues with a Nested Form

In order to capture data to create and edit Recipes, I also needed to collect data about Quantities and Ingredients. I needed to create a nested form for this, along with a custom setter method in my Recipe model. Initially, my nested form was structured as follows (with individual fields eliminated for clarity of nested relationship):

```
form_for [@user, @recipe] do |f|
  f.fields_for :ingredients do |i|
   i.fields_for :quantities do |q|
   end
  end
end
```

I added the `accepts_nested_attributes_for` macro to both my Recipe model and my Ingredients model, and built out “dummy” objects in my controller so that the form builder had objects to work off of. My custom setter method built new ingredients if the name was not blank, and then built new quantities off of that ingredient. 

I managed to get the form to work when creating new instances of Recipe. However, there were two problems with the edit/update process. First, the form would not pre-populate the quantities in the edit form. Second, and more disturbingly, whenever I tried to update a recipe, the ingredients would duplicate themselves.

Dealing with the second problem first, and with the help of Luisa Scavo during Portfolio Project Prep Study Group, I realized that each time I was updating, I was creating completely new Quantity objects each time I was updating, so the old join model instances were still present. In my custom setter method, I needed to first destroy all the original join model instances that initially existed before building new ones. 

The second issue was a bit more complicated. With the help of Jon Foster, we were able to figure it out. In looking at a pry session initiated when loading the edit form was being loaded, we looked at the form builder object `i` and `q`. The `q.object` was missing when loading. Because an ingredient has many quantities, it wasn’t sure which quantity we were looking for. 

I then reversed the nesting so that we queried the join table first, but we were still having the same problem with finding the associated ingredient, until we realized that we were not looking for multiple Ingredients but instead a single ingredient. Therefore, the final nesting of the forms was as follows:

```
form_for [@user, @recipe] do |f|
  f.fields_for :quantities do |q|
   q.fields_for :ingredient |i|
   end
  end
end
```

I also had to move my `accepts_nested_attributes_for` macro from my Ingredient model to my Quantity model. Once we had this figured out, I was able to revise my custom setter method to deal with the new params, which ended up as follows:

```
  def quantities_attributes=(quantities_attributes)
    self.quantities.destroy_all
    quantities_attributes.each do |key, value|
      if value["ingredient_attributes"]["name"] != ""
        ingredient = Ingredient.find_or_create_by(name: value["ingredient_attributes"]["name"])
        self.quantities.build(ingredient: ingredient, amount: value["amount"])
      end
    end
  end
```

## Authentication and OmniAuth

Initially, I had set up Users to have a username for login and authentication with a password. I was satisfied with the way users created their accounts and logged in. However, one of the requirements for the project was to provide third-party authentication with OmniAuth, which I chose to do using Github. As I was reflecting on how to pull in the Github data, I realized that if I used the Github nickname (or username) field as a proxy for my User’s username, I would potentially have overlapping usernames in my database, which would not be good. A Github user with the same username as someone already in my database would be able to see all of that person’s recipes and post as if they were the same person. Therefore, I went back and changed my User model to use email instead of username as their login. This resulted in me changing more than I had anticipated throughout my app. I did not want individual users to be able to see other users’ email addresses, so I eliminated my Users index page, and made it so a user could only see their own recipes index page. 

In order to actually set up the third-party authentication, I added the `omniauth` and `omniauth-github `gems to my Gemfile. I also added `dotenv-rails` to my Gemfile in order to set up a `.env` file in my root directory to store my Github key and secret, and added that to my `.gitignore` file so that it would not be shared on Github. I added an `omniauth.rb` file to `config/initializers` that had the content: 

```
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET']
end
```

Next, I needed to add a route to my `config/routes.rb` file that mapped `get ‘/auth/github/callback'` to the create method in my Sessions controller. From there, I followed Avi Flombaum’s “TodoMVC- Implementing Omniauth’ video to set up my create route to deal with this new OmniAuth data and create the method `User.find_or_create_by_omniauth(auth_hash)` in my User model. 


## Styling my App

Finally, after I had worked through all the logistical issues with my app, I wanted to add some styling. First, I added Bootstrap using the `bootstrap-sass` gem (with instructions from https://stackoverflow.com/questions/42597050/what-is-the-best-way-to-install-bootstrap-with-rails-app). I wrapped my contents in divs with class “container” and gave my buttons some simple styling. I also styled my flash alerts and my validation error explanations with basic Bootstrap stylings. Finally, I added a nav bar that would be shown to logged in users that had links to All Recipes, Your Recipes, Recipes by Ingredient, New Recipe, and Logout. 

Initially, I had the Logout as a form that was routed to `post /logout`. However, the form and button would not fit in the nav bar, so I changed the route to `get /logout` and re-styled the logout as a link instead. I know that destroy requests should generally be sent as post requests, but I made this choice for the aesthetics of the site. 

Finally, I wanted to give my table a bit more styling, since it was the primary way Users interacted with the Recipes. I used CSS to give the table a border and some padding, and I added a hover effect on the table rows so they highlighted as a User hovered over them. Because I have not yet studied the Asset Pipeline, I did not know how to integrate CSS. Fortunately, Luisa Scavo helped out once again by recommending adding a `styles.css` file in the `Public` folder and providing a link to the stylesheet in my application layout file. 

## Reflection

Overall, I am happy with how my application turned out. I feel the functionality of the app provides for a smooth and easy user experience, and I am happy with the way the app looks in the end. I have really learned a lot more about Rails by building an application from scratch, and I’m starting to feel like a real developer after all the issues I faced and the problem-solving that came with them involving a lot of Googling and collaboration. 

If I were to recommend any further refinements to the application, I would like to see the ability to add comments and reviews to the recipes. In exploring this, using email addresses as User identifiers makes that difficult, for the privacy reasons explored above. 

Another issue I cannot seem to resolve is that my CSS styling of my table seems to disappear after creating or updating a recipe, but reappears when I refresh the page. I suspect that this is due to my style sheet not being part of the Asset Pipeline, but am not sure how to fix that just yet.

However, I am proud of the work I have done as it stands. If you’d like to view my source code for the project, it is available at https://github.com/anikonchuk/quick-and-easy-recipes.
