---
layout: post
title:      "Sorting an Object in JavaScript"
date:       2019-03-18 14:10:20 +0000
permalink:  sorting_an_object_in_javascript
---


Objects are a great way to store data in JavaScript. They are comparable to objects in real life. According to [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects), an object is “a collection of properties, and a property is an association between a name (or key) and a value.” Unlike an array (which is just a type of object), in standard objects, there is no native way to sort these properties. This past week, for a technical interview, part of the challenge involved sorting the properties of an object. While I won’t get into the specifics of the actual challenge, I thought it would make for an interesting blog post to explain how I went about sorting an object.

## The Problem

Let’s say we are given an object that represents the prices of a set of groceries, and we want to return an array of the items listed from most expensive to least expensive. For example, the object looks like the following: 

```
let groceries = {
	peanutButter: 2.49,
	cleaningWipes: 4.99,
	cannedPeas: 0.67,
	cereal: 2.99,
	milk: 2.19
}
```

## Planning a Strategy 

As I said earlier, there is no straightforward way to return a sorted array based on the data structure as it stands. We need to find a way to extract the values from the keys into a data structure that we can sort, while still maintaining the relationship with their keys. We then need to find a way to return just the keys and not their associated values. So overall, the strategy would be as follows:
1. Iterate through the object and extract the data into an array format
1. Sort the new array
1. Iterate through the array to create a new array with just the grocery items’ names

## Iterating Through the Object

The first step in the process is iterating through the object to extract the keys and values into an array. Starting with ECMAScript 5, we can now use a “`for… in`” loop to traverse an object’s properties, making the process fairly straightforward. We will be pushing an array of the item’s key and its price into a new array, keeping those values tied together.

```
let groceriesToSort = []

for (const item in groceries) {
	groceriesToSort.push([item, groceries[item]]);
}
```

We need to use the bracket notation to access our item inside the iteration because if we used the dot notation (`groceries.item`), the JavaScript interpreter would look for a key that is called “item”, and return undefined for each item. With bracket notation, the computed member access operator will interpret “item” as the iterated key and use that to access the value associated with that key.

Our new groceriesToSort array after this operation should look like the following (note that regardless of data type, the JavaScript interpreter will convert our keys into strings):

```
[ 
	[“peanutButter”, 2.49],
	[“cleaningWipes”, 4.99],
	[“cannedPeas”, 0.67],
	[“cereal”, 2.99],
	[“milk”, 2.19]
]
```

## Sorting the New Array

Now that we have an array, we can use JavaScript’s built in `sort` functionality to efficiently sort this array the way we want it. Because `sort` sorts the array in place in a destructive manner, we do not have to save it to a new variable and can just call on `groceriesToSort` afterwards to deal with the fully sorted array.

```
groceriesToSort.sort(function(itemA, itemB) {
	return itemB[1] - itemA[1]
}

//or, using an arrow function

groceriesToSort.sort((itemA, itemB) => itemB[1] - itemA[1])
```

There are two things to take note of in the comparison function inside the `sort` function. First, when we do the comparison itself (`itemB[1] - itemA[1]`), we are actually comparing the second value in the array, or the price. We are not comparing the two item arrays as a whole. Second, because we want the list to be a descending list, with the highest priced item first, we are subtracting itemA’s price from itemB’s price. If we wanted to return a list in ascending order, we would subtract itemB’s price from itemA’s.

Our `groceriesToSort` array should now look like the following:

```
[
	[“cleaningWipes”, 4.99],
	[“cereal”, 2.99],
	[“peanutButter”, 2.49],
	[“milk”, 2.19],
	[“cannedPeas”, 0.67]
]
```

## Creating a Results Array

With our sorted array, we can now iterate through that array to create a new array with our final result: the keys ordered by descending price. We can do this two ways. We can either use a simple `for` loop, or we can use `map` to create the new array. I’ll show you both ways to do it below:

```
let resultsArray = []

for (let i = 0; i < groceriesToSort.length; i++) {
	resultsArray.push(groceriesToSort[i][0]);
}

//or, using map

let resultsArray = groceriesToSort.map((item) => item[0]);
```


Note that in both cases, we are only pushing the zero index of the array, or the key, into the results array. We only want that part of it, and not the values associated with that key. Now, our `resultsArray` looks like the following, which was our goal:

```
[“cleaningWipes”, “cereal”, “peanutButter”, “milk”, “cannedPeas”]
```

## Abstracting Away to a Function

So far, we have seen how to sort an object’s keys by descending value. What if we wanted to abstract that into a function so we can do it with any object? That’s right! We can! 

```
function sortObjectKeysByDescendingValues(obj) {
	let sortingArray = []

	for (const key in obj) {
		sortingArray.push([key, obj[key]]);
	}

	sortingArray.sort((itemA, itemB) => itemB[1] - itemA[1]);

	let resultsArray = sortingArray.map((item) => item[0]);

	return resultsArray;
}
```

## Conclusion

Objects are handy ways to store data in JavaScript, but sometimes manipulating that data can be tricky. Just because there is no way to natively sort an object by its values does not mean that it can’t be done! By using other available data structures and iterations, we can achieve our desired goal of sorting. We were even able to abstract this out to a function that can be used on any object! This method can be easily adapted to return an ascending sort, as we saw in the sorting section of this post. Overall, this was a fun challenge to attempt, and I hoped you gained some insight from this post!
