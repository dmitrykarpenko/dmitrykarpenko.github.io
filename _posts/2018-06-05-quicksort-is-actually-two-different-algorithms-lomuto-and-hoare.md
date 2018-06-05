---
layout: default
title: "QuickSort is actually two different algorithms (Lomuto and Hoare)"
date:   2018-06-05 00:00:00 +0000
tags: Computer-Science
comments: true
#image:
#  src: "/assets/images/converging-parts-rainbow.jpg"
#  title: "parts to combine"
---

# {{ page.title }}

## General observations

QuickSort is a Divide & Conquer algorithm, which immediately implies how you should start writing it:

```csharp
public void QuickSort<T>(T[] items)
    where T : IComparable<T>
{
    QuickSort(items, 0, items.Length - 1);
}

private static void QuickSort<T>(T[] items, int low, int high)
    where T : IComparable<T>
{
    if (low < high)
    {
        // moves smaller or equal items to the left of the index pi
        // and greater or equal to the right
        int pi = Partition(items, low, high);

        // recursively sort the two parts;
        // that's why "Divide & Conquer"
        QuickSort(items, low, pi);
        QuickSort(items, pi + 1, high);
    }
}
```

What's left is to implement the `Partition` function that choses the `pivot` and moves smaller or equal (compared to `pivot`) items to the left of the index pi and greater or equal (compared to `pivot`) to the right.

And if turns out, that there are two types of such functions - the Lomuto partition an the Hoare partition, which:
* have completely different performance -- Hoare's considers the degenerate case of a sorted array (subarray) and makes [~3 times less swaps](https://cs.stackexchange.com/a/11550) than Lomuto's;
* have completely different code and looping -- Hoare's moves from two ends, Lomuto's -- form one end;
* Lomuto's sets the `pivot` item to it's final place in the items array and returns its index, while as Hoare's lefts it either in the left or the right part, thus, it affects the recurring `QuickSort` function itself as it should either include or not include the item at pi to the next recurring calls.
