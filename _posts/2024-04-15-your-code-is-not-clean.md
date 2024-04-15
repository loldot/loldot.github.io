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
good news is: no one cares either.

Today, Ozan Yarcı showed an example of an anti-pattern on twitter and it sparked a 
lot of engagement. 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This code structure is called an arrow anti-pattern. How to fix this code? <a href="https://t.co/epXYnN0RVB">pic.twitter.com/epXYnN0RVB</a></p>&mdash; Ozan Yarcı (@ozanyrc) <a href="https://twitter.com/ozanyrc/status/1778921269670342776?ref_src=twsrc%5Etfw">April 12, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

Many people claimed to have the only correct solution. Two pretty distinct groups of 
critism formed; the performance camp and the clean code camp. The former worried about
the possible issues with branch predictions the code poses, and the latte worried about
how unreadable the code is. It would seem most people agreed that the code was bad. But 
no, one developer would have fired everyone not conforming to his opinion which was: 
keep the code as it is. He even strongly opposed my favorite of the posted solutions for 
being "impossible to set a breakpoint in".

```csharp
public string CheckGrade(int score)
{
    if (score >= 90) return "A";
    if (score >= 80) return "B";
    if (score >= 70) return "C";
    if (score >= 60) return "D";

    return "F";
}
```

So how can I claim that no one cares about the clean code, when clearly a lot of people
have very strong opinions about it, and spend a lot of time discussing the issue on 
twitter? In the end, "how clean is the code" just isn't a metric anyone ever use to measure 
the quality of a software product. Most of the time, with closed source software, it isn't 
even available. Good software is correct and performant - and while clean code indirectly 
can have an effect on those metrics, there are many programs out there which have a "clean" 
codebase, but is the product is just miserable.

The function in question is either too trivial to ever be read again or it is wrong,
in which case all the other proposed solutions are also wrong. Spending time discussing how
the ifs are nested is just plain and simple bike shedding.

So yes, even though a lot of developers have strong opinions on this piece of code, when 
its compiled they don't really care. As long as this function is correct and is not slowing 
down some hot path, it is unlikely anyone would ever have to look at it again after the code 
review.