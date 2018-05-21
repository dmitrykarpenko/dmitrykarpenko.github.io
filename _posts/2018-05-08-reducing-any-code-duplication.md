---
layout: default
title: "Reducing any code duplication"
date:   2018-05-08 00:00:00 +0000
tags: Clean-Code C-Sharp Concept
comments: true
image:
  src: "/assets/images/converging-parts-rainbow.jpg"
  title: "parts to combine"
---

# {{ page.title }}

## General observations
I'm a big fan of the ["don't repeat yourself" (DRY)](http://wiki.c2.com/?DontRepeatYourself) principle (as I've mentioned for instance [here](/2018/05/03/multiple-inheritance-of-DTOs-would-be-a-good-idea.html) a bit). IMO, this very principle should be respected from the very development start. At the same time, the principle is one of the most violated -- especially on the front-end. However, lower layer applications and libraries tend to adhere to the principle much more -- as it's much easier to "scare" product owners or managers with a prospect of maintaining and aligning two _identical_ (or even worse -- _kinda identical_) parts of the long-lasting code further on.

But some developers just tend or even like to copy-paste code and their arguments for that are:
1. it's faster [for a developer to write];
1. this way we don't affect the other (presumably) well-working code parts that we're copying from;
1. the refactoring isn't possible in this case; the other wording for this is "this is not code duplication".

IMO, not only code duplication isn't aesthetic -- it's also:
1. a [technical debt](https://en.wikipedia.org/wiki/Technical_debt);
1. can pull some implicit bugs or design flows that could lead to further two (or more) place fixing; also, when a good developer does the refactoring, the developer could also review and debug the repeating code parts once more -- therefore, make them even more reliable or even more general -- e.g. when a DB-querying method can take a lot of optional parameters (to filter the data) instead of querying the same table or join of tables with separate methods for each application.

The third pro-copy-pasting argument is a bit more tricky, as it's often applied to cases like the following:

```csharp
public void ActOriginal()
{
  // same code

  // same code

  // unique code

  if (condition)
  {
    // conditional unique code

    // same code
  }
}

public void ActSimilar()
{
  // same code

  // same code

  // other unique code

  if (condition)
  {
    // other conditional unique code

    // same code
  }
}
```

## Reducing code duplication with delegates

At first glance, `ActOriginal` and `ActSimilar` seem to really have different parts in different places. But the refactoring is still possible using delegates:

```csharp
public void ActOriginal()
{
  return Act(
    () =>
    {
      // unique code
    },
    () =>
    {
      // conditional unique code
    });
}

public void ActSimilar()
{
  return Act(
    () =>
    {
      // other unique code
    },
    () =>
    {
      // other conditional unique code
    });
}

private void Act(Action onBeforeConditionCheck, Action onConditionTrue)
{
  // same code

  // same code

  onBeforeConditionCheck();

  if (condition)
  {
    onConditionTrue();

    // same code
  }
}
```

The `onBeforeConditionCheck` and `onConditionTrue` delegate parameters could have their own input parameters to pass some `Act`'s local variables outside (e.g. `Action<string, int>`) or they could return some values (e.g. `Func<string, int, long>` -- to pass the values inside `Act`) if needed.

The delegate parameters could be made nullable to make Act even more general and C# 6 [expression-bodied function members](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-6#expression-bodied-function-members) syntax could be used:

```csharp
// new methods utilize the same Act logic:

public void ActSimple() =>
  Act();

public void ActWithOnConditionTrue() =>
  Act(
    onConditionTrue: () =>
    {
      // new conditional unique code
    });

// old methods left untouched (except for the syntax):

public void ActOriginal() =>
  Act(
    () =>
    {
      // unique code
    },
    () =>
    {
      // conditional unique code
    });

public void ActSimilar() =>
  Act(
    () =>
    {
      // other unique code
    },
    () =>
    {
      // other conditional unique code
    });

private void Act(
  Action onBeforeConditionCheck = null,
  Action onConditionTrue = null)
{
  // same code

  // same code

  onBeforeConditionCheck?.Invoke();

  if (condition)
  {
    onConditionTrue?.Invoke();

    // same code
  }
}
```

We assume that a caller shouldn't inject it's own logic with the delegates -- that's why `Act` is `private` and `ActSimple` is used to call `Act` with `null` delegates.

## Conclusion

IMO, almost any logic duplication can be refactored out -- even when different code is hiding inside methods. While the logic will not be duplicated, the code can seem to get bigger initially and some code duplication will still remain -- e.g. method calls and explicitly named parameters are technically code duplication (as you copy the names from where they're defined --  that's also why the code initially gets bigger).

But even having the code size increased, you are able to refactor added parts with automatic refactoring tools (e.g. change a delegate's or method's name later on to make it more clear); the code in general becomes more self-documented and has higher cohesion.
