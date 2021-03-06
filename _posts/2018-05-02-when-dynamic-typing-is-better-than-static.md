---
layout: post
title: "When dynamic typing is better than static"
date:   2018-05-02 00:00:00 +0000
tags: Computer-Science C-Sharp JavaScript
comments: true
image:
  path: "/assets/images/static-or-dynamic.jpg"
  title: "Static or dynamic?"
---

## General observations

The "static vs. dynamic typing" confrontation is a rather long going one. More to that -- unlike many other computer science related disputes -- this one goes not only in academia, but also with the real-world<sup>TM</sup> projects too (e.g. Node.js vs. ASP.NET for back-end solutions).

The main arguments in this controversy are:
> You can utilize refactoring tools, IDE features, etc.

for static typing (e.g. `C#`); and
> It's easier to learn and to program on it.

for dynamic (e.g. `JavaScript`).

Which sound to me more like
> I'm used to it.

and
> I'm used to it.

respectively.

<!--preview-break-->

Disclaimer:
I'm obviously not talking about the cases of:
* strict performance requirements;
* few platform-specific languages.
That is, there is a choice between e.g. `C#`, `Python`, and `JavaScript`.

## IMO

I personally prefer statically-typed languages, because:
* compile-time errors are much better than runtime errors (except when those are caused by an IDE or compiler itself or not well-described);
* they are indeed faster - therefore could be applied to much more interesting tasks;
* IDEs also work much better with compile time statically typed languages -- e.g. even the best IDEs for `Python` tend to fail at variable search/rename -- which lead to the good old "search the whole project" option; in this regard static language code is much more **maintainable** -- which is a decent reason itself;
* with a dynamic language you are much likely to overlook an error on an early stage of a project; even if everything in the project if to be tested, having a possible option of failing a presentation or even a production release due to a deeply rooted bug is quite disturbing; or more tests should be written (and then also **maintained**) which is redundant spendings.

Once I heard an argument of how "thinking of a variables type in advance can be complicated".
IMO, that should not be an argument even for a beginner.
If we assume that debugging/development time ratio corresponds to _the 80/20 rule_
(i.e. debugging and code review take significantly more time than coding itself) -
then making code self-documented and expressive in advance should be welcomed even when writing a prototype -
because beginners (either with a new project, technology, or in general) spend even more time debugging.

## Static-Dynamic vs. Strong-Weak

That being said, those language characteristics should not be confused as it's well described here:
* [Weak And Strong Typing](http://wiki.c2.com/?WeakAndStrongTyping)
* [Introduction to Static and Dynamic Typing](https://www.sitepoint.com/typing-versus-dynamic-typing/)

## Objective

Despite all the above, sometimes dynamically typed languages are indeed more convenient, expressive, or easy to understand -- which leads to arguments like "but in `JavaScript` you can do ... [better]".

Therefore, I want to list such cases - that could either show how specific they are and how they could be worked around or how those are legit examples of how a dynamic language actually does better.

To make things more interesting, let's take into account that `C#` has [`dynamic`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/dynamic) and [`ExpandoObject`](https://msdn.microsoft.com/en-us/library/system.dynamic.expandoobject(v=vs.110).aspx) which lets us solve inherently dynamic tasks (like receiving objects of different structures on a single REST API endpoint and [probably] converting them to some statically typed objects with a separate logic). So, we should consider the cases where a dynamic language is better in general.

### Collection operations

When projecting, regrouping, etc. you often want to keep your names short and your variable set small.
In `JavaScript` it could be written like
```js
// assume we have groupBy implemented
items = groupBy(items, i => i.name);
otherItems = otherItems.map(oi => { id: oi.id, name: oi.name });
```
while in `C#` it should be written like
```csharp
var itemsByNames = items.GroupBy(i => i.Name);
var projectedOtherItems = otherItems.Select(oi => new { oi.Id, oi.Name });
```

If you will need initial collections after, it's fine. Otherwise it's just redundant -- especially if you have to adhere to a specific naming convention (which can lead to even more cumbersome variable names).

### Parsing

Roughly the same applies to parsing (e.g. strings to integers).

`JavaScript`:
```js
input = parseInt(input);
```
`C#`:
```csharp
var parsedInput = int.Parse(input);
```

## Conclusion

As you can see, the list of frequently occurring real-life cases when a dynamic language
and dynamic variables are more appropriate is quite short. I would really like to expand it,
so feel free to share your examples and observations.
