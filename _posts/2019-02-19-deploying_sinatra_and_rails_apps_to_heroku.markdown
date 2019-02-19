---
layout: post
title:      "Deploying Sinatra and Rails Apps to Heroku"
date:       2019-02-19 20:56:19 +0000
permalink:  deploying_sinatra_and_rails_apps_to_heroku
---


As a student, I (somewhat foolishly) put off deploying my portfolio projects. Now that I have graduated, I want to make sure that the apps that I have built are accessible to potential employers and the world at large. My project this week, therefore, was to deploy my Sinatra, Rails, and Rails with JavaScript apps to Heroku. This blog post will detail the process I used, as well as some setbacks I encountered, with the solutions I put in place.

## Heroku and Databases

Heroku (www.heroku.com) is a cloud-hosting service that allows a certain number of free apps (five for unverified accounts) to be hosted on its service. Deployment on Heroku is quick and easy, with a number of guides to walk you through the process (https://devcenter.heroku.com/articles/rack for Sinatra and Rack-based apps, and https://devcenter.heroku.com/articles/getting-started-with-rails5 for Rails 5.x-based apps). 

The primary sticking-point for deployment for most projects is the database used. Sinatra projects bootstrapped by the corneal gem and Rails projects by default use SQLite for their databases. Heroku, on the other hand, uses PostgreSQL for its databases. According to [Heroku](https://devcenter.heroku.com/articles/sqlite3), SQLite doesn’t work well with the service because it “runs in memory, and backs up its data store in files on disk.” Meanwhile, Heroku has an ephemeral filesystem that allows you to write and read from it, but clears periodically. This means that if you use SQLite on Heroku, you would have your database cleared frequently. There is also an issue with the way dynos work on Heroku. Each dyno would need its own copy of the database file, and if there were more than one dyno running your app, those copies of the data would not be synchronized. Due to these limitations, the main adjustment to my apps that had to be made before deployment was changing over from SQLite to Postgres.

## Deploying My Sinatra App

The following instructions come primarily from one of Enoch Griffith’s study groups for the Flatiron School. They do not align entirely with the instructions from Heroku itself for [Sinatra projects](https://devcenter.heroku.com/articles/rack), which I think is primarily because my Sinatra project was bootstrapped using the [corneal gem](https://github.com/thebrianemory/corneal), which provides a fully functioning Sinatra app right out of the box with configurations already in place that do not align with the instructions. I have also noted where I had to make additional adjustments based on my experience with the process.

### Adjusting the Gemfile

The first thing I needed to do was adjust the `Gemfile` of the project. I removed the gems for ‘`pry`’ and ‘`sqlite3`’ and added the following code: 

```
group :production do
	gem ‘pg’
end

group :development, :test do
	gem ‘pry’
	gem ‘sqlite3’
end
```

After deleting the `Gemfile.lock`, I ran `bundle install` to set up the new configuration.

### Modifying the Database Setup

To start with a clean slate, I manually deleted the `development.sqlite` database and the `schema.rb` file. I then made sure my migrations were all versioned by adding [4.2] to the class name as follows: 

`class CreateUsers < ActiveRecord::Migration[4.2]`

I then created a `database.yml` file in my `config` folder and added the following code:

```
development:
	adapter: postgresql
	encoding: unicode
	database: development
	host: localhost
	pool: 2
production: 
	adapter: postgresql
	encoding: unicode
	pool: 5
	host: <%= ENV[‘DATABASE_HOST’] %>
	database: production
```

This is the first place I had to adjust Enoch’s instructions. When I tried to run my `rake` db tasks locally in my development environment, I was getting an error that Postgres could not connect. By following the instructions in this [Stack Overflow post](https://stackoverflow.com/questions/14225038/postgresql-adapter-pg-could-not-connect-to-server), I added “`host: localhost`” to my development configuration to allow for Postgres to create the database locally.

I then had to remove ActiveRecord’s connection with SQLite by removing the following lines from my `environment.rb` file: 

```
ActiveRecord::Base.establish_connection(
	:adapter => “sqlite3”,
	:database => “db/#{ENV[;SINATRA_ENV’]}.sqlite”
)
```

### Housekeeping and Setting Up New Databases

One final set of lines had to be removed. In my `config.ru` file, I removed the following, because Heroku did not recognize the command: 

```
if ActiveRecord::Migrator.needs_migration?
  raise 'Migrations are pending. Run `rake db:migrate` to resolve the issue.'
end
```

Then, I created my databases locally by running `rake db:create` and `rake db:migrate` (make sure you have a Postgres server running locally on your machine). 

### Setting Up Heroku

After setting up my Heroku account, I created a new app and selected “`Connect to GitHub`”. I then followed the on-screen instructions and chose the branch I wanted to connect and hit deploy. The on-screen instructions were easy to follow!

Once it successfully built my app in Heroku, I used the drop-down menu at the top to select “`Console`” and ran `rake db:migrate` to create my database structure in the production server. Once I had all these steps completed, my app was live and ready to go! You can see the final project at https://mycomicsbox.herokuapp.com/!

## Deploying My Rails App

Once I had my Sinatra app deployed, the Rails app went much more smoothly. I followed Seth Alexander’s study group from the learn.co Instruct site, which pretty closely mirrored the [Heroku docs for Rails deployment](https://devcenter.heroku.com/articles/getting-started-with-rails5).

### Modifying the Gemfile

The first task in integrating Postgres into my Rails app was to modify the `Gemfile`. I removed the ‘`sqlite3`’ gem and replaced it with the ‘`pg`’ Postgres gem. One problem that came up later on when I was deploying was that Heroku only supports [certain versions of Ruby](https://devcenter.heroku.com/articles/ruby-support#supported-runtimes). Therefore, at the top of the Gemfile, where the Ruby version is specified, I had to change it to `ruby ‘2.5.3’`. I then ran `bundle install` to update the remainder of the gems to work with the newer version of Ruby. 

### Configuring the Database

My next step in the process took me to the `config/database.yml` file. I began by copying the file structure given by the Heroku guide, which basically replaced all of what was initially there with the following: 

```
default: &default
	adapter: postgresql
	encoding: unicode
	pool: <%= ENV.fetch(“RAILS_MAX_THREADS”) { 5 } %>

development: 
	<<: *default
	host: localhost
	database: myapp_development

test:
	<<: *default
	host: localhost
	database: myapp_test

production:
	<<: *default
	database: myapp_production
	username: myapp
	password: <%= ENV[‘MYAPP_DATABASE_PASSWORD’] %>
```

Once again, I had to add `host: localhost` to the test and development configurations to allow me to run a database locally. I also had to change the “`myapp`” in the database names to the name of the individual app so there was no name overlap on my local Postgres server. I then ran `rake db:create` and `rake db:migrate` on my local computer to create the database and schema for the project.

### Ugly Uglifier Error

When I was trying to deploy the my JavaScript app to Heroku, I ran into a problem with my `Uglifier`. When it was compiling the build I received the following error:

`Uglifier::Error: Unexpected token: name (Ingredient). To use ES6 syntax, harmony mode must be enabled with Uglifier.new(:harmony => true).`

I found an answer on a GitHub repository with the same issue (https://github.com/lautis/uglifier/issues/127). In my `config/environments/production.rb` file, I had to change my JavaScript compressor configuration to the following: 

`config.assets.js_compressor = Uglifier.new(harmony:true)`

### Deploying to Heroku and Setting Up Environment Variables

The deployment process to Heroku is identical to the one described above for the Sinatra app. I created a new project, linked to GitHub, and deployed the branch. I then also opened up the dropdown for the `Console` and ran `rake db:migrate` to establish the database configuration on the Heroku app.

The final step was to fix the configuration of my OAuth with Github. On the GitHub developers site, I updated the website addresses I had been using from the `localhost:3000` address to my fancy new Heroku address. But I still had the problem of no longer having a `.env` file synced to the project. Fortunately, Heroku has a solution for that as well. On the app setup page, under “`Settings`”, you can add “`Config Vars`”. Here, I added my `GITHUB_KEY` and `GITHUB_SECRET` from my `.env` file. My OAuth was now functional on my new projects, and they were fully deployed! You can see my Rails project at https://quickandeasyrecipes.herokuapp.com/ and my Rails with JavaScript project at https://quickeasyrecipes-js.herokuapp.com!

## Conclusion

Overall, the main hurdle to jump when deploying a Sinatra or Rails project to Heroku is ensuring that you have Postgres as your database instead of SQLite. Once you figure out how to transition over to the new database, deploying is as simple as following on-screen instructions clearly provided by Heroku. I learned a lot about troubleshooting while deploying these projects, especially regarding Postgres database issues. Hopefully, this blog walked you through some of the friction points in that process and provided some solutions to some of the roadblocks you might encounter.
