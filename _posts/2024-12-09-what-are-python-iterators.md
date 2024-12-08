---
title: What is an iterator in Python?
description: Iterators are python objects for traversing sequences (Sequence of elements/objects)
date: 2024-12-09 01:45:00 +0530
categories: [pythyon, programming]
tags: [python3, iterators, iterable]     # TAG names should always be in lowercase
render_with_liquid: false
---

Let's start with the answering some basic questions!

## Iterators

### 1. What is a Sequence?

A sequence is a container that holds items in an ordered manner (like `list`, `str`, `tuple` etc.). It allows indexing
and slicing using integer indices.

### 2. What is an Iterator?

An [iterator](https://docs.python.org/3/glossary.html#term-iterator) is an object representing a stream of data; this object returns the data one element at a time.  
The iterator objects themselves are required to support the following two methods, which together form the ***iterator protocol***:
+ [`__iter__()`](https://docs.python.org/dev/reference/datamodel.html#object.__iter__): Return the iterator object
  itself. This is required to allow both containers and iterators to be used with the `for` and `in` statements.
+ [`__next__()`](https://docs.python.org/dev/library/stdtypes.html#iterator.__next__): Return the next item from the
  container (one at a time). If there are no further items, raise the `StopIteration` exception.

### 3. What is an Iterable?

An [iterable](https://docs.python.org/3/glossary.html#term-iterable) is an object capable of returning its members one
at a time (*sequences* for example).   

In simple words: An iterable is an object that can be looped over and when passed to [`iter()`](https://docs.python.org/3/library/functions.html#iter) or
`__iter()__` methods it returns an iterator obejct.  

>*Iterable object must implement
[`__iter__()`](https://docs.python.org/dev/library/stdtypes.html#container.__iter__) method (which returns an iterator object that support the iterator protocol discribed above) to provide
iteration support.*
{: .prompt-info}

### 4. What are the difference between *Iterators* and *Iterables*?

>*Every iterator is an iterable but every iterable need not be an iterator*.
{: .prompt-info}

For an object to be an iterator it required to support *iterator protocol*.

For example: Python list is an iterable because it supports `__iter__()` method and applying built-in `iter()` method on a list returns an iterable object which further can be iterated over. Since this object doesn't support `__next__()` method, it is not an iterator. 

NOTE: [`iter()`](https://docs.python.org/dev/library/functions.html#iter) and
[`next()`](https://docs.python.org/dev/library/functions.html#next) are built-in functions which can be applied on the
objects which implements `__iter__()` and `__next__()` methods respectively.

Confused?
Ah! I got you :), do not worry and let's see some examples that will help you understand better.

Let's start with some a simple example:

```python
>>> nums = [1, 2, 3]          # nums is a list object and it is an iterable
>>> nums_it = iter(nums)      # nums_it is an iterator (list_iterator) object. __iter__() method is supported by list class
>>> type(nums)
<class 'list'>
>>> type(nums_it)
<class 'list_iterator'>
>>>
```
{: .nolineno }

`nums` is a sequence (`list`) object containing integer numbers. Since it supports `__iter__()` method, it is an
iterable. Calling `iter(nums)` will return an iterator. Note that, `nums` it self is not an iterator.

```python
>>> next(nums)  # or nums.__iter__() will through an error

Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: 'list' object is not an iterator
>>>
```
{: .nolineno }

but calling `next()` on `nums_it` will return elements from the iterator

```python
>>> next(nums_it)
1
>>> nums_it.__next__()
2
>>>
```
{: .nolineno }

So, if `nums` is not an iterator then why 
```python
>>> for num in nums:
        print(num)
>>>
```
{: .nolineno }
prints all the elements in the list `nums`?

Behind the scene, `for` calls `iter()` on `nums` container. This returns an iterator object that defines method
`__next__()` which accesses elements in the container one at a time. Run `dir(iter(nums))` and it will print all the methods it
defines

```python
>>> dir(nums_it)
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__length_hint__', '__lt__', '__ne__', '__new__', '__next__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setstate__', '__sizeof__', '__str__', '__subclasshook__']
>>>
```
{: .nolineno }

What happens when we do `for num in nums_it:` ?  
Well, it is a bit different than `for num in nums:` or (`for num in iter(nums):`) in the sense that `nums_it` will
exhaust once last element of the iterator will be fethced by `next(nums_it)`. Meaning if you run the below code twice
then it will print nothing for second time

```python
>>> for num in nums_it:
        print(num)
1
2
3

>>> for num in nums_it:
        print(num)
>>>
```
{: .nolineno }

Let's write a custom iterable and iterator to make it more simple!

### 5. Example of iterator and iterable class

Classes can be given iterator behaviour by implementing both methods `__iter__()` and  `__next__()` of *iterator protocol*.   
In the example below `Counter` is an iterable class which only supports `__iter__()` method and this method returns an
iterable object of either class `ForwardCounter` or `BackwarCouneter`. Both of these class implements the *iterator protocol* so
they are iterators (as well as iterable, as `__iter__()` method returns `self` object that supports `__next__()`
method).

```python
class ForwardCounter:
    """
    A forward counter class that counts from 1 up to a specified limit.

    This class implements the iterator protocol, making its objects both 
    iterators and iterables. You can use it directly in a `for` loop or 
    with `iter()` and `next()`.

    Attributes:
        limit: The maximum number up to which the counter will iterate.
        current: The current value of the counter during iteration.
    """

    def __init__(self, limit):
        """
        Initializes the ForwardCounter with an upper limit.

        Args:
            limit: The maximum number up to which the counter will iterate.
        """
        if limit < 1:
            raise ValueError("Limit must be a positive integer.")
        self.limit = limit
        self.current = 0

    def __iter__(self):
        """
        Returns the iterator object itself.

        Returns:
            ForwardCounter: The iterator instance.
        """
        return self

    def __next__(self):
        """
        Returns the next number in the counter sequence.

        Returns:
            int: The next number in the sequence.

        Raises:
            StopIteration: When the counter reaches its upper limit.
        """
        if self.current >= self.limit:
            raise StopIteration
        self.current += 1
        return self.current


class BackwardCounter:
    """
    A backward counter class that counts down from a specified number to 1.

    This class implements the iterator protocol, making its objects both 
    iterators and iterables. It can be used directly in a `for` loop or 
    with `iter` and `next`.

    Attributes:
        current: The current value of the counter during iteration.
        limit: The minimum number to stop at.
    """

    def __init__(self, start):
        """
        Initializes the BackwardCounter with the starting number.

        Args:
            start: The number to start counting down from.
        """
        if start < 1:
            raise ValueError("Start must be a positive integer.")
        self.current = start + 1
        self.limit = 1

    def __iter__(self):
        """
        Returns the iterator object itself.

        Returns:
            BackwardCounter: The iterator instance.
        """
        return self

    def __next__(self):
        """
        Returns the next number in the counter sequence.

        Raises:
            StopIteration: When the counter reaches its lower limit.
        
        Returns:
            int: The next number in the sequence.
        """
        if self.current <= self.limit:
            raise StopIteration
        self.current -= 1
        return self.current


class Counter:
    """
    A flexible counter class that provides forward or backward counting 
    based on the mode.

    This class implements the iterable protocol by defining the `__iter__` 
    method, which returns an iterator object based on the mode.

    Attributes:
        mode: The mode of counting. 0 for forward, 1 for backward.
        limit: The maximum number for counting.
    """

    def __init__(self, mode=0, limit=10):
        """
        Initializes the Counter with a mode and a limit.

        Args:
            mode: The mode of counting. 0 for forward, 1 for backward.
            limit: The maximum number for counting.
        """
        if mode not in (0, 1):
            raise ValueError("Mode must be 0 (forward) or 1 (backward).")
        if limit < 1:
            raise ValueError("Limit must be a positive integer.")
        self.mode = mode
        self.limit = limit

    def __iter__(self):
        """
        Returns an iterator object based on the mode.

        Returns:
            ForwardCounter or BackwardCounter: An iterator for the specified mode.
        """
        if self.mode == 0:
            return ForwardCounter(self.limit)
        else:
            return BackwardCounter(self.limit)

```

Now let's play around with the iterator and iterable we wrote:

```python
>>> counter = Counter(1, 5)     # creates an iterable object. Not an iterator

>>> next(counter)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: 'Counter' object is not an iterator

>>> counter_it = iter(counter)
>>> next(counter_it)
5
>>> next(counter_it)
4
>>> next(counter_it)
3
>>> next(counter_it)
2
>>> next(counter_it)
1

>>> fw_it = ForwardCounter(5)  # Iterable as well as iterator
>>> fw_it
<__main__.ForwardCounter object at 0x121afb980>

>>> for i in fw_it:     
        print(i)
1
2
3
4
5

>>>
```
{: .nolineno }

The last sample is the answer that why we need `__iter__()` method to be supported by an iterator (and why it should be
an iterable!).   
In `for i in fw_it:`, internally the `for` statement calls `iter()` on `fw_it` object, even if it is already an
iterator. 


---------

<h3>References: </h3>  

+ [iterator types](https://docs.python.org/dev/library/stdtypes.html#iterator-types)  

+ [Iterators](https://docs.python.org/3/tutorial/classes.html#iterators)  

+ [Iterators](https://docs.python.org/dev/howto/functional.html#iterators)  
