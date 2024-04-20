---
layout: post
title: Your code is not clean
author: Lorentz Vedeler
date: 2024-04-15
tags:
    - Clean Code
    - Development
---

Ozan Yarcı showed an example of an anti-pattern on twitter and it sparked
a lot of engagement.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This code structure is called an arrow anti-pattern. How to fix this code? <a href="https://t.co/epXYnN0RVB">pic.twitter.com/epXYnN0RVB</a></p>&mdash; Ozan Yarcı (@ozanyrc) <a href="https://twitter.com/ozanyrc/status/1778921269670342776?ref_src=twsrc%5Etfw">April 12, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The code in the example is not bad. I will concede it is not how I would have
written it, but it does not need fixing, at least not for the reason the tweet
author claims. By some people's definition of clean code, it is not "clean", but
it is not hard to reason about. 

Instead, the tweet and its responses highlight how easily programmers are
distracted by minutia. In fact it is so common that there is a term for it: bike
shedding.

The tweets also prove that there just isn't a commonly accepted definition for
clean or good code, because a lot of developers just use aesthetically pleasing
as a measure for clean code, but code does not have to be aesthetically
pleasing. Suggesting a rewrite to something "more aesthetic", but semantically
equivalent is a waste of everyone's time unless everyone on the team has the
same definition. No matter how you structure that function, someone is going to
hate it with a passion. The book clean code has fallen out of favor lately, for
many different reasons. I think an important reason is this shift to measure
code quality by the aesthtics instead of other, more important metrics. Good
code is first and foremost correct. Nothing with bugs in it is clean. If your
carpet came back from the cleaner with bugs in it, you would ask for your money
back. 

Good code should also be performant and easy to understand. Of three qualities
mentioned, "easy to understand" is the most subjective one. What a person
considers understandable depends heavily on their background: both their
knowledge of the business domain and programming in general, but also on how
that person thinks about the problem. When you hire people and train people, you
set and raise the bar for prerequisite knowledge. While you should aim to make
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
uncertainty makes it tempting to focus on pointless dogma instead. And remember
if your code has bugs, its not clean, but it can still be quite good.
