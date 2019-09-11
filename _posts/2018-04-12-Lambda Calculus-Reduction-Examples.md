---
title: Lambda Calculus Simple Reduction Examples
topic: Math
---

This is a post I made for a now defunct blog my friends and I ran in 2018. I think it's an interesting topic, so I'm putting it up here.

I've been studying Lambda Calculus lately, and I realized that there isn't exactly a wealth of examples for reducing expressions.I might just be a dummy, but I find that looking at examples really helps me wrap my head around things.  So, in the interest of helping someone else who might be struggling, I have decided to give a brief explanantion of $$\lambda$$-calculus, and provide some simple examples of reduction that would help a beginner get going.
### What is Lambda Calculus?
Simply put, $$\lambda$$-calculus is a system of mathematics based only on applying functions to other functions. If you've ever heard of [Peano Arithemtic](http://googology.wikia.com/wiki/Peano_arithmetic), it's a pursit in a similar vein. The goal is to create as simple a formal system as possible that can still be used to solve complex problems. In the case of Peano arithmetic, we show that we only need a few axioms about natural numbers to prove a whole bunch of stuff. Lambda calculus does the same thing, but instead of proving things about numbers, we can prove things about computation. The system can be shown to be Turing complete, and allows us to determine whether problems are computable by algorithm. Though it can behave like a Turing machine, $$\lambda$$- calculus generally feels more like working with a high-level programming language. In fact, many programming languages have been developed based on lambda calculus. They are classified as functional languages, and they include: Haskell (named after [Haskell Curry](https://en.wikipedia.org/wiki/Haskell_Curry)), F#, O-Caml, and Lisp. There are functional aspects of many other programming languages.
### Some Good Resources
The basic concepts of $$\lambda$$-calculus were kind of hard for me to wrap my head around, so in the interest of helping someone who may stumble upon this page, I have a few links that might help with understanding the basics:

This one has some good illustrations, but the bird analogy becomes very tedious:
[http://dkeenan.com/Lambda/](http://dkeenan.com/Lambda/)

A lot of people seem to think this is clever, but I haven't seen anyone saying they actually learned $$\lambda$$-calculus with it:
[http://worrydream.com/AlligatorEggs/](http://worrydream.com/AlligatorEggs/)

I've been working through this textbook  It's very mathy, which is probably why I had so much trouble to start. But, if you like proof based exercises and mathematical rigour, this is a good option:
[http://www.cse.chalmers.se/research/group/logic/TypesSS05/Extra/geuvers.pdf](http://www.cse.chalmers.se/research/group/logic/TypesSS05/Extra/geuvers.pdf)

Honestly, the thing that helped me the most was this Stack Overflow response:
[https://stackoverflow.com/questions/34140819/lambda-calculus-reduction-steps](https://stackoverflow.com/questions/34140819/lambda-calculus-reduction-steps) 

In any case, if you're here, you probably have a resource that you're working with.
### Examples
I'm going to writing these examples based on the notation given in the textbook above. This may be slightly different than what you're using, so I'll try to make note of how notation may differ.
&nbsp;
$$\lambda xy.((\lambda x.yx)((\lambda z.z)x))$$
&nbsp;
Firstly, I'll point out that a lot of resources write out a lambda for each bound variable (i.e. $$\lambda x \lambda y.xy = \lambda xy.xy$$), but it's fine to drop the extra lambdas and just assume every variable before the "." is bound. The first thing we need to do is figure out where the most deeply nested application is. That would be $$\lambda xy.((\lambda x.yx)((\underline{\lambda z.z)x}))$$. So, we'll apply our function $$\lambda z.z$$ to $$x$$. We can write this as $$(z)[z := x]$$. The := is the substitution operator, so we're substituting every $$z$$ in $$(z)$$ with the variable $$x$$. This then reduces to $$(x)$$. This is known formally as _Beta-Reduction_. Now we have:
&nbsp;
$$\lambda xy.((\lambda x.yx)((\lambda z.z)x)) = \lambda xy.((\lambda x.yx)((x)))$$
&nbsp;
If we remove the unnecessary parentheses, we get $$\lambda xy.(\underline{(\lambda x.yx)x})$$ as our next application. Now, we substitute $$(yx)[x := x]$$ and get $$(yx)$$. This leaves:
&nbsp;
$$\lambda xy.(yx)$$ or $$\lambda xy.yx$$ 
&nbsp;
which is a function that swaps the order of two parameters.

This is a simple example to showcase _Alpha_ _Conversion_:
$$\lambda x.(\lambda y.xy)y$$
This expression can be reduced by substituting the $$x$$'s in $$(\lambda y.xy)$$ for $$y$$, or so it seems. The problem is that the function we are substituting into already has a variable named $$y$$, and this is not necessarily the same as the $$y$$ on the outside. If you're familiar with programming, it's the equivalent of declaring variables of the same name in a different namespaces. Fortunately, $$\alpha$$-conversion lets us change variable names. So, all we have to do is change the $$y$$ on the outside to something else, like $$y'$$, $$y_1$$, or $$z$$. The name doesn't matter. So:
$$\lambda x.(\lambda y.xy)y = \lambda x.(\lambda y.xy)z$$
$$(\lambda y.xy)[x := z]$$
$$\lambda y.zy$$

So, there are some (hopefully understandable) written descriptions. Now, I'll just show work for some more problems, and make notes if necessary.

$$((\lambda x.((\lambda y .(xy)x))(\lambda z.w))$$
Here, it turns out that the first application is with $$\lambda x$$ and $$(\lambda z.w)$$. 
$$(\lambda y.(xy)x)[x:=\lambda z.w]$$
$$\lambda y.((\lambda z.w)y)(\lambda z.w)$$
$$((\lambda z.w)y)[y := \lambda  z.w]$$
$$(\lambda z.w)(\lambda z.w)$$
$$w[z := \lambda z.w]$$
Since there aren't any $$z$$'s in $$w$$, our result is simply $$w$$.
### Encodings
So, one of the big things people like to do with lambda calculus is make encodings that do things similar to numbers or logical operators. It's kind of boring to just reduce these expressions for no reason, so I'll be using some encodings for my examples. A few simple ones are encodings for True, False, and Not.
True = $$\lambda xy.x$$
False = $$\lambda xy.y$$
Not = $$\lambda x.((x false) true)$$

So, now that we have these, let's evaluate some logical expressions using $$\lambda$$-calculus, :
not false
(not)($$\lambda xy.y$$)
$$(\lambda x.((x $$ false$$)$$ true$$))(\lambda xy.y)$$
$$((\lambda xy.y ($$ false$$) ($$ true$$)$$
$$((\lambda xy.y (\lambda xy.y)(\lambda xy.x)$$
$$(\lambda xy.x)$$
Which is the function we defined as true.
So, we can see that applying our not function to the false function returns the true function. Isn't that neat?

Let's test out something a little more complicated:
not (not true)
$$(\lambda x.((x\,\,\,false))$$ true)__(not__ __true)__
(__(not__ __true)__ false) true
((($$\lambda$$ xy.((x false) true)  __true__)  false)( true)
Note the number of parentheses, this is how we can tell we are applying the "not" function to the second "true". Also, in the step before, we can see the "not" is being applied to that "true". It can get hard to keep track of the parentheses, so it's important to be aware of what you're doing over all the steps.

(((true false) true) false) true
((($$\lambda$$ xy.x false) true) false) true
((($$\lambda$$ y.false) true) false) true
((false) false) true
$$\lambda$$ xy.y (false) (true)
true

It worked again!
That's all for now. I'm planning on making a post on [Church Numerals](https://en.wikipedia.org/wiki/Church_encoding) after some other various posts.
