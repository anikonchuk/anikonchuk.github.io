---
layout: post
title:      "Solving the Diagonal Difference Problem"
date:       2019-03-11 13:40:15 +0000
permalink:  solving_the_diagonal_difference_problem
---


As I was working over the weekend to brush up on my JavaScript problem solving skills on HackerRank.com, I came across the Diagonal Difference problem. Given a square matrix, I was tasked with calculating the absolute difference between the sums of its diagonals. At first, I was a bit overwhelmed, but I was able to reason through the problem and come up with (what I think is) a good solution to it. In this blog, I will detail the thought process and reasoning that led me to my answer.

## The Problem

As stated above, I was given a square matrix, which could be any size, and was tasked with returning the absolute difference between the sums of the diagonals of the matrix. An example matrix of 4x4 size would look like the following:

```
5  8  3  2
3  9  1  6
6  7  4  2
3  4  9  6
```

The left-to-right diagonal would be 5 + 9 + 4 + 6 = 24. The right-to-left diagonal would be 2 + 1 + 7 + 3 = 13. The absolute difference would be | 24 - 13 | = 11, where the vertical bars (||) represent absolute value. 

The matrix was given in the form of an array of arrays, like the following:

```
arr = [
	[5, 8, 3, 2],
	[3, 9, 1, 6],
	[6, 7, 4, 2],
	[3, 4, 9, 6]
]
```

## Visualizing the Solution

My first step to figuring out the problem was to try to start to find a pattern in what I was adding together. I started by making the following table:

![](https://imgur.com/a/dfqo9d1)

I could start to see a pattern emerging. For the left-to-right diagonal, each additional number increased its first and second index by 1. For the right-to-left diagonal, the first index increased by one each additional number, and the second index decreased by one. The key would be to find a way to represent that second index in an iterative way.

## Pseudocode

At this point, I was ready to start planning my approach to the problem by writing some pseudocode.

```
function diagonalDifference(arr) {
	set up variable for leftToRightDiagonal
	set up variable for rightToLeftDiagonal

	iterate through the array and sum up the leftToRightDiagonal
	iterate through the array and sum up the rightToLeftDiagonal

	find the absolute difference between the diagonals
}
```

The key now was to figure out the iterations and the relationship for the second index of the right-to-left diagonal.

## Figuring Out the Iterations

I realized that the way to figure out the second index of the right-to-left diagonal was to integrate the length of the array into the iteration in my for loop.

![](https://imgur.com/a/UdRak5L)

## Writing the Solution

Looking back at my pseudocode, I saw that I also needed to set up a variable for the length of the array, and I could combine the two iterations into one for loop. My final solution is as follows:

```
function diagonalDifference(arr) {
	const matrixLength = arr.length;
	let leftDiagonalSum = 0;
	let rightDiagonalSum = 0;

	for (let i = 0; i < matrixLength; i++) {
		leftDiagonalSum += arr[i][i];
		rightDiagonalSum += arr[i][n - i - 1];
	}

	return Math.abs(leftDiagonalSum - rightDiagonalSum);
}
```

## Conclusion

While initially, I was puzzled by this puzzle, I was able to figure out the solution by writing out charts to better visualize relationships. By taking apart the problem and using an example set of data to deconstruct the patterns, I was able to find a working solution to the problem. Looking at the Big O notation of the solution, which in this case is O(n) due to the single iteration through the length of the array, I do not see a way to increase the efficiency of this solution. Overall, this was a fun puzzle to solve that gave me more insight into the problem-solving process!
