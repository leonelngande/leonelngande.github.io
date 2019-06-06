---
layout: post
hidden: false
title: On Exception Handling with try/catch in Javascript
tags:
  - Javascript
  - Exceptions
  - Exception Handling
  - try/catch
---
Early today was researching why I don't see a lot of exception handling with try/catch in Javascript codebases.

I got the best response in this:

> The try-catch-finally construct is fairly unique. Unlike other constructs, it creates a new variable in the current scope at runtime. This happens each time the catch clause is executed, where the caught exception object is assigned to a variable. This variable does not exist inside other parts of the script even inside the same scope. It is created at the start of the catch clause, then destroyed at the end of it.
>
> Because this variable is created and destroyed at runtime, and represents a special case in the language, some browsers do not handle it very efficiently, and placing a catch handler inside a performance critical loop may cause performance problems when exceptions are caught.

That's from [this answer](https://stackoverflow.com/a/12609630/6924437) on Stackoverflow, which pulls the above statement from this [Dev.Opera article](https://dev.opera.com/articles/efficient-javascript/?page=2#trycatch).

_Tip:_ There are practical code examples in the Dev.Opera article.

Even more importantly I came upon [this other answer](https://softwareengineering.stackexchange.com/a/189226) on Stackoverflow which clearly outlines the use cases for exception statements.

> The use case that exceptions were designed for is "I just encountered a situation that I cannot deal with properly at this point, because I don't have enough context to handle it, but the routine that called me (or something further up the call stack) ought to know how to handle it."
>
> The secondary use case is "I just encountered a serious error, and right now getting out of this control flow to prevent data corruption or other damage is more important than trying to continue onward."
>
> If you're not using exceptions for one of these two reasons, there's probably a better way to do it.

The above statement is what prompted me to put all this a blog post if just so it sticks more in my head. I use exceptions in my Laravel APIs mainly for error handling, and the above statement clearly set some boundaries for me.

What are you thoughts on this.., do you often throw and catch exceptions in your Javascript code?
