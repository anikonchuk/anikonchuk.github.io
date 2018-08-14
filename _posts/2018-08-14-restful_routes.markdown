---
layout: post
title:      "RESTful Routes"
date:       2018-08-14 21:07:54 +0000
permalink:  restful_routes
---


As I dive into learning about Sinatra, one of the major concepts is that of MVC architecture: using Models, Views, and Controllers. Part of this is using controllers to assign routes. A concept I found very helpful while trying to put this information together was that of RESTful routes. RESTful routes are conventions that match routes we define to CRUD (create, read, update. and delete) methods. It stands for Representation State Transfer, and was created by computer scientist Roy Fielding for his doctoral thesis, Architectural Styles and the Design of Network-based Software Architectures. 

#### Why are RESTful routes important?
One of the more difficult concepts for me to process while studying the MVC architecture is how to name my routes and where to direct forms to post. RESTful routes provides a framework for naming these routes that clarifies data flow in applications both small and large. This convention is important for collaboration and open-source projects, so outsiders can understand what is happening in our controllers. In addition, it helps keep a project organized as it scales up from a simple application to a complex one. Using RESTful routes can help developers expose their data to the world through API’s. The consistency provided by RESTful routes is important for both users and developers. 


#### What are the RESTful routes?
As shown in the table below, there are seven standard paths we can use for our controllers, lining up with CRUD methods. The table uses an example of a Song model.

![](https://i.imgur.com/sVvQoko.png?1)

The RESTful actions can be grouped according to CRUD methods, with many of them pairing up to perform actions in tandem. The “index” and “show” actions line up with the “READ” CRUD method. “New” and “create” work together to “CREATE” a new instance, while “edit” and “update” collaborate to “UPDATE” the instance. Finally “destroy” does just what it implies… it “DELETES” an instance.

One thing to be careful of in your controller is the relative placement of these routes. If you place the dynamic route for show (get ‘/artists/:id’) before the route for new (get ‘/artists/new’), when the user tries to navigate to the ‘new’ route, the controller will try to look up an object with the id of new instead of navigating directly to the ‘new’ route.

#### Summary
RESTful routes take the guesswork out of how to name your routes in the MVC architecture. This simple convention allows us to create clear paths for data in our applications that can be understood both by the programmer and anyone with whom the application is shared. By using RESTful routes, we are clearly communicating our architecture and data with potential collaborators and with the world. 

