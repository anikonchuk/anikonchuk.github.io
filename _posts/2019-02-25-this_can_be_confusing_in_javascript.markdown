---
layout: post
title:      "'This' Can Be Confusing in JavaScript!"
date:       2019-02-25 15:31:55 -0500
permalink:  this_can_be_confusing_in_javascript
---


## What is “this”?

In JavaScript, “this” is a keyword that acts as a pronoun would in the English language. It is a shortcut to refer to an object. (A quick note… in the English language, “this” could also behave as an adjective, like when you refer to “this dog”, or an adverb, like when you can’t believe the meeting starts “this early”, but for our purposes in JavaScript, “this” will behave most closely to a pronoun, replacing a “noun”, or an object.) In JavaScript, “this” will always refer to an object. It will take up the properties of that object as well. 


## Different contexts yield different references

The value of “this” in JavaScript depends on the context in which it is called. Just like a pronoun in the English language needs to refer to a noun for it to make sense, “this” must have some sort of reference point. The lexical scope of where “this” is defined provides that context. 

### Outside of a function

Outside of a function or an object, “this” will refer to the global object. In web browsers, the global object is the window. According to [Mozilla’s MDN docs](https://developer.mozilla.org/en-US/docs/Web/API/Window), the Window “represents a window containing a DOM document; the document property points to the DOM document loaded in that window” Therefore, in the console of your browser…

```
console.log(this)
// Window
```

### Inside a standalone function

Inside a standalone function, “this” will refer to the global object once again. So if you have the following function and execute it, you will once again console.log “Window”.

```
function foo() {
	console.log(this)
}

foo()
// Window
```

However, if you are in strict mode (more about strict mode in [MDN’s documents here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)), “this” will be undefined. 

### Inside an object method

An object method in JavaScript is a property of an object that points to a function. Inside an object method, “this” will refer to the object on which the method was called. For example:

```
let dog = {
	name: “Katie”,
	breed: “Golden Retriever”,
	speak: function () {
		console.log(`${this.name} says Woof!`)
	}
}

dog.speak()
//Katie says Woof!
```

In this case, when you call `dog.speak()`, the “this” inside the console.log refers to the dog object on which it is called, whose name property is “Katie”.

### Changing the context of an object method

Let’s take our code above for the dog object and modify it slightly to see what happens when we remove some of the context.

```
let newSpeakFunction = dog.speak
// this copies the function defined in dog.speak into a new function

newSpeakFunction()
// says Woof!
```

What happened here? Let’s take a closer look. When we copied over the function into the new variable `newSpeakFunction`, we removed it from its original context. The `newSpeakFunction` has no concept of the `dog` object. Instead, “this” will refer to the global object, or the window. Here, there is no defined “`Window.name`”, so `this.name` was undefined in this context.

### Functions inside functions

Let’s take a look at what happens when we try to invoke functions inside other functions. Remember, we already have our dog object defined, so we are just going to add another property to it using dot notation.

```
dog.heel = function() {
	function otherHeelFunction() {
		console.log(this)
	}
	return otherHeelFunction();
}
```

Let’s think about what happens when we invoke `dog.heel()`. First, it will define `otherHeelFunction`, which console.logs “this”. Then, it executes `otherHeelFunction`. What do you think will be console.logged? What will the value of “this” be?

```
dog.heel()
//Window
```

That’s surprising! Instead of referring to the object dog, in this case, “this” refers to the global object, or the window. Here, “this” is not referenced directly inside an object method. Instead it’s referenced by a function inside a method. `otherHeelFunction` is just a function, not a method itself. 

When you have a function inside of another function, as also happens with callbacks and closures, “this” will refer to the global context, or the window in a web browser. 

### Arrow functions

Introduced in ES6, arrow functions provide a more concise way of writing functions. However, they also introduce a new wrinkle to the complexity of context for “this” references. Arrow functions do not have their own “this.” Instead, they use the scope of whatever they are defined in to adopt that “this.” Let’s go back to our dog example and add another object method using dot notation:

```
dog.fetchMethod = function () {
	const newFetchArrowFunction = () => {
		console.log(`Go get the ball, ${this.name}`)
	}
	newFetchArrowFunction()
}
```

This looks pretty similar to our `heel` object method from the last example. Let’s see what happens when we call the function:

```
dog.fetchMethod()
// Go get the ball, Katie
```

Here, because the arrow function does not define its own context for “this,” the function adopts the context in which it was defined. In this case, that’s the fetch object method `fetchMethod`, which has a context that defines “this” as the `dog` object.

### Call, apply, and bind

You can use `call()`, `apply()`, and `bind()` to define the context for “this” in a function, even if it is a standalone function. 

Let’s take a look at using call and apply first. We are going to create a new object and function, so let’s forget about Katie the dog for now.

```
let cat = {
	name: “Whiskers”,
	age: 8
}

function summon () {
	console.log(`Come here, ${this.name}!`)
}
```

Let’s think about what happens when we call `summon()` without `call` or `apply`… Because `summon()` is a standalone function and not an object method, the context for “this” will default to the global context, or the window in a web browser. Because the window has no name property, it will be undefined. However, we can alter the context of `summon()` using `call()` and `apply()` as follows:

```
summon()
// Come here, !

summon.call(cat)
// Come here, Whiskers!

summon.apply(cat)
// Come here, Whiskers!
```

With `call()` and `apply()`, the function is now bound to the object `cat`, and this.name now refers to “Whiskers.” The only difference between `call` and `apply` is passing in arguments. If the function has arguments that need to be passed in, first you pass in the context for “this”, then for `call()`, you pass in all additional arguments separated by commas. For `apply()`, you pass in additional arguments in an array. Looking at our cat example again:

```
function feedCat (amount, typeOfFood) {
	console.log(`Time to feed ${amount} of ${typeOfFood} to ${this.name}.`)
}

feedCat.call(cat, “1 cup”, “dry food”)
// Time to feed 1 cup of dry food to Whiskers.

feedCat.apply(cat, [“1 cup”, “dry food”])
// Time to feed 1 cup of dry food to Whiskers.
```

`call()` and `apply()` both set the context of the function and then immediately invoke it. If you want to define the context of “this” for a function to be used later, you can use `bind()`. Let’s take a look, using our `cat` object again:

```
function groom() {
console.log(`Time to groom ${this.name}!`)
}

let newGroom = groom.bind(cat)

newGroom()
// Time to groom Whiskers!
```

Our new function is now bound to the context of the object `cat`. So now we’ve seen how we can use `bind()`, `call()`, and `apply()` to define contexts for our functions.

## Summary

As I have shown throughout this post, “this” can have many different meanings depending on its context. To summarize (with heavy thanks to the learn.co curriculum, on which this summary is based):
* Outside of any function, “this” refers to the global object, which, in the case of a web browser, is the window.
* Inside an object method, “this” will refer to the object that is receiving the method call.
* In a function that is not tied to an object directly (including a function inside a function), “this” will refer to the global object.
* If using “strict mode”, “this” will be undefined.
* Arrow functions do not define their own context for this. Instead, “this” will refer to whatever context is defined by the parent context. 
* You can use `bind()`, `call()`, and `apply()` to set the context of the "this" keyword.
