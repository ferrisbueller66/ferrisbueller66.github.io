---
layout: post
title:      "Crib Notes on Javascript's Hoisting, Closures, and Arrow Functions"
date:       2020-07-25 01:40:01 -0400
permalink:  crib_notes_on_javascripts_hoisting_closures_and_arrow_functions
---


I won't try to reinvent the wheel. Plenty has been written on these three related topics in Javascript. The MDN material is robust with resources (such as these entries on Closures and Hoisting).
Additionally, Sukhjinder Arora did 3 consecutive pieces on the topic in 2018 (First on Hoisting, then on Closures, then on scope and scope chain). What do I have to add to these topics? Nothing other than consolidation. Sukhjinder has done a great job and I encourage you all to read his 3 blogs. My attempt here is to consolidate everything into one single blog for easier reference, rearrange the logic and ordering of the material to make it more accessible, and highlight Javascript terminology to ensure these concepts are understand.


To do that we need to start with terminology. Let's get some definitions set, so we're all on the same page with the terms we're using.

## First some basic, basic terminology
A variable is *declared*:

`let coffee`

and it is assigned value, which initializes the variable:

`coffee = "What he needs if he's going to explain Hoisting to you"`

The same goes for functions. Functions are declared:

```
function pourDrink(coffee){
  console.log(`Please pour Trevor a big cup of coffee because that is ${coffee}`)
}
```

And then they are *invoked* (if you're feeling like a member of royalty), *executed* (if you're feeling angry and like a member of royalty), or *called* (if you feel like a vassal programmer trying to write your quota of code for the week):

`pourDrink(coffee)`


Now,  3 variable types in Javascript also need mention at this point:

1. Var: if a variable with *var* is declared *without* being assigned a value at the same time, the JS engine will automatically assign it a value of *undefined*.

2. Let: a variable with *let* can be declared without being assigned a value, but if JS is called to use this variable before it initializes, JS will assign it a value of *undefined*.

3. Const: you have to assign a value to a variable with *const* when the variable is defined. The following snippet returns a syntax error:
`const coffee //=> SyntaxError: Missing initializer in *const* declaration`
*Const* has to be initialized when it is declared.

Also, you might [have] reach[ed] a point in your career where you can parse the difference between assigning value and initiliazing a variable. That's excellent. For those just getting started, when you assign value to a variable, you're initializing it. When you initialize a variable, it's being assigned value. It isn't quite the same, but when beginning JS, all these terms get thrown around and become confusing. You can think of initialize and assign as very closely related, and we'll discuss these 2 just a little bit more in the section below.

The takeaways from this are that *const* is the most stable form of variable to utilize before you can't forget to assign it a value when you. Var is the least desirable variable type to use because JS will automatically assign it a value of undefined if you don't assign it a value when it is declared.

Take a quick break; get up and walk around, and then take a minute to review before you tackle a quiz, presented to you by Teenage Mutant Ninja Turtles 3 low level boss, Rocksteady.

![](https://i.imgur.com/V2Rr4sb.png)

### Level 1 Quiz:

1. What is the terminology used for creating variables and functions?
2. What are 3 important types of variables in JS?
3. How are each 
4. Write an example of declaring a variable
5. Write an example of assigning value to a variable
6. Write an example of both declaring and assigning value to a variable


----------------------------------------------------------------------------------------------------------------------------------------
## Second, now some basic terminology
The JS engine takes to passes to run your code:
1. The first pass is the compilation phase in which it does 3 things:
     1. as it hits each variable and function declaration, it allocated memory for them
     2. it creates an execution context for each function
     3. it sets the references to any and all parent scopes for that particular execution context
2. The execution phase then runs through the program again assigning values to variables (initializing them) and executing any functions that are invoked (that's right. I'm feeling that good about myself right now).

We just covered a ton in those 5 bullet points, so let's take some time to unpack them.

Take a quick break; get up and walk around, and then take a minute to review before you tackle a quiz, presented to you by Teenage Mutant Ninja Turtles 3 low level boss, GroundChuck.



![](https://i.imgur.com/CGhUZBZ.png)

### Level 2 Quiz:

1. What are the 2 stages of processing by the JS engine?
2. What happens to variables and functions in each stage?

----------------------------------------------------------------------------------------------------------------------------------------
## Level 3 (Hoisting) - Some not so basic terminology

If you're coding in repl.it or another IDE, run this code:
```
let coffee
console.log(coffee)
coffee = "what I need if I'm going to explain Hoisting to you."
```
The console.log(coffee) //=> *undefined*.

Reviewing the compilation/execution phase, this makes sense. JS first sees the declaration for the variable coffee in the compilation phase. No assignment is conducted in this phase. In the execution phase, the assignment of value doesn't occur until line 3. When JS starts at line one with an undefined, unitialized variable (coffee), it initializes it and assigns it the value undefined. This couldn't happen with *const* (why?).

Let's try this slightly differently with the following code:
```
console.log(coffee)
var coffee = "what I need if I'm going to explain Hoisting to you."
```

What happens when we run this code? Again, we get
`The console.log(coffee) //=> undefined.`

But we declared and assigned value to var in the same line of code (var coffee = "what I need if I'm going to explain Hoisting to you.").

This is an example of an issue caused by hoisting.

Before you read further, think about these 2 lines of code. What are the phases of the JS engine when it runs?
Can you explain what's happening?

In the compilation phase, var coffee is being declared as a variable, but it is not assigned value at this phase - that comes in the execution phase. When the execution phase arrives, the variable is called *before* the JS engine has seen code to assign it value. So in line 1, JS initializes it with a value of undefined. Then in line 2 it assigns the variable *coffee* the value ""what I need if I'm going to explain Hoisting to you."

This is hoisting. Hoisting is a lousy term. Your code doesn't actually hoist up or move. It is simply an issue caused by the order of operations in the JS engine. Hoisting causes issues when you call a variable before assigning it value and initializing it. That's it.

2 things can prevent problems caused by "hoisting":

1. use *const* and never use *var*. Why? Because you have to assign a value when you declare a variable with const, and JS initializes it when it is declared (does it?). Using *const* will largely mitigate these hoisting problems.
2. Put your variable assignments at the top of your code. Since hoisting is an order of operation issue, if the variables in your code are assigned value and initialized before anything else, you'll also mitigate against hoisting issues. The above code reversed renders without any issue:

```
var coffee = "what I need if I'm going to explain Hoisting to you."
console.log(coffee)
```

You ready for LeatherNeck, Boss #3?

![](https://i.imgur.com/THR7yY1.png)
hoist yourself up, buddy! (too soon?)

### Level 3 Quiz:

1. What are the 2 stages of processing by the JS engine?
2. What is hoisting, and use an example in repl.it as you explain it
3. Specifically describe what issues with a variable can result from hoisting.
4. Give 2 preventive measure to avoid these issues.

----------------------------------------------------------------------------------------------------------------------------------------
## Level 4 (Execution Context)

Execution Context
Global, Function, and Scope

Closures
Arrow Functions to follow in the next 48 hours...!
