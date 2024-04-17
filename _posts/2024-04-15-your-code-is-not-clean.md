---
layout: post
title: Your code is not clean
author: Lorentz Vedeler
date: 2024-04-15
tags:
    - Clean Code
    - Development
---

If you have written a non-trivial amount of code, no one thinks your code is
*clean*. Not even you - if you think so right now, just wait a few months and
check again. Code with bugs is, by definition, not clean. And based on my
anecdotic evidence, I conclude that all software has bugs, ergo no code is
clean. Feel free to prove me wrong.

Today, Ozan Yarcı showed an example of an anti-pattern on twitter and it sparked
a lot of engagement.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This code structure is called an arrow anti-pattern. How to fix this code? <a href="https://t.co/epXYnN0RVB">pic.twitter.com/epXYnN0RVB</a></p>&mdash; Ozan Yarcı (@ozanyrc) <a href="https://twitter.com/ozanyrc/status/1778921269670342776?ref_src=twsrc%5Etfw">April 12, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


Many people claimed to have the only correct solution. Two pretty distinct
groups of criticism formed; the performance camp and the clean code camp. The
former worried about the possible performance issues with branch predictions,
and the latter worried about how unreadable the code is. It would seem most
people agreed that the code was bad. But no, one developer would have fired
everyone not conforming to his opinion which was: keep the code as is. He even
strongly opposed my favorite of the posted solutions for being "impossible to
set a breakpoint in".

The code in the example is not bad. I will concede it is not how I would have
written it, but it does not need fixing, at least not for the reason the tweet
author claims. By some people's definition of clean code, it is not "clean", but
it is not hard to reason about. If you struggle to understand that method, you
should probably try to learn a C-based programming language. The truth is that,
even though a lot of developers have strong opinions on this piece of code, they
do not struggle to understand what it does. What's more problematic about this
code is that it is public, even though it should be a private method. The
grading only makes sense on a scale from 1 to 100, but the score parameter is an
int that is not checked. The signature reveals nothing about the way the
function works, so anyone trying to reuse the method will have to look at the
definition to understand how to call it. Most of the clean alternatives failed
to mention that.


Instead, the tweet and its responses highlight how easily programmers are
distracted by minutia. In fact it is so common that there is a term for it: bike
shedding. The tweets also prove that there just isn't a commonly accepted
definition for clean or good code, because a lot of developers just use
aesthetically pleasing as a measure for clean code, but code does not have to be
aesthetically pleasing. Suggesting a rewrite to something "more aesthetic", but
semantically equivalent is a waste of everyone's time unless everyone on the
team has the same definition. No matter how you structure that function, someone
is going to hate it with a passion.


If code is easy to understand, correct, and performant, it is good code. Of
those three qualities, "easy to understand" is the most subjective one. What a
person considers understandable depends heavily on their background: both their
knowledge of the business domain and programming in general, but also on how
that person thinks about the problem. When you hire people and train people, you
set and raise th bar for prerequisite knowledge. While you should aim to make
the code as easy to understand as possible, the bar needs to be set somewhere.
Ideally, most of the code should be understandable by interns and junior
developers, but it would be silly if a company developing in Haskell could not
use monads just because they are hard to understand. Similarly, we do not add
comments explaining in detail what each keyword does; everyone reading the code
is expected to have that knowledge.


In the end, any time you write code, its purpose should be to solve a problem.
Most of the time it should solve a business problem. Sometimes it might be
necessary to solve a problem with existing code. And once in a while, it may
make sense to proactively solve a problem you do not even have at the moment. Be
wary of predicting the future though - it is incredibly hard, and that
uncertainty makes it tempting to focus on pointless dogma instead.
