---
author: "Karthik M"
title: "Demystifying code quality, one technique at a time"
date: "2020-06-24"
canonicalUrl: https://litebreeze.com/blog/2020/06/24/demystifying-code-quality/
tags:
- code quality
- programming
- software engineering
---

I shall start with two laws written 40+ years ago about software evolution by Lehman & Bélády:

1. _Law of continuous change_: Any software system used in the real-world must change or become less and less useful in that environment.
2. _Law of increasing complexity_: As a system evolves, its complexity increases unless work is done to maintain or reduce it.

Being a software engineer, I often emphasize a lot about code quality along with other aspects of application
development. A great deal of effort is required for writing clean code. The list only covers selected fundamentals in an easy
digestible format.

## Training
There’s a famous poem by Zen

```
To follow the path:
look to the master,
follow the master,
walk with the master,
see through the master,
become the master.
```

My close friend [Ajith D](https://www.linkedin.com/in/ajith-dinesan/) has always quoted the following sentence:
Three things in life which require practice

1. Driving
2. Swimming
3. Programming

Hence to be a good programmer, you require the right mindset, practice and the best apprenticeship.
To sum up: Programming is a collection of acquired SKILLS.

## Naming
Always use intention revealing names – the name of a variable, function, or class, should answer all the big questions.
It should tell you why it exists, what it does, and how it is used[1].

## Functions
A function should only do one thing and that one thing well. Software entities should be open for extension but closed
for modifications (aka Open-closed principle[2]).

## Dead code and variables
Requirements keep changing and so does the code when we develop applications using the agile methodology. Remove
unused code/variables which doesn’t affect the application. Apart from shrinking the program size, it avoids the
execution of unnecessary functionality. We can always gracefully recover from this deletion using version control
software.

## Hard coding
Avoid hard coding values, use environment variables instead. This is important as values differ in development, staging
and production. Laravel users can take one step further by utilizing
[configuration caching](https://laravel.com/docs/7.x/configuration#configuration-caching).

## Long IF conditions
A complex/lengthy IF condition will be difficult for a programmer to comprehend the logic involved in it. Moreover, you
are demanding the developer to hold/process all the half-finished thoughts in their head. Simplify long IF conditions
with smaller functions which promote easy perception.

## Error handling
Always practice the use of TRY-CATCH-FINALLY blocks for functions which generate unexpected outputs. In this manner, we
increase the reliability/robustness of the application.

## Foreign keys
Most of our applications use a relational database for storage. Always declare a foreign key column explicitly. Indexes
on foreign key columns can provide performance benefits for table joins involving the primary and foreign key. If you
are tasked with maintenance of an existing project which doesn’t have foreign keys, make sure to index the foreign key
columns.

## Avoid SQL inside loops
Avoid writing SQL queries inside loops as it exploits the system resources.

## Convention over configuration
If we are using a framework for development, always strive to follow the framework’s convention rather than a custom
developer configuration. We use a framework to increase our productivity. By following convention, introducing a new
developer versed in the framework can identify/relate the working with ease. Additionally, a future update of the
framework won’t hamper the application.

## HTTP error codes
Understand and use HTTP error codes. Use a HTTP-404 when a page isn’t found. It should not be used when a requested
resource(eg: user, post) is not found in the datastore. Rather, when a requested resource is missing, the browser should
be redirected to the resource listing page with an appropriate message.

## Technical debt
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Technical debt <a href="https://t.co/w0wDsAN1Mm">pic.twitter.com/w0wDsAN1Mm</a></p>&mdash; Kent Beck 🌻 (@KentBeck) <a href="https://twitter.com/KentBeck/status/1048157302350663680?ref_src=twsrc%5Etfw">October 5, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
The image in the tweet paints a very good picture of technical debt. Clean/good code means sustained development. The
quality of code today is directly related to the ease of development/extension tomorrow. Each step we take today to
write good code provides compounding benefits to the application. Besides, it benefits oneself as a developer by
improving our skill set. If the quality of the code isn’t good, future changes become perilous and complicated.

## Duplicate code
Never duplicate code. If duplicated code requires modification, there’s a danger that one section of the code is updated
without checking further for other instances. Duplicate code can be fixed by refactoring it into its own function.

## Separating code and presentation
Avoid writing HTML inside Javascript. HTML can be passed to Javascript either by using a variable or by using AJAX calls.
This processing method will provide more clarity in separating code and presentation logic. Separation of concerns
minimizes time dependency and allows for concurrent development. Moreover, if a change in design is to be carried out,
the designer needs to only refer to the HTML templates rather than the Javascript code. Also, this separation promotes
readability. It is easier to read business(code) logic that is not intermingled with presentation logic & vice-versa.

## YAGNI
Is an abbreviation for _You aren’t gonna need it_. A programmer should not add functionality until deemed necessary. XP
co-founder Ron Jeffries explains it as: Always implement things when you actually need them, never when you just foresee
that you need them. Even if you’re totally, totally, totally sure that you’ll need a feature, later on, don’t implement
it now. Usually, it’ll turn out either a) you don’t need it after all, or b) what you actually need is quite different
from what you foresaw needing earlier[3].

## KISS
Is an acronym for _Keep it simple, stupid_. The KISS principle states that most systems work best if they are kept simple
rather than made complicated; therefore, simplicity should be a key goal in design, and unnecessary complexity should
be avoided[2].

## Premature optimization is the root of all evil
<img src="https://imgs.xkcd.com/comics/the_general_problem.png" alt="Premature optimization" width="100%">
Premature optimization happens when a programmer lets performance considerations affect the design of a piece of code[2].
Developers waste enormous amounts of time thinking about the speed/memory usage of their programs. In other words,
developers often tend to optimize the code upfront before completing the functionality. Instead, the developers should
initially code the program to complete the task. Then make sure to verify the code can be maintained. And finally,
address performance[3]. Occasionally, it's acceptable to overlook decreased speed in favor of easy maintenance.

## Commits and Formatting
Code is read more often than it is written. Commits shouldn’t contain any debugging codes. Follow coding standards and
indentation style relevant to the programming language/framework. Hence formatting viz. indentation, line breaks,
splitting lengthy conditions all add value to your code.

## Communication
Communication skills are very valuable for programmers; especially due to the global nature of software projects these
days. With English being the universal language, learning functional English and being able to convey your message
without spelling mistakes, grammatical errors etc. is a must.

## Bonus: tips for developing features
Keep a To-do list. This is done to keep tabs of pending/upcoming tasks. Similarly, when reading Kent Beck’s book about
programming, he would jot down the feature requirements as To-do tasks. As a task gets coded, he would mark it as
completed. If he finds a new problem which arose while coding the task, he would add it to the To-do tasks. Picking each
task one at a time lets you complete the feature in an organised manner. I would suggest the
[Plain-tasks package](https://github.com/aziz/PlainTasks) for Sublime Text enthusiasts.

**Flawless code isn’t always feasible. Nevertheless, we should strive to upskill. All it takes is a little diligence,
creativity and adherence to best practices.**

References:
1. [Clean Code](https://www.amazon.in/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
2. [Wikipedia](https://www.wikipedia.org/)
3. [Cunningham & Cunningham](https://c2.com/)
