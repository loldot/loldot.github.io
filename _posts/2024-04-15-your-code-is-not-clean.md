---
layout: post
title: Your code is not clean
author: Lorentz Vedeler
date: 2024-04-15
tags:
    - Misc
    - Development
---

If you have written a non-trivial amount of code, no one thinks your code is *clean*.
Not even you - if you think so right now, just wait a few months and check again. The
good news is: no one cares about your code either.

Today, Ozan Yarcı showed an example of an anti-pattern on twitter and it sparked a
lot of engagement.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This code structure is called an arrow anti-pattern. How to fix this code? <a href="https://t.co/epXYnN0RVB">pic.twitter.com/epXYnN0RVB</a></p>&mdash; Ozan Yarcı (@ozanyrc) <a href="https://twitter.com/ozanyrc/status/1778921269670342776?ref_src=twsrc%5Etfw">April 12, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Many people claimed to have the only correct solution. Two pretty distinct groups of
critism formed; the performance camp and the clean code camp. The former worried about
the possible performance issues with branch predictions, and the latte worried about
how unreadable the code is. It would seem most people agreed that the code was bad. But
no, one developer would have fired everyone not conforming to his opinion which was:
keep the code as is. He even strongly opposed my favorite of the posted solutions for
being "impossible to set a breakpoint in".

So how can I claim that no one cares about the clean code, when clearly a lot of people
have very strong opinions about it, and spend a lot of time discussing the issue on
twitter?

The code in the example is not bad. By some peoples definition of clean code, it is not
"clean", but it is not hard to reason about. If you struggle to understand that method,
you should probably try to get a deeper understanding of C-based programming languages.
The truth is that, even though a lot of developers have strong opinions on this piece of
code, they do not struggle to understand what it does. What's more problematic about this
code is that it is public, even though it should be a private method. The grading only
make sense on a scale from 1 to 100, but the score parameter is an int that is not checked.
The signature reveals nothing about the way the function works, so any one trying to reuse
the method will have to look at the defintion to understand how to call it. Most of the
clean alternatives failed to mention that.

In stead, the tweet and its responses highlights how easily programmers are distracted
by minutia. In fact it is so common that there is a term for it: bike shedding. The
tweets also prove that there just isn't a commonly accepted definition for clean or good
code. No matter how you structure that function, some one is going to hate it with a
passion. Any time you write code, its purpose should be to solve a problem. Most of the 
time it should solve a business problem. Sometimes it might be necessary to solve a 
problem with existing code. And once in a while it may make sense to proactively solve 
a problem you do not even have at the moment. Be wary of predicting the future though - 
it is incredibly hard, and that uncertainty makes it tempting to focus on pointless 
dogma instead. 

In the end, "how clean is the code" just isn't a metric anyone ever use to measure
the quality of a software product. Most of the time, with closed source software, it isn't
even available. Good software is correct and performant - and while clean code indirectly
can have an effect on those metrics, there are many programs out there which have a "clean"
codebase, but the product is still miserable. If the function in question is properly
encapsulated, works as expected, and is not in a performance hot path. No one is going to
look at it again. And that is what you should aim for: code that no one has to read again -
code that just solves a problem.
