---
layout: post
title:      "Thinking About Thunk: Asynchronous Actions in Redux"
date:       2019-02-11 16:19:00 +0000
permalink:  thinking_about_thunk_asynchronous_actions_in_redux
---


As I was working on my React project (more about that [here](http://andrewnikonchuk.com/fleddit-_a_react_redux_application_with_a_rails_back_end )), I was a bit confused by how asynchronous requests worked in the React/Redux workflow. So I decided to do a bit of research into the topic and dove head first into Redux Thunk. 

## What is a Thunk?
According to [Dave Ceddia](https://daveceddia.com/what-is-a-thunk/), a ‘thunk’ in general is just a function that’s returned by another function. In terms of Redux, Thunk, in its most basic form, is a piece of Redux middleware that allows you to use an action creator to return another function and call that function.

## The Problem Redux Thunk Solves
Redux has a well-defined course of events to manage state in an application. Action creators create actions (just plain old JavaScript objects that contain a type and sometimes a payload). These actions are then dispatched to the a reducer, which then updates the store with a new state. These changes are then sent back to components that subscribe to the store, and if these components are affected by the change they are re-rendered. 

The problem arises when we try to integrate asynchronous calls into our Redux flow. Let’s say I tried to program an action creator called `updateDogs()` to fetch a list of dogs from an API as follows (thank you to the learn.co curriculum for the example idea):

```
function updateDogs() {
	let dogs = fetch(‘http://www.sampleAPIforDogs.com')
 	return {
		type: ‘UPDATE_DOGS’
		dogs
	};
};
```

If we try to simply add a `fetch` request before returning our action in the action creator, we won’t have a payload because the `fetch` request is asynchronous and the return will run first. The `fetch` request returns a Promise, which is an object that represents that data will be returned later. The Promise can have values of pending, fulfilled (resolved), or rejected (there was an error). Once the Promise resolves, we can chain a `.then()` function onto our `fetch` call to capture that data and use it. However, even if we were to chain a `.then()` to the `fetch`, we still have the problem of the action creator returning its action before the Promise resolves. 

## Thunk to the Rescue
This is where Redux Thunk comes in to play. Thunk allows us to call action creators that return a function instead of just an action object. This allows us to take in the store’s dispatch method as an argument, which can then be used to dispatch multiple actions from inside that returned function. This returned function does not need to be pure, so it can have side effects like making API calls. The returned function can also take in the method getState, which the [Redux docs](https://redux.js.org/advanced/async-actions) indicate is “useful for avoiding a network request if a cached value is already available”, but that is beyond the scope of our discussion here.

So Redux Thunk allows us to first dispatch an immediate action to the store to say that we’ve started our loading process with an external API. You want to start with an immediate action to the store because your asynchronous action won’t happen right away, regardless of how quickly your API responds. Then we can make our actual fetch request, followed by dispatching another action with the payload and a success message, as seen below:

```
function updateDogs() {
	return (dispatch) => {
		dispatch({ type: ‘LOADING_DOGS’ });
		return fetch(‘http://www.sampleAPIforDogs.com')
			.then(response => response.json())
			.then(dogs => dispatch({ type: ‘UPDATE_DOGS’, dogs }));
	};
};
```

## Setting Up Redux Thunk
Setting up Redux Thunk is fairly straightforward. First, from the terminal in the root directory of the project, run the following:

`npm install --save redux-thunk`

This will add Redux Thunk to the package.json. Then, in the index.js file, be sure to import applyMiddleware from redux and thunk from redux-thunk. Then, when creating your store, you can integrate thunk. Assuming you are using a rootReducer, your code should look like the following:

```
import { createStore, applyMiddleware } from ‘redux’;
import thunk from ‘redux-thunk’;
import rootReducer from ‘../reducers’;

const store = createStore(rootReducer, applyMiddleware(thunk));
```

From there, you can use Redux Thunk’s powerful abilities in your action creators to handle asynchronous actions!

## Conclusion
With only 14 lines of code (see the whole shebang [here](https://github.com/reduxjs/redux-thunk/blob/master/src/index.js)), Redux Thunk allows us to execute asynchronous actions within the context of Redux’s workflow. It allows our action creators to return functions instead of simple JavaScript objects that can dispatch multiple actions from within one action creator. For more information on Thunk, see the Github for the middleware [here](https://github.com/reduxjs/redux-thunk), and check out the Redux docs’ article on asynchronous actions [here](https://redux.js.org/advanced/async-actions).
