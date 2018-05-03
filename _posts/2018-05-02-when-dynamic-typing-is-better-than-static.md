---
layout: default
title: "When dynamic typing is better than static"
date:   2018-05-02 00:00:00 +0300
---

# {{ page.title }}

The main arguments in this controversy are:
> You can utilize refactoring tools, IDE features, etc.

for static typing; and
> It's easier to learn and to program on it.

for dynamic.

Which sound to me more like
> I'm used to it.

and
> I'm used to it.

respectively.

```
Disclaimer:
I'm obviously not talking about the cases of:
 - strict performance requirements;
 - few platform-specific languages.
That is, there is a choice between e.g. C#, Python, and JavaScript.
```

I personally prefer statically-typed languages, because:
* compile-time errors are much better than runtime errors
  (except when those are caused by an IDE or compiler itself or not well-described);
* they are indeed faster - therefore could be applied to much more interesting tasks;
* IDEs also work much better with compile time statically typed languages -
  e.g. even the best IDEs for Python tend to fail at variable search/rename.

Once I heard an argument of how "thinking of a variables type in advance can be complicated".
IMO, that should not be an argument even for a beginner.
If we assume that debugging/development time ratio corresponds to the 80/20 rule
(i.e. debugging and code review take significantly more time than coding itself) -
then making code self-documented and expressive in advance should be welcomed even when writing a prototype -
because beginners (either with a new project, technology, or in general) spend even more time debugging.

But sometimes dynamically typed languages are indeed more convenient, expressive, or easy to understand -
which leads to arguments like "but in JavaScript you can do ... [better]".

Therefore, I want to list such cases - that could either show how specific they are
and how they could be worked around or how those are legit examples of how a dynamic language actually does better.

## Collection operations

When projecting, regrouping, etc. you often want to keep your names short and your variable set small.
In JavaScript it could be written like
```js
// assume we have groupBy implemented
items = groupBy(items, i => i.name);
otherItems = otherItems.map(oi => { id: oi.id, name: oi.name });
```
while in C# it should be written like
```csharp
var itemsByNames = items.GroupBy(i => i.Name);
var projectedOtherItems = otherItems.Select(oi => new { oi.Id, oi.Name });
```

If you will need initial collections after, it's fine. Otherwise it's just redundant -
especially if you have to adhere to a specific naming convention (which can lead to even more cumbersome variable names).
