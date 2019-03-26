---
layout: post
title:      "What is JSX, Anyway?"
date:       2019-03-26 14:12:59 +0000
permalink:  what_is_jsx_anyway
---


As I was working on my React project for Flatiron School (more about that [here](http://andrewnikonchuk.com/fleddit-_a_react_redux_application_with_a_rails_back_end)), I became interested in the JSX language that was being used. How could I use both HTML and JavaScript seemingly in the same expression? This blog post will explore the basics of JSX and show that while using JSX isn’t necessary, it sure does make using React a lot more convenient!

## JSX, The Basics

JSX is a domain-specific language (DSL) used in React to simplify writing elements that will be added to the DOM. It looks a lot like regular HTML with JavaScript mixed in. It is a “syntax extension to JavaScript” according to the [React docs](https://reactjs.org/docs/introducing-jsx.html). React must be in scope where you’re writing JSX code, so at the top of your file, always `import React from ‘react’`. The Babel processor turns JSX into normal JavaScript written with the React library (see “Syntactic Sugar” below for more details). 

To write JSX code, you basically just write HTML markup as you normally would, but then you can insert JavaScript expressions using curly braces ( `{}` ). Let’s look at a sample inside a class-based React component:

```
const user = {
	firstName: “Joe”,
	lastName: “Smith”
}

function formatName(firstName, lastName) {
	return `${firstName} {lastName}`
}

render() {
	return (
		<h2 className=“welcome”>Hello, {user.firstName} {user.lastName}</h2>
		<p>Welcome to the store {formatName(user.firstName, user.lastName)}<p>
	)
}
```

This will render “Hello, Joe Smith” as an `h2` header to the DOM, followed by “Welcome to the store Joe Smith” wrapped in a regular `p` tag. Notice that we were able to integrate the JavaScript expression by simply wrapping it in curly braces. You can integrate any sort of JavaScript expression into these curly braces, from a function expression to a ternary conditional statement. Also note that we used `className` instead of just `class`… more on that in “Caveats” below. 

## Syntactic Sugar

There is nothing magical about JSX all by itself. It is just syntactic sugar for React’s built in `createElement(component, props, …children)` function. The `createElement` method takes three arguments: the element’s name, an object representing the element’s properties, and an array of the element’s children. JSX is not a requirement for using React. It just makes it more convenient. For example, let’s take a sample below:

```
<h1 id=“storeName”>Joe’s Pie Shop</h1>

<ul className=“list">
	<li>Cherry Pie</li>
	<li>Apple Pie</li>
	<li>Peach Pie</li>
</ul>
```

This may look like simple HTML, but because it is written in JSX, the Babel preprocessor will go through it and convert it to JavaScript written for React by changing it to `React.createElement()`, turning its properties into a props object, and embedding its children inside the function. The example would look like the following:

```
React.createElement(
	‘h1’, 
	{id: ‘storeName’},
	‘Joe’s Pie Shop’
)

React.createElement(
	‘ul’,
	{className: ‘list’},
	React.createElement(
		‘li’, 
		null, 
		‘Cherry Pie’
	),
	React.createElement(
		‘li’, 
		null, 
		‘Apple Pie’
	),
	React.createElement(
		‘li’, 
		null, 
		‘Peach Pie’
	)
)
```

As you can see, you can absolutely write out all of your components using just `React.createElement()`, but it’s a lot more difficult to read and interpret than writing JSX! JSX preserves the tree structure of HTML and lets you write in the markup language, integrating JavaScript where you see fit.

## Caveats

While writing JSX is fairly straightforward, there are a few things to watch out for…
* Because `class` and `for` are reserved words in JavaScript, you need to use `className` and `htmlFor` respectively.
* You have to close all tags, even self-closing tags when using JSX. For example, if you had `<img src=“#”>`, you would need to close it like `<img src=“#” />`.

## Conclusion

While you can write React components without JSX, it doesn’t really make sense to do it that way. JSX takes cumbersome JavaScript and transforms it into readable and easily manipulated HTML-like statements. 
