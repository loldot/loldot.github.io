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
twitter?

I am almost a 100 percent certain that none of those people, if they stopped bike shedding 
for a little while, ran a program with any of the proposed solutions would be able to
tell the difference. If this piece of code is correct and is not slowing down some hot path, 
no one should ever have to look at it after the code review. However, almost every person 
chiming in failed to mention what in my opinion is the biggest issue; the grading only makes 
sense with a maximum score of 100, but the signature does not reveal that. Any developer 
using this method will have to look at its implementation to understand what they should pass 
as an argument (or *just know* that the score parameter should be between 0 and 100). In 
my opinion it does not make sense on its own, and should be a private method. I would
probably have renamed it as well. 