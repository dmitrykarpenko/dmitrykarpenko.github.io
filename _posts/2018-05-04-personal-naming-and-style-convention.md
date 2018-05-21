---
layout: default
title: "Personal naming and style convention"
date:   2018-05-04 00:00:00 +0000
tags: C-Sharp Concept
comments: true
image:
  src: "/assets/images/code-through-glasses.jpg"
  title: "it also helps to see"
---

# {{ page.title }}

## General observations

It always surprised me how little time developers spend considering and discussing naming conventions. An explanation of why it's that way from a typical developer would probably be:
> The way we name things should be clear right from the code.

for a good project and
> It's too late to change something.

for a bad one.

The counterargument to the second one is easy -- at such state you probably will not make the project much worse in terms of structure and style -- regardless of what structure and style do you use. So just make places that you work with as comfortable and organized as possible. Once I encountered a project with almost flat structure -- it had three or four poorly filled folders and all the other code files piled in the root of the project (the code itself had similar issues). IMO, in such cases you should just apply a proper style -- at least for your specific tasks -- as arguments like "we should keep the existing convention [even if it's pointless]" or "what if a new developer does not know what a presentation layer [or any other common term] is" does not sound reasonable to me.

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
  Order firstOrder = GetOrders(count: 1).First();

  // implicitly typed variables are also made clear this way
  var topOrder = GetTopOrder();
}
```

Speaking of implicitly typed variables -- I tend to use them a lot. And such naming really helps to make such code self-documented -- and you shouldn't spend time hovering over a variable or returning to it's definition.

### Name collections with their respective plural forms

IMO, `orders` is much better than `orderList` (and even better than `orderEnumerable`) as:
* it's much shorter;
* it reads like a natural language;
* actual type can change later on (e.g. list to array if a read-only collection is now sufficient) and a name like `orders` would still fit;
* it can be confusing for someone --  what's better -- `orderList` or `ordersList` -- which could lead to further confusions like "should this list contain only one item or it's a convention?"

```csharp
IEnumerable<Order> orders = GetOrders(); // OK
var tenFirstOrder = GetTopOrder(count: 10); // OK
```

### Name groupings with an initial collection's name followed by the name of it's key

At first glance such naming seems cumbersome, but it's made this way for a reason.

```csharp
var ordersByStatuses = orders.GroupBy(o => o.Status);

foreach (var ordersByStatus in ordersByStatuses)
{
  var status = ordersByStatus.Key;

  foreach (var order in ordersByStatus)
  {
    // ...
  }
}
```

In the example above, `ordersByStatuses` is effectively a collection of collections -- therefore, its name has two plural parts (`orders...` and `...Statuses`). When we iterate through it, each `ordersByStatus` element is also a collection. Note how `ordersByStatus` (the one that's of type `IGrouping<OrderStatus, Order>`) still reads plural, as it has the plural part `orders...` in it, while `...Status` is now single.

The thing is, `ordersByStatus` has two almost independent parts in it -- the `ordersByStatus.Key` key and `ordersByStatus` as an `IEnumerable<Order>` itself. Therefore, both of those parts' names should be reflected in the grouping name -- and that's done by making such names using the following template: `{collectionName}By{KeyName}`.

Thus, expressions like `var status = ordersByStatus.Key` and `var order in ordersByStatus` are self-documented and clear without mouse-hovering.

### Dictionaries

A dictionary could be named either like a collection (most of the times) or like a collection of groupings (when their keys collection is explicitly used later on).

```csharp
var currentOrders = new Dictionary<Guid, Order>(); // OK
var ordersByIds = orders.ToDictionary(o => o.Id); // also OK

// ...

var orderIds = ordersByIds.Keys; // self-documented
```

### A variable name should have as few plural endings as possible

For example:

```csharp
var orderNames = orders.Select(o => o.Name); // OK
var ordersNames = orders.Select(o => o.Name); // not OK
```

Here `orderName` should be regarded as a solid word and `ordersNames` to be it's plural form. With this approach we have:
* less naming ambiguity (e.g. between `orderNames` and `ordersNames`); also now one collection corresponds to one plural ending;
* iteration variable names like `var orderName in orderNames` or `orderNames.Select(orderName => { /* ... */ })` now emerge naturally.

### Start an explaining comment with a space, a code comment without one; comment a code-related comment along with the code

StyleCop has a default advice which is similar to "start an explaining comment with a space", but I don't like the part where they ask to artificially start comments with `////`. I like to leave `////` for a more natural case (as it's shown in the following example). Sometimes I also use `///` as it allows to use typed references -- e.g. [with the `<see cref="GetItems"/>` tag](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/xmldoc/see).

Multi-Line comment (`/* ... */`) should be used for specific cases of commenting a large code comment manually (e.g. in an unspecialized text editor where the autocomment feature isn't available) or commenting a small part inside of a code line (e.g. to temporarily comment a method's argument or to write a comment specifically pointing at some place). In the following examples such comments are also used to mark explanations (i.e. comments to comments).

```csharp
void ActWithComments()
{
  /*
    regular code comment:
  */
  //var pointlessItem = GetItemObsolere();

  /*
    code comment with an explaining comment:
  */
  /// TODO: implement <see cref="GetItems"/>
  //var items = GetItems();

  /*
    code-related comment is commented along with the related code:
  */
  //// temporary commented action explanation
  //TemporaryCommentedAction();
}

/*
  Multi-Line comment examples:
*/
void ActWithMultiLineComments(int number/* <- should rename this argument */,
  /* int redundantNumber, */ int orderNumber)
{
  // ...
}
```

### You shouldn't use shortenings or abbreviations of existing names to create a new one, except for inline lambdas

For example:

```csharp
var order = new Order(); // OK
var ord = new Order(); // shortening, not OK

var topCustomerOrders = GetCustomerOrders(customerId, count); // OK
var topOrders = GetCustomerOrders(customerId, count); // still OK if no collisions
var topOrds = GetOrders(); // shortening, not OK
var topCOs = GetOrders(); // abbreviations, not OK
var cos = GetOrders(); // abbreviations, not OK
```

In this case we'll find all related snippets when searching by a name part, e.g. `customerorders` or `orders`, case insensitive.

The exception here are inline lambdas:

```csharp
var paidCustomerOrders = customerOrders
  .Where(/* here it's OK -> */co => co.Status == OrderStatus.Paid);

var paidCustomerOrders = customerOrders
  .Where(/* still OK -> */o => o.Status == OrderStatus.Paid);
```

### Booleans (nullable booleans) should be named as a true-or-false (true-or-false-or-do-not-know) question

For example:

```csharp
bool isValid; //OK
var areAllOrdersValid = orders.All(o => o.CalculateIsValid()); // OK
bool? isOrderValid = order?.CalculateIsValid(); // OK

bool validOrder; // not OK, sounds like it's of type Order
```

### Start methods with a verb followed by a meaningful return value name

Even with very long and specific names it looks fine.

```csharp
var verySpecificOrders = GetVerySpecificOrders();
```

Method verb should also indicate whether a method returns something:
* `Get...`, `Create...`, etc. for "queries" (that return a value);
* `Do...`, `Act...`, etc. for "commands" (the void-returning ones);
* specific verbs like `Correct...` (which is both a verb and an adjective) or combination of the previous two (e.g. ActAndCreate...) for the methods that are both "commands" and "queries"; if such method is commonly used, sometimes it's reasonable to name it shorter -- e.g. Update instead of AddOrUpdate (if the method will also add a new element to a database if passed object's ID is a default value) -- but such shortcut cases are rather exceptional.

### A delegate variable should be named as if it were a method, but starting with a lowercase letter

The lambda

```csharp
Func<string, string> getFormattedName =
  originalName =>
  {
    //...
    return formattedName;
  };
```

should to the same thing as the method

```csharp
string GetFormattedName(string originalName)
{
  //...
  return formattedName;
};
```

As we often use `Action`s and `Func`s and not self-defined delegates, a lambda's name could sometimes include a name of it's variable to make it clear for the caller. For example:

```csharp
Func<string, string> getFormattedNameFromOriginalName =
  originalName =>
  {
    //...
    return formattedName;
  };
```

here it's obvious that the argument should be and should be named `originalName` and not `obj` or `arg1`.

## Formatting

### Lambda's formatting should look like method's

Lambdas

```csharp
Action actInline = () => actOther(someValue);

Action actInlineLong = () =>
  actOtherLongName(someValueWithLongName);

Action actWithBody =
  () =>
  {
    //...
  };
```

should look similar to

```csharp
void ActInline() => ActOther(_someValue);

void ActInlineLong() =>
  ActOtherLongName(_someValueWithLongName);

void ActWithBody()
{
  //...
}
```

### Move arguments to another line, starting with an additional tab

That applies to both arguments declaration and usage. Such formatting is especially important when passing lambda arguments:

```csharp
public void ProcessOrder(OrderModel order) =>
  ProcessOrderBase(order,
    doFirst: o =>
    {
      // ...
    },
    doThird: o =>
    {
      // ...
    });

public void ProcessOrderBase(OrderModel order,
  Action<OrderModel> doFirst,
  Action<OrderModel> doSecond = null,
  Action<OrderModel> doThird = null)
{
  // ...
}
```

The `doFirst` named argument here is optional, added just to make things clear. The `doThird` named argument here is mandatory, even if the first one isn't specified, as we want to call `doThird` specifically.

If the `OrderModel order` declaration would start form a new line in the definition, it should also start from a new line on a caller's side if several arguments are passed.

### Move expression-bodied function arguments to another line, starting with two additional tabs

Here, `ProcessOrderMore`'s arguments `CustomerModel customer` and `SellerModel seller` have two additional tabs in front of them in order to visually stand out from the `ProcessOrderMoreBase` call.

```csharp
public void ProcessOrderMore(OrderModel order,
    CustomerModel customer,
    SellerModel seller) =>
  ProcessOrderMoreBase(order,
    customer,
    seller,
    m =>
    {
      // ...
    });
```

## Neverending story

I've only started writing to this list. And new conventions will probably appear -- at least because languages evolve -- and their new features can require new conventions in order to be used in concert with the previous ones.
