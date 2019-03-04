---
layout: post
title:      "For and While Loops in JavaScript"
date:       2019-03-04 18:32:18 +0000
permalink:  for_and_while_loops_in_javascript
---


Since graduation from Flatiron School’s Online Web Development program, I have been looking at data structures and algorithms in JavaScript. I have noticed that both “for” and “while” loops come up repeatedly (no pun intended) in covering these topics. In this blog post, I intend to take a deeper dive into both the “for” and “while” loops. This will not go into iteration topics like “for.. in” or forEach (perhaps a future blog post?), but will focus on an in-depth examination of two of the most common looping structures available in JavaScript.

## Looping
According to the [MDN documents](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration), loops provide a straightforward way “to do something repeatedly”. This can be anything from console.logging something a number of times to iterating through a collection of items like an array. 

## The “for” loop

The “for” loop is the most common loop in JavaScript. It will continue to repeat until a specified condition evaluates to false. It consists of four parts of the following structure:

```
for ([initialization]; [condition]; [iteration]) {
	[loop body]
}
```

* `initialization`- This expression initializes one or more loop counters. You can also declare variables here. Be careful to use “let” and not “const” to declare your variable, because it needs to change over the course of the loop. 
* `condition`- This expression is evaluated before each pass of the loop, and the loop will continue to be executed until the expression evaluates to false, at which point the looping behavior will stop.
* `iteration`- At the end of each execution of the loop body, this iteration expression will execute. Generally, this will either increase or decrease the counter established in the initialization expression. 
* `loop body`- This is the code that is executed each time the loop runs. 

In terms of order of things that happens during the looping process, the following control flow happens:
1. The initialization expression is executed.
1. The condition is evaluated to see if it is true or false.
1. The loop body is executed.
1. The iteration is executed, which generally changes the counter.
1. Return to step 2.

For a more concrete example:

```
for (let i = 0; i < 3; i ++) {
	console.log(`This is loop ${i}`)
}
//This is loop 0
//This is loop 1
//This is loop 2
```

Notice that this ran three times. After the third time through the loop (when it console.logged “This is loop 2”), the `i` counter will increase to 3, which makes the condition false, and the loop stops. 

It is important to ensure that you do not get stuck in an infinite loop. For example, can you see what is wrong with the code below?

```
for (let i = 5; i < 7; i --) {
	console.log(“Hello!”)
}
```

In this case, the counter will increment down each time, so the first loop has a counter of 5, then 4, then 3. At no point will the counter ever reach 7, so the condition will always be true. In this case, the system will console.log(“Hello!”) forever! 

These “for” loops can also be used to iterate over strings and arrays. Because strings and arrays have indexes that can be accessed, we can use our counter as a proxy for that index. Let’s take a look at some examples:

```
let sample = “hello”

for (let i = 0; i < sample.length; i += 2) {
	console.log(sample[i])
}
//h
//l
//o
```

Note how in this example, our counter incremented by 2 each time the loop executed its loop body. So, the first time, it logged `sample[0]`, which was “h”, then `sample[2]`, which was “l”, then `sample[4]`, which was “o” before finally reaching a condition where `i=6`, which is greater than the length of the sample string, breaking the loop.

The same process can be used to iterate through arrays! 

```
const primes = [2, 3, 5, 7, 11]

for (let n = 0; n < primes.length; n ++) {
	if (primes[n] % 2 === 0) {
		console.log(primes[n])
	}
}

//2
```

Here, you can see that first, you don’t have to use `i` for a counter variable. You can use anything! Also, you can build multiple lines of code into your loop body and still have it work. I added a conditional statement to see if the remainder of dividing by 2 is 0. 

## The “while” loop

The “while” loop works similarly to the “for” loop in that it will keep running while a condition is true. The general structure is as follows:

```
while ([condition]) {
	[loop body]
}
```

Here, while the condition you set forth is true, the “while” loop will continue to execute the loop body. 

The control flow is as follows:
1. Evaluate the condition set forth. If true, continue. If false, exit the loop.
1. Execute the loop body.
1. Return to 1.

For example: 

```
let counter = 5

while (counter > 0) {
	console.log (counter)
	counter --
 }

//5
//4
//3
//2
//1
```

You can also use this for times when you are uncertain how many times a loop will run. Take this example from the learn.co curriculum:

```
function maybeTrue() {
	return Math.random() >= 0.5;
}

while (maybeTrue()) {
	console.log(“Hello”);
}
```

You don’t know if the `Math.random()` will return a number greater than or less than 0.5, so you do not know how many times the console.log will run. 

Just like with the “for” loop, you need to make sure you don’t send yourself down an infinite loop. You need to make sure your loop will eventually terminate. What’s wrong with the code below?

```
let counter = 5

while (counter < 10) {
	console.log(counter);
	counter --;
}
```

Here, the counter is decrementing each loop, so it will never reach ten, and will turn into an infinite loop. 

## Conclusion

Both “for” and “while” loops can be very helpful when working with data in JavaScript. While I have only scratched the surface in this post, I hope it has given you a clear understanding of the basics of both “for” and “while” loops. I plan on writing more posts in the future concerning iteration techniques in JavaScript to dive even more in depth into looping and iteration. 

