---
layout: post
title:  "Linq Any() performance compared to Count() > 0"
date:   2021-05-25 12:46:22 -0400
categories: dotnet
---

I often see `.Count() > 0` or `.Count > 0` to check if there are any elements in some collection. Linq provides an `Any()` method to check if there are any elements. `Any()` sure reads better but does it come with any downsides? Inspired by a twitter post on using `Any()` and benchmarking it, I decided to run some of my own tests using [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet).


The benchmarks that were run compare the use of these two methods, comparing enumerables and lists with two sizes, 1000 and 1000.

```
BenchmarkDotNet=v0.13.0, OS=Windows 10.0.19042.985 (20H2/October2020Update)
Intel Core i7-8650U CPU 1.90GHz (Kaby Lake R), 1 CPU, 8 logical and 4 physical cores
.NET SDK=5.0.203
  [Host]     : .NET 5.0.6 (5.0.621.22011), X64 RyuJIT
  DefaultJob : .NET 5.0.6 (5.0.621.22011), X64 RyuJIT
```

<table>
<thead>
<tr>
<th>Method</th>
<th>N</th>
<th align="right">Mean</th>
<th align="right">Error</th>
<th align="right">StdDev</th>
<th align="right">Median</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Enumerable_Any</strong></td>
<td><strong>1000</strong></td>
<td align="right"><strong>12.7297 ns</strong></td>
<td align="right"><strong>0.2874 ns</strong></td>
<td align="right"><strong>0.2400 ns</strong></td>
<td align="right"><strong>12.7911 ns</strong></td>
</tr>
<tr>
<td>Enumerable_CountGreaterThanZero</td>
<td>1000</td>
<td align="right">83,476.7798 ns</td>
<td align="right">1,015.1761 ns</td>
<td align="right">949.5963 ns</td>
<td align="right">83,264.1113 ns</td>
</tr>
<tr>
<td>List_Any</td>
<td>1000</td>
<td align="right">5.6846 ns</td>
<td align="right">0.1155 ns</td>
<td align="right">0.1080 ns</td>
<td align="right">5.6549 ns</td>
</tr>
<tr>
<td>List_CountGreaterThanZero</td>
<td>1000</td>
<td align="right">0.0248 ns</td>
<td align="right">0.0268 ns</td>
<td align="right">0.0358 ns</td>
<td align="right">0.0000 ns</td>
</tr>
<tr>
<td><strong>Enumerable_Any</strong></td>
<td><strong>10000</strong></td>
<td align="right"><strong>12.5423 ns</strong></td>
<td align="right"><strong>0.1560 ns</strong></td>
<td align="right"><strong>0.1382 ns</strong></td>
<td align="right"><strong>12.5044 ns</strong></td>
</tr>
<tr>
<td>Enumerable_CountGreaterThanZero</td>
<td>10000</td>
<td align="right">835,227.9053 ns</td>
<td align="right">5,795.2965 ns</td>
<td align="right">4,524.5863 ns</td>
<td align="right">834,107.7637 ns</td>
</tr>
<tr>
<td>List_Any</td>
<td>10000</td>
<td align="right">5.5781 ns</td>
<td align="right">0.1236 ns</td>
<td align="right">0.1423 ns</td>
<td align="right">5.5443 ns</td>
</tr>
<tr>
<td>List_CountGreaterThanZero</td>
<td>10000</td>
<td align="right">0.0106 ns</td>
<td align="right">0.0153 ns</td>
<td align="right">0.0143 ns</td>
<td align="right">0.0000 ns</td>
</tr>
</tbody>
</table>

## Enumerable performance

It's pretty safe to say that `Count() > 0` on an enumerable is pretty slow, but why is that? Enumerable.Count() needs to iterate over each element and count them. Let's first take a look at `Any()`.

``` cs
public static bool Any<TSource>(this IEnumerable<TSource> source)
{
    if (source == null) ThrowHelper.ThrowArgumentNullException(ExceptionArgument.source);

    if (source is ICollection<TSource> collectionoft) return collectionoft.Count != 0;
    else if (source is IIListProvider<TSource> listProv)
    {
        // Note that this check differs from the corresponding check in
        // Count (whereas otherwise this method parallels it).  If the count
        // can't be retrieved cheaply, that likely means we'd need to iterate
        // through the entire sequence in order to get the count, and in that
        // case, we'll generally be better off falling through to the logic
        // below that only enumerates at most a single element.
        int count = listProv.GetCount(onlyIfCheap: true);
        if (count >= 0) return count != 0; 
    }
    else if (source is ICollection collection) return collection.Count != 0;

    using (IEnumerator<TSource> e = source.GetEnumerator())
    {
        return e.MoveNext();
    }
}

```

Here we can see that there are a lot of conditionals to find the fastest way to determine if there are any elements. The last bit of code is what gets executed in our benchmark. There is a single iteration `e.MoveNext()` and a Boolean return. This doesn't depend on the size, it's O(1). Lets now take a look at `Count()`.

```cs
public static int Count<TSource>(this IEnumerable<TSource> source) {
    if (source == null) throw Error.ArgumentNull("source");
    ICollection<TSource> collectionoft = source as ICollection<TSource>;
    if (collectionoft != null) return collectionoft.Count;
    ICollection collection = source as ICollection;
    if (collection != null) return collection.Count;
    
    int count = 0;
    using (IEnumerator<TSource> e = source.GetEnumerator()) {
        checked {
            while (e.MoveNext()) count++;
        }
    }
    return count;
}
```

Here we see that the iteration is called for every element `while (e.MoveNext())`, a counter incremented, and the count returned. Looking back at the benchmark data and comparing Enumerable_CountGreaterThanZero with N = 1,000 and N = 10,000 the runtime is a direct function of the size, it's O(n).

So if you want to know if there are any elements in an Enumerable, use `Any()`. If you use Resharper, it will provide a hint to use `Any()` instead of `Count() > 0`

![alt text](https://gist.githubusercontent.com/Adam--/1f45499bfb256ec00b1f5fcda0cfa139/raw/14fd8ebc06eea1f96534664b367919013349aae3/resharper-enumerable-any-recommendation.jpg "Resharper recommending using Any() instead of Count() > 0")

## List performance

When it comes to lists, calling count `Count` is near instantaneous. `Any()` takes longer, however, it's still fast. Both are O(1).

The `Any()` method, is the same method that is called on the enumerable since list implements IEnumerable. Referring to the code for `Any()`, we can see that it finds the fastest way to check if there are any elements. Since `List` is an `IColleciton`, it executes `.Count != 0` for this check.

Notice that `Count` isn't a method, it's a property. In lists, it doesn't need to iterate over every element to determine how many there are of them, but instead lists keep track of how many elements there are and it returns that integer.

```cs
// Read-only property describing how many elements are in the List.
public int Count => _size;
```

The performance difference between `Any()` and `Count > 0` is the type checking overhead in `Any()`. The performance is almost negligible however and it is always advised to avoid premature optimization and to prefer legibility. So unless you are running this in a hot path, calling it very heavily, `Any()` would be preferred.

## Benchmark Source Code

The benchmark source code used for this comparison is included in it's entirety here.

LinqAnyBenchmark.csproj
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="BenchmarkDotNet" Version="0.13.0" />
  </ItemGroup>

</Project>
```

Benchmarks.cs
```cs
namespace LinqAnyBenchmark
{
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using BenchmarkDotNet.Attributes;

    [MemoryDiagnoser]

    public class Benchmark
    {
        private IEnumerable<Guid> enumerableGuids;
        private List<Guid> listGuids;

        [Params(1000, 10000)]
        public int N;

        [GlobalSetup]
        public void Setup()
        {
            enumerableGuids =
                Enumerable
                    .Range(0, N)
                    .Select(_ => Guid.NewGuid());
            listGuids =
                Enumerable
                    .Range(0, N)
                    .Select(_ => Guid.NewGuid())
                    .ToList();
        }

        [Benchmark]
        public bool Enumerable_Any() => enumerableGuids.Any();

        [Benchmark]
        public bool Enumerable_CountGreaterThanZero() => enumerableGuids.Count() > 0;

        [Benchmark]
        public bool List_Any() => listGuids.Any();

        [Benchmark]
        public bool List_CountGreaterThanZero() => listGuids.Count > 0;
    }
}
```

Program.cs
```cs
namespace LinqAnyBenchmark
{
    using BenchmarkDotNet.Running;

    class Program
    {
        static void Main(string[] args) => BenchmarkRunner.Run<Benchmark>();
    }
}
```
