---
layout: default
title: "Personal naming and style convention"
date:   2018-05-04 00:00:00 +0000
tags: C-Sharp Concept
image:
  src: "/assets/images/code-through-glasses.jpg"
  title: "also helps to see"
---

# {{ page.title }}

It always surprised me how little time developers spend considering and discussing naming conventions. An explanation of why it's that way from a typical developer would probably be:
> The way we name things should be clear right from the code.

for a good project and
> It's too late to change something.

for a bad one.

The counterargument to the second one is easy -- you probably wouldn't make the project much worth -- so just make a places that you work with as comfortable and organized as possible. Once I encountered a project with almost flat structure -- it had three or four poorly filled folders and all the other code files piled in the root of the project (the code itself had similar issues). IMO, in such cases you should just apply a proper style -- at least for your specific tasks -- as arguments like "we should keep the existing convention [even if it's pointless]" or "what if a new developer does not know what a presentation layer [or any other common term] is" does not sound reasonable to me.

The first one could really be true -- at least that's our goal -- especially when a team spent some time properly setting up code analysis tools like [StyleCop](https://github.com/StyleCop/StyleCop) or even using [Code Contracts](https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/code-contracts) -- some decent standards can be achieved. But most of the times -- implementing features is more valuable and leaving opportunities for hacks is more convenient.

Everybody's probably heard of how [naming things is one of the hardest things](https://skeptics.stackexchange.com/questions/19836/has-phil-karlton-ever-said-there-are-only-two-hard-things-in-computer-science). On top of that -- while code effectively is a text, it's perceived more like a scheme.

Have you ever noticed how even heavily blurred code is still distinguishable?

![Blurred code](/assets/images/blurred-code-without-a-hat.jpg  "Blurred code")

You probably can guess a language -- or even what concrete snippet it is -- if you've reviewed (or written) the snippet attentively.

The same thing is not easy to do with a regular text (except for the case when an editor or reader knows where a paragraph of such rectangular shape is).

![Blurred text](/assets/images/blurred-book-page.jpg "Blurred text")

And that's how human brain does its optimization -- you don't have to read each and every word (or even half of them) to guess what code you're looking at.

Therefore, style aspects -- such as line breaks, braces, operator selection, etc. -- are as important for increasing perception speed as naming.
In fact, they complement each other, as you find a proper code snippet with a general look first, and then you dig into it by also reading the variable names (trying to understand how it works or why it breaks).

Such rules should also be intuitive -- as most of the times no refactoring tool can check them completely.

Here I list my personal naming and style principles along with some thoughts on why it's reasonable to apply them.
