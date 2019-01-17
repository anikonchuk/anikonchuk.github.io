---
layout: post
title:      "Fleddit- A React/Redux Application with a Rails Back End"
date:       2019-01-17 10:23:40 -0500
permalink:  fleddit-_a_react_redux_application_with_a_rails_back_end
---


My fifth and final project for Flatiron School’s Online Full Stack Web Development Program involved creating an application with a React and Redux front end and a Rails API back end. I created Fleddit, a blog application that allows users to share their ideas and comment on them. The requirements for this project were a bit more open-ended than previous projects. We had to create a React app with a  certain number of container and stateless components and routes, and we had to make use of async actions that sent and received data from a Rails API. Beyond that, we were free to design whatever we wanted.

## Planning the Architecture of the Application
My first major hurdle was designing the overall architecture of the application itself. I knew that I wanted the following major routes:

* “`/`“- This home page consists of a Welcome component that welcomed the user to the website and gave some context as to what to do there. 
* “`/posts`”- This posts index page displays the `PostsIndexContainer` component, which iterates through the posts and displays a card for each of them (`PostPreview` component) with just the title of the post and the author. The user can then click on the title and be taken to the post show page (see below). The `PostsIndexContainer` also contains my `PostForm` component, into which the user can enter information and submit a new post. 
* “`/posts/:postId`”- The post show page has a `PostsShowContainer` as the parent component to the `PostCard` component, which displays the full post, including title, author, content, and optional image. The `PostsShowContainer` also contains the `CommentsContainer`. The `CommentsContainer` filters through the comments and displays `CommentCard` components for each of the comments associated with the parent component’s post. It also contains a `CommentForm` component that the user can use to create a new comment. The `PostsShowContainer` passes down a prop of the target post’s id to the `CommentsContainer` so that only the appropriate comments are displayed, and so that the new comment created is associated with the appropriate post.
* “`/about`”- This simple `About` component gives some context for the project, like the purpose it was created for and the technology used to create it. 

Once I had the architecture of the site down, I was able to move on to creating my Rails back end.


## Setting Up My Rails API
My first priority with the project after the planning phase was setting up my Rails API to appropriately handle JSON data coming in and respond with the correct JSON data. I created my models (Posts and Comments), routes, and controllers. I knew I wanted my routes namespaced under “api” so I set up my routes as the following: 

```
  namespace :api do
    resources :posts, only: [:show, :index, :create]
    resources :comments, only: [:index, :create]  
  end
```

I only wanted the front end client to be able to do specific things, so I limited my routes to show, index, and create for posts and index and create for comments. 

I then used `ActiveModel Serializers` to customize the specific information being sent through the API. For example, in the current iteration of my application, I have no need for the user to see the timestamps for posts and comments, so I did not list them under the attributes in my serializer. I had initially considered adding the `belongs_to/has_many` relationship to the serializer to send the associated posts and comments together through JSON, but upon reflection, decided against it. There was no need to load all the comments for posts when loading the initial post index page. I made the decision to send them separately, and then just filter through the comments on the front end when displaying comments for an individual post. Once I had my back end set up, it was on to the React/Redux front end!

## Controlled Forms and Submitting Data
I used controlled forms for all of my data input in this project. What this means is that the value of the form input is linked to the state of the component, and every time a change was made to the input, it is reflected in the state. This made submission of data much more straightforward. When the user hit submit on the form, I was able to send the entire state by dispatching an action that took in the state as an argument. The action then fired off a “`POST`” fetch request to my API and packaged the data as JSON by using `JSON.stringify`. One thing I had to be careful of though was the naming of the data. Because I am using strong parameters in my Rails API, I had to namespace the data. For example, the following code needed to be used as my fetch request: 

```
fetch('http://localhost:3005/api/posts', {
      method: "POST",
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ post: post })
    })
```

I needed to tell the server that I was sending json data in the headers, and when I stringified my data, I needed to add `{post: post}` as the argument (with the second “post” being the data passed in from my component’s state). 

`Routing to My Post Show Page`
A major challenge I faced while building this app was figuring out how to route to my Posts Show page, where only a single post was displayed. I knew I wanted to follow RESTful routing and have the app route to `/posts/:postId`. Fortunately, `react-router-dom` was able to handle this readily. I created a route in my `App.js` file to `<Route exact path={`/posts/:postId`} component={PostsShowContainer}/>`, which would display my `PostsShowContainer`. It would also pass to `PostsShowContainer` in the props a ‘match’ property. Within that match property, I was able to pull out the postId by calling `this.props.match.params.postId`, which I could then pass to an action to fetch a particular post to display. Problem solved!

## Styling the Site
I knew that I wanted my website to have some styling beyond what was present in the initial `create-react-app` package. My first step was to integrate Bootstrap into the project using the `react-bootstrap` npm package. After running `npm install`, I added a link to the Bootstrap CDN to my `index.html` page for the stylesheets. This allowed me to take advantage of all of Bootstrap’s built-in classes for styling things like buttons. I was also able to use Bootstrap’s grid system to layout my posts index page to have the PostPreview cards display on the left two-thirds of the screen and the new post form on the right third of the screen. I also used regular CSS to customize how my `PostPreview` cards and post show page displayed. I added box-shadow effects to create a “card”-like look for each of the Preview cards and the Post Show component. After finishing my app, I realized that I did not even use any of the components within the `react-bootstrap` package. I was only using what was provided by the CDN for Bootstrap, so I uninstalled the package by removing it from my `package.json` and running `npm install` again.

## Navigation Bar
Another issue I had as I was developing my app was getting my Navigation Bar to display correctly. I tried following several tutorials about using `react-bootstrap` to create a responsive navigation bar, but none of them seemed to work with my app. I ended up using `react-router-dom`’s built-in `NavLink` feature to create links and style them. I was able to create links to the different routes in my app that had individual styling within the link itself. Further, by adding an “`exact`” property, when the user navigated to that route, the link changed to an Active state with a different background color.

## Solving the “Ghost Post” Issue
As I was navigating through my site, I noticed that when I went to a post’s show page after visiting a different post’s show page, I could still see the image from the previous post as the new post was loading.  I didn’t want that ‘ghost’ of a post from before appearing. Instead, I wanted the user to see “Loading Post” while it was loading. I realized that the “target post” that was being displayed was still stored in the Redux store and displaying the old post while it was loading. Therefore, I needed to create an action and a reducer case that would clear the target post from the store when I left the page. I used the component lifecycle event `componentWillUnmount` to accomplish this. As the component prepares to unmount, an action is dispatched to the reducer that resets the target post property in the store to its initial empty state. This cleared the target post out of the store and allowed me to see the “Loading” message I was looking for.

## Reflection and Going Forward
Overall, I feel much more confident working with React and Redux after completing this project. I was able to effectively pass information between React components and into and out of the Redux store. Once I had the architecture of the site planned, it was fairly straightforward which components were needed and how they would connect together. 

There are still a number of improvements to be made to the application going forward. I had initially wanted to have a User class and authenticate users and authorize them to do certain things. That was beyond the scope of this project, so I put it on hold, but eventually, I would like to incorporate that functionality to secure the site. This would also allow me to build out full CRUD functionality. I also want to add an Upvote feature, like Reddit. I plan on adding both of these features going forward. 

If you would like to view my source code for the project, it is available on Github. The repository for the Rails API is at https://github.com/anikonchuk/fleddit-api, and the repository for the React/Redux client side is at https://github.com/anikonchuk/fleddit-client.
