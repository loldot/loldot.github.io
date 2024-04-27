---
layout: post
title: Cleaning out the bike shed
author: Lorentz Vedeler
date: 2024-04-15
tags:
    - Clean Code
    - Development
---

Ozan Yarcı showed an example of an anti-pattern on twitter and it sparked
a lot of engagement.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This code structure is called an arrow anti-pattern. How to fix this code? <a href="https://t.co/epXYnN0RVB">pic.twitter.com/epXYnN0RVB</a></p>&mdash; Ozan Yarcı (@ozanyrc) <a href="https://twitter.com/ozanyrc/status/1778921269670342776?ref_src=twsrc%5Etfw">April 12, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


The tweet and its responses highlight how easily programmers are distracted by
minutia. In fact it is so common that there is a term for it: bike shedding.

Most of the suggestions in the replies and quote tweets are versions of the code
where the logic of the function is structured in a more aesthtically pleasing
way. The problem is, that no matter how you structure that function, someone is
going to hate the way you did it with a passion. The heated debate on twitter
proves just that. More importantly, though, is that many of the refsctorings are subtly different. For instance a variant which calculates an index in an array to find the correct grade will fail if the score is higher than a hundred or less than zero. The author also propses an even more subtly wrong solution which is based on looping through a C# dictionary, where the keys are the lower bounds and the value the letter of the grade. however, if you look up the documentation for dictionaries you will find that they are not guaranteed to be sorted by keys. I think an important reason the book Clean Code has fallen out
of favor lately is that many of its most adamant proponents measure code quality
by the aesthtics instead of other, more important metrics.

At the function level, good code is first and foremost a correct solution to a
business problem. As the code was given to us with no other context, we can only
assume it is correct. In order to write good code, you should spend more time on
understanding the business problem and less on thinking about the technical
implementation. Only when you understand the problem and solution can you write
the code that solves the problem.

Good code should also be efficient and easy to understand. I think it is
unlikely that the function is going to become a performance bottleneck. Even
though the code looks prone to performance issues due to branch predictions, the
compiler might optimize that away entirely. We would have to inspect the
disassembly and profile the compiler optimized release build to make any
conclusions about performance.

That leaves us with "easy to understand". Of the three qualities mentioned,
"easy to understand" is the most subjective. What a person considers
understandable depends heavily on their background: both their knowledge of the
business domain and programming in general, but also on how that person thinks
about the problem. When you hire people and train people, you set and raise the
bar for prerequisite knowledge. While you should aim to make the code as easy to
understand as possible, the bar needs to be set somewhere. In my opinion, most
of the code in a project should be understandable by interns and junior
developers, but it would be silly if a company developing in Haskell could not
use monads just because they are hard to understand. Similarly, we do not add
comments explaining in detail what each keyword does; everyone reading the code
is expected to have that knowledge. I don't think the if statements in the
example code are hard to understand, although I would probably not have written
it like that myself. I find the signature a little confusing though; the name is
weird and I would be a little concerned that it is public. Minor details, but
ones I think are more important than the inner workings of the function. For
example what would you expect the result of `CheckGrade(9999)` to be? Now you
would have to check the implementation to know what it does. My guess is that
this function only makes sense in a certain context and it should be private or
internal to the package.

It is usually pretty easy to write code that is good when it is written. The
hard part is writing code that is still good long after it was written. It seems
like code becomes bad over time, often referred to as code rot. I think that is
a bad term, because it sounds like the code is just going to rot by itself; like
if you forget the code in the back of the fridge, after a while its just going
to spoil. When code becomes unmaintainable, it is not the fault of the code, but
the programmers. The code was either bad to begin with or the maintainers make
it bad. However, the way your conditionals are structured within a function have
little effect on maintainability in the larger context.

What happens to the `CheckGrade` method when a course is graded according to a
gaussian curve of the test scores instead of the static lower limits? What
happens when you get a customer in Europe that has an 'E' grade as well? Or when
Alex should never get an 'F' because his parents funded the university's new
computer science lab? The author of the code did not accomodate those possible
future changes. It is easy to blame the original authors for making assumptions
that does not hold when we have to change the code, but it is impossible to
model an accurate representation of reality, because reality, and our perception
of it, constantly changes.

Sometimes it is better to not make any predictions about the future, but if your
code is not future proof it should not be exposed to the rest of the codebase.
All code makes assumptions, being aware of the assumptions you make, and the
certainty of those assumptions is helpful when you write code. The more
uncertain assumptions should not be depended on. At the beginning of a software
project, very little is certain. When you start out, you should focus on a few
boundaries, interfaces and api's. Because they are depended on, interfaces and
api's should try to encompass future needs. Interfaces and api's encode our
assumptions about the future. Since we expect our understaning of the problem to
change, and our assumptions to be challenged, we should aim to make the
boundaries between modules general and small, but the modules themselves should
be large, so they can encapsulate as much knowledge and as many assumptions as
possible. The book "A philosophy of software design" calls this deep modules.
For example, maybe the only public method of our module should be:

```csharp
public class AssignmentEvaluation
{
    public Grade Evaluate(Assignment assignment)
    {
        var score = Score(assignment);
        var gradeLetter = CheckGrade(score);

        return new Grade(gradeLetter);
    }

    private int Score(Assignment assignment) { }
    private string CheckGrade(int score) { }
} 
```

Our interface supports all the mentioned use cases, but we limit our initial
implementation to what we need right now. Beacause we are careful to only expose
assumptions we are pretty certain is going to hold for the future, we are free
to change all the inner workings later.

As you realize your assumptions do not hold, it might be better to discard what
you already have and start again. That is why it is important to
compartmentalize the code, so that it is easy to start over on just a smaller
part. Code that is cohesive and loosely coupled tends to make it easy to delete
and replace smaller parts within a larger module. When modules are too
intertwined, it is much harder to fix bad design desicions. The end result is
often that these bad desicions are kept untill the only possible solution is a
big rewrite.

In the end, any time you write code, its purpose should be to solve a problem.
Most of the time it should solve a business problem. Sometimes it might be
necessary to solve a problem with existing code. And once in a while, it may
make sense to proactively solve a problem you do not even have at the moment. Be
wary of predicting the future though - it is incredibly hard, and that
uncertainty makes it tempting to focus on pointless technical dogma instead.
