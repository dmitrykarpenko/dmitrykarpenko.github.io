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

## General observations

It always surprised me how little time developers spend considering and discussing naming conventions. An explanation of why it's that way from a typical developer would probably be:
> The way we name things should be clear right from the code.

for a good project and
> It's too late to change something.

for a bad one.

The counterargument to the second one is easy -- you probably wouldn't make the project much worth -- so just make a places that you work with as comfortable and organized as possible. Once I encountered a project with almost flat structure -- it had three or four poorly filled folders and all the other code files piled in the root of the project (the code itself had similar issues). IMO, in such cases you should just apply a proper style -- at least for your specific tasks -- as arguments like "we should keep the existing convention [even if it's pointless]" or "what if a new developer does not know what a presentation layer [or any other common term] is" does not sound reasonable to me.

The first one could really be true -- at least that's our goal -- especially when a team spent some time properly setting up code analysis tools like [StyleCop](https://github.com/StyleCop/StyleCop) or even using [Code Contracts](https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/code-contracts) -- some decent standards can be achieved. But most of the times -- implementing features is more valuable and leaving opportunities for hacks is more convenient.

Everybody's probably heard of how [naming things is one of the hardest things](https://skeptics.stackexchange.com/questions/19836/has-phil-karlton-ever-said-there-are-only-two-hard-things-in-computer-science). On top of that -- while code effectively is a text, it's perceived more like a scheme.

## Code is like a scheme

Have you ever noticed how even heavily blurred code is still distinguishable?

![Blurred code](/assets/images/blurred-code-without-a-hat.jpg  "Blurred code")

You probably can guess a language -- or even what concrete snippet it is -- if you've reviewed (or written) the snippet attentively.

Same principle doesn't apply to a regular text though (except for the case when an editor or reader person knows where a paragraph of such rectangular shape is).

![Blurred text](/assets/images/blurred-book-page.jpg "Blurred text")

And that's how human brain does its optimization -- you don't have to read each and every word (or even half of them) to guess what code you're looking at.

Therefore, style aspects -- such as line breaks, braces, operator selection, etc. -- are as important for increasing perception speed as naming.
In fact, they complement each other, as you find a proper code snippet with a general look first, and then you dig into it by also reading the variable names (trying to understand how it works or why it breaks).

Such rules should also be intuitive -- as most of the times no refactoring tool can check them completely.

Here I list my personal naming and style principles along with some thoughts on why it's reasonable to apply them.

## Naming

### Make type names meaningful, name variables/fields/properties after them

That sounds a bit like [Hungarian notation](https://en.wikipedia.org/wiki/Hungarian_notation) but it's not the case -- as `Order order` (or even `Order thatVeryOrder`) is IMO much better than `string strName`.

```csharp
private class Order
{
  public string Name { get; set; }
  public long Price { get; set; }

  // tautology is OK here ...
  public Customer Customer { get; set; }
}

public void Act()
{
  // ... and here too
  Order order;

  // allows meaningful prefixes
  Order firstOrder = GetOrders(count: 1).First();l

  // implicitly typed variables are also made clear this way
  var topOrder = GetTopOrder();
}
```

Speaking of implicitly typed variables -- I tend to use it a lot. And such naming really helps to make such code self-documented -- and you shouldn't spend time hovering over a variable or returning to it's definition.

### Name collections with their respective plural forms

IMO, `orders` is much better than `orderList` (and even better than `orderEnumerable`) as:
* it's much shorter;
* it reads like a natural language;
* actual type can change later on (e.g. list to array if a read-only collection is now sufficient) and a name like `orders` would still fit;
* it can be confusing for someone --  what's better -- `orderList` or `ordersList` -- which could lead to further confusions like "should this list contain only one item or it's a convention?"

```csharp
public void DoWithCollections()
{
  IEnumerable<Order> orders = GetOrders(); // OK
  var tenFirstOrder = GetTopOrder(count: 10); // OK
}
```

### Start an explaining comment with a space, a code comment without one; comment a code-related comment along with the code

StyleCop has a default advice which is similar to "start an explaining comment with a space", but I don't like the part where they ask to artificially start comments with `////`. I like to leave `////` for a more natural case (as it's shown in the following example). Sometimes I also use `///` as it allows to use typed references -- e.g. [with a `<see cref="DTOs.OrderDto"/>` tag](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/xmldoc/see).

```csharp
public void ActWithComments()
{
  // regular code comment:
  //var pointlessItem = GetItemObsolere();

  // code comment with an explaining comment:
  /// TODO: implement <see cref="GetItems"/>
  //var items = GetItems();

  // code-related comment is commented along with the related code:
  //// temporary commented action explanation
  //TemporaryCommentedAction();
}
```

## Neverending story

I've only started writing to this list. And new conventions will probably appear -- at least because languages evolve -- and their new features can require new conventions in order to be used in concert with the previous ones.
