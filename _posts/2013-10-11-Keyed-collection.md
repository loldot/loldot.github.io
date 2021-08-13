---
layout: post
title: Keyed Collection
author: Lorentz Vedeler
date: 2013-10-11
tags: Code
---
This week I found a cool .net-class in the `System.Collections.ObjectModel` namespace; the class `KeyedCollection<TKey,TItem>`.

It provides a collection that can be indexed by a property. If your item-class has a natural key, all you have to do is derive from the abstract KeyedCollection-class and implement the GetKeyForItem method. It behaves pretty much like a mix between a dictionary and a list. Lookups are indexed by a key you can specify, which in turn is used as the key for an internal dictionary, and so are faster than searching a regular list.

It is documented further [here][1].

```csharp
public class SchoolClass : KeyedCollection<uint, Student>
{
    protected override uint GetKeyForItem(Student newStudent) => newStudent.Number;
}
```

[1]: https://docs.microsoft.com/en-us/dotnet/api/system.collections.objectmodel.keyedcollection-2