---
layout: post
title: "QuickSort is actually two different algorithms (Lomuto and Hoare)"
date:   2018-06-05 00:00:00 +0000
tags: Computer-Science
comments: true
image:
  src: "/assets/images/face-off.jpg"
  title: "do not confuse"
---

## General observations

QuickSort is a [Divide & Conquer](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm) sorting algorithm, which immediately hints how you should start implementing it:

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
        // pi is for partition index;
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

What's left is to implement the `Partition` function which internally choses the `pivot` and moves smaller or equal (compared to `pivot`) items to the left of the returned index `pi` and greater or equal (compared to `pivot`) to the right of `pi`.

It turns out, that there are two commonly used types of such functions - the Lomuto partition an the Hoare partition, which:
* have completely different performance -- Hoare's not only considers the degenerate case of a sorted array (subarray), but also makes [~3 times less swaps](https://cs.stackexchange.com/a/11550) than Lomuto's;
* have completely different code and looping -- Hoare's moves from two ends, Lomuto's -- form one end;
* Lomuto's sets the `pivot` item to it's final place in the initial `items` array and returns its index, while as Hoare's leaves `pivot` either in the left or the right part (i.e. either at `pi` or at `pi + 1`), thus, it affects the recurring `QuickSort` function itself as it should either include or not include the item at `pi` to the next recurring calls.

## Confusion

So, are they usually referred to as two different algorithms of the same idea? Not necessarily. For example, the [QuickSort article on GeeksForGeeks](https://www.geeksforgeeks.org/quick-sort/) describes the algorithms using the Lomuto's partition -- apparently because it's code is shorter -- but the partition function isn't explicitly called this way (except for the "miscellaneous" section).

![Partition function isn't called](/assets/images/geeks-for-geeks_guicksort_lomuto-search.png  "Partition function isn't called")

The article is quite detailed and it even has mentions of possible QuickSort's modifications, e.g. "3-Way QuickSort" and "Iterative Quick Sort", but not how the classic recursive algorithm could use other partition function. And there's also [a dedicated article on the topic](https://www.geeksforgeeks.org/hoares-vs-lomuto-partition-scheme-quicksort/) -- but you have to know you need to search it first.

The confusion is also is also well-described in [the answer](https://www.reddit.com/r/compsci/comments/2tnzuc/could_someone_explain_the_clrs_quicksort/co0xfsd) by [ggchappell](https://www.reddit.com/user/ggchappell), where he notes the following:

> Fun fact: For many years, the [Wikipedia article on Quicksort](https://en.wikipedia.org/wiki/Quicksort) has had **pseudocode for the Lomuto algorithm**, and shown an **animation of the Hoare algorithm**, with no acknowledgement that these are different ...

## Algorithms' code

So, the two QuickSort algorithms respective to the two partition functions are:

### Lomuto's

```csharp
public void Sort<T>(T[] items)
    where T : IComparable<T>
{
    QuickSortLomuto(items, 0, items.Count - 1);
}

private static void QuickSortLomuto<T>(T[] items, int low, int high)
    where T : IComparable<T>
{
    if (low < high)
    {
        // if pi is partitioning index,
        // items[pi] is now at the right place
        int pi = PartitionLomuto(items, low, high);

        // the current pivot element is in place (pi),
        // thus, do not include it to further sorting

        // recursive calls, the item at pi will not be moved further
        QuickSortLomuto(items, low, pi - 1);
        QuickSortLomuto(items, pi + 1, high);
    }
}

private static int PartitionLomuto<T>(T[] items, int low, int high)
    where T : IComparable<T>
{
    T pivot = items[high];

    // index of the end of the "smaller" list
    int i = low - 1;
    // index of the last item that has being checked
    int j = low;
    for (; j < high; ++j)
    {
        // If current element items[j]
        // is smaller than or equal to pivot
        if (items[j].CompareTo(pivot) <= 0)
        {
            ++i;
            // insert smaller (for ascending) element to the end
            // of the "smaller" list - i.e. insert from j to i;
            // here i <= j always
            Swap(items, i, j);
        }
    }

    // "i end" (ie) - index after the last index of the "smaller" list
    int ie = i + 1;
    // as i <= j < high, items[high] wasn't touched;
    // therefore, here items[high] still holds the pivot value here
    Swap(items, ie, high);

    return ie;
}

private static void Swap<T>(T[] items, int i, int j)
{
    T temp = items[i];
    items[i] = items[j];
    items[j] = temp;
}
```

### Hoare's

```csharp
public void Sort<T>(T[] items)
    where T : IComparable<T>
{
    QuickSortHoare(items, 0, items.Count - 1);
}

private static void QuickSortHoare<T>(T[] items, int low, int high)
    where T : IComparable<T>
{
    if (low < high)
    {
        // if pi is partitioning index,
        // items[pi] is smaller than or equal to the current pivot
        int pi = PartitionHoare(items, low, high);

        // Here the index of the pivot is either pi or pi + 1,
        // thus could be moved later on;
        // so, include both of those to further sorting

        // recursive calls, items at pi and pi + 1 may be moved further on
        QuickSortHoare(items, low, pi);
        QuickSortHoare(items, pi + 1, high);
    }
}

private static int PartitionHoare<T>(T[] items, int low, int high)
    where T : IComparable<T>
{
    // shouldn't necessarily be the first element
    T pivot = items[low];

    // index of the leftmost (thus il) element
    // to be greater than or equal to pivot
    int il = low - 1;
    // index of  the rightmost (thus ir) element
    // to be smaller than or equal to pivot
    int ir = high + 1;

    while (true)
    {
        // find the leftmost element greater than or equal to pivot
        // by skipping all the smaller ones by increasing il
        // while  items[il] < pivot (for ascending)
        do il++;
        while (items[il].CompareTo(pivot) < 0);

        // find the rightmost element smaller than or equal to pivot
        // by skipping all the greater ones by increasing ir
        // while items[ir] > pivot (for ascending)
        do ir--;
        while (items[ir].CompareTo(pivot) > 0);

        // if two indices met, return ir, that should be either
        // the index of the pivot-value element or
        // the index of the closest lower than the pivot element here
        if (il >= ir)
            return ir;

        // else, if il and ir are not yet met, swap the respective items
        Swap(items, il, ir);
    }
}

private static void Swap<T>(T[] items, int i, int j)
{
    T temp = items[i];
    items[i] = items[j];
    items[j] = temp;
}
```

### Visual comparison

As you can see, the algorithms **almost don't have any common parts** to be refactored out except for the `Swap` and `Sort<T>` functions that have little to no logic. Even the recurring `QuickSortLomuto` and `QuickSortHoare` functions (that are to represent the Divide & Conquer nature of QuickSort) were affected by the respective partition functions (by either including or not the element at the `pi` partition index to further sorting).

## Execution time

The following unit tests measure the performance of the two algorithms:

```csharp
[TestMethod]
public void Sorter_Quick_Lomuto_Test() =>
    SorterTest(new QuickLomutoSorter());

[TestMethod]
public void Sorter_Quick_Hoare_Test() =>
    SorterTest(new QuickHoareSorter());

public void SorterTest(ISorter sorter)
{
    var largeInputPart =
        Util.CreateArrayFromEnumerables(new[] { (1, 2), (2, 1), (3, 1) });

    var scale = 400;

    Func<int[]> getLargeInput =
        () => Util.CreateArrayFromEnumerables((largeInputPart, scale));

    Func<int[]> getLargeOutput =
        () => Util.CreateArray((1, 2*scale), (2, scale), (3, scale));

    var largeSameItemsInput = Util.CreateArray((5, scale));

    var inputObjects = new[]
    {
        new {
            Input = new[] { 1, 1, 3, 2, 9, 0 },
            Output = new[] { 0, 1, 1, 2, 3, 9 }
        },
        new {
            Input = new[] { 1, 2, 3, 4, 5 },
            Output = new[] { 1, 2, 3, 4, 5 }
        },
        new {
            Input = new[] { 0 },
            Output = new[] { 0 }
        },

        // [1, 1, 2, 3,  1, 1, 2, 3,  1, 1, 2, 3,  ...]
        new {
            Input = getLargeInput(),
            Output = getLargeOutput()
        },

        // [1, 1, 1, 1, ... , 2, 2, ... , 3, 3, ...]
        new {
            Input = getLargeOutput(),
            Output = getLargeOutput()
        },

        // [5, 5, 5, 5, ... ]
        new
        {
          Input = largeSameItemsInput,
          Output = largeSameItemsInput.ToArray()
        },
    };

    foreach (var inputObject in inputObjects)
    {
        sorter.Sort(inputObject.Input);
        // asserts collection elements equality
        AssertUtil.AssertCollection(inputObject.Output, inputObject.Input);
    }
}
```

The unit test has an unsorted input, input of small sorted chunks, sorted input, and input of the same elements.

And the results are:

```
Test Name:	Sorter_Quick_Hoare_Test
Test Duration:	0:00:00.0030293

Test Name:	Sorter_Quick_Lomuto_Test
Test Duration:	0:00:00.0507912
```

Therefore, **here, Hoare's version works ~17 times faster, than Lomuto's**. The results are also look as results of two completely different algorithms.

## Subjectively

To me, the Hoare's partition function looks much more intuitive. Imagine yourself as a thimblerigger - how would you move the cups in order to do it fast?

![Thimblerigger](/assets/images/thimblerigger-cups.jpg  "Thimblerigger")

To me a sequence of exchanges of some left cup with some right cup while keeping a selected cup between them (whether it's empty or not) seems natural (and that's also how the actual "performers" do it), which is similar to Hoare's method.

[Steven-de-Rooij](https://www.quora.com/profile/Steven-de-Rooij) in his comment [here](https://www.quora.com/What-is-the-best-explanation-of-the-QuickSort-partition-algorithm/#nRpKSu) also considers Hoare's method not only faster, but also more intuitive:

> I find the Hoare scheme the easiest to understand, and it is also the most efficient.
