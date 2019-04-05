---
layout: post
title:      "Variables in PHP"
date:       2019-04-05 15:18:28 +0000
permalink:  variables_in_php
---


As I’m starting to learn PHP for a new job (hooray!), I thought it might be useful to do a little review on the most basic of topics: variables. While most of the information about variables is similar to Ruby and JavaScript, there are a few important distinctions.

## Variables: The Basics
Variables are like containers for data. They store data so that we can easily reuse them throughout the program. Variables have names, and in PHP, there are some rules to naming our variables:
* Variables always start with a dollar sign (`$`). We call this a sigil, which just means that it shows our program that we are dealing with a variable.
* Variable names can contain letters, numbers, or underscores. They cannot contain dashes or other symbols. 
* Variable names can begin with a letter or an underscore, but never a number.
* Variable names are case-sensitive. `$my_variable` is different from `$My_variable`.
* As a convention, we use snake case to name variables, so we separate words with an underscore. 

## Declaration and Assignment
There are two distinct processes associated with creating variables: declaration and assignment. Declaring a variable is the process of reserving a variable name, or a word that we can later refer to in the code. Assignment is the process of associating a value with that variable name. Every time going forward, we can then refer to the variable name and the computer will pull up the value associated with it.

In PHP we can declare a variable without assigning it. To declare a variable, simply write the variable name, like `$my_variable;` (you always need to end a statement with a semicolon in PHP). Assignment is just as easy. You just have to use the assignment operator, or the equal sign (`=`). So we can assign a value by using a statement like `$my_variable = “Hello”;`. You can declare and assign a value at the same time; you don’t need to declare a variable separately from the assignment process. Values can also be re-assigned at any time by simply writing the name of the variable and the assignment operator before the new value. 

We can use more complex statements during the assignment process as well. For example, see the following statement: 

```
$my_number = 5 + 4;
echo $my_number;
// 9
```

We see that during the assignment process, the computer evaluated everything to the right of the assignment operator before assigning the value to the variable name. 

## Assigning by Reference
We can assign a new variable to a previously declared and assigned variable’s value. For example:

```
$first_variable = “Tina”;
$second_variable = $first_variable;
echo $second_variable;
//Tina
```

We see that `$second_variable` now points to the value of `$first_variable`. What gets interesting is when we start reassigning variables.

```
$first_variable = “Tina”;
$second_variable = $first_variable;
$first_variable = “Louise”;
echo $second_variable;
//Tina
```

What’s interesting here is that even though we reassigned the value of `$first_variable` to “Louise”, the value of `$second_variable` is still “Tina.” This is because variables in PHP are assigned by reference. The new variable is a copy of the value of the original variable, but it is independent from the original variable. 

## Associating Variables
If we want to associate two variable names, we can create an alias for a variable name by using the reference assignment operator (`=&`). This means that whenever we change the value of one variable name through reassignment, the value of the other variable name will change as well. For example:

```
$my_original_variable = “Bob”;
$my_new_variable =& $my_original_variable;
$my_original_variable = “Linda”;
echo $my_new_variable;
//Linda
```

The opposite will work as well: 

```
$my_original_variable = “Teddy”;
$my_new_variable =& $my_original_variable;
$my_new_variable = “Gene”;
echo $my_original_variable;
//Gene
```

These two variable names are now associated with each other. All you’ve done is assign an alias to the first variable’s name. 

## Conclusion
Variables in PHP behave very similarly to variables in other languages. However, some rules still need to be followed to ensure that variables will behave in an expected way. I hope this blog clearly explained the concepts and shed some new light on variables in PHP!
