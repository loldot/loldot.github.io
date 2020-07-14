---
layout: post
title: Keyed Collection
date: 2013-10-11
tags: Code
---
This week I found a cool .net-class in the System.Collections.ObjectModel namespace; the KeyedCollection<TKey,TItem>.

It provides a collection that can be indexed by a property. If your item-class has a natural key, all you have to do is derive from the abstract KeyedCollection-class and implement the GetKeyForItem method. It behaves pretty much like a dictionary, except it enumerates it's values rather than KeyValuePairs.

It is documented further here: https://msdn.microsoft.com/en-us/library/ms132438(v=vs.110).aspx


    public class SchoolClass : KeyedCollection<uint, Student>
    {
        protected override uint GetKeyForItem(Student newStudent)
        {
            return newStudent.Number;
        }
    }
