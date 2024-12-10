---
title: What is an iterator in Python?
description: Iterators are python objects for traversing sequences (Sequence of elements/objects)
date: 2024-12-09 01:45:00 +0530
categories: [pythyon, programming]
tags: [python3, iterators, iterable]     # TAG names should always be in lowercase
render_with_liquid: false
---

## Understanding Python Iterators: A Beginner’s Guide
In Python, iterators and iterables form the backbone of iteration. If you've ever used a `for` loop, you've benefited
from these powerful concepts. This blog will unravel the mystery behind them, setting the stage for understanding
generators, which we’ll explore in the next blog.

Let's start with answering few basic questions!

### 1. What is a Sequence?

A sequence is a container that holds items in an ordered manner (like `list`, `str`, `tuple` etc.). It allows indexing
and slicing using integer indices.
Examples:
+ `nums = [1, 2, 3]`
+ `message = "Hello, World!"`
+ `coordinate = (5, 10)`

### 2. What is an Iterator?

An [iterator](https://docs.python.org/3/glossary.html#term-iterator) is an object representing a stream of data, yeilding one element at a time.  
The iterator objects themselves are required to support the following two methods, which together form the ***iterator protocol***:
+ [`__iter__()`](https://docs.python.org/dev/reference/datamodel.html#object.__iter__): Return the iterator object
  itself. This is required to allow both containers and iterators to be used with the `for` and `in` statements.
+ [`__next__()`](https://docs.python.org/dev/library/stdtypes.html#iterator.__next__): Return the next item from the
  container (one at a time). If there are no further items, raise the `StopIteration` exception.

### 3. What is an Iterable?

An [iterable](https://docs.python.org/3/glossary.html#term-iterable) is an object that produce an iterator when
passed to [`iter()`](https://docs.python.org/3/library/functions.html#iter) method. Examples of iterables include sequences like `list`, `string`, and `tuple`.   

>*Iterable object must implement
[`__iter__()`](https://docs.python.org/dev/library/stdtypes.html#container.__iter__) method which returns an iterator
object (that must support the iterator protocol discribed above).*
{: .prompt-info}

### 4. What are the difference between *Iterators* and *Iterables*?

>*Every iterator is an iterable but every iterable need not be an iterator*.
{: .prompt-info}

For an object to be an iterator it required to support *iterator protocol*.

For example: Python list is an iterable because it supports `__iter__()` method and applying built-in `iter()` method on a list returns an iterable object which further can be iterated over. Since this object doesn't support `__next__()` method, it is not an iterator. 

NOTE: [`iter()`](https://docs.python.org/dev/library/functions.html#iter) and
[`next()`](https://docs.python.org/dev/library/functions.html#next) are built-in functions which can be applied on the
objects which implements `__iter__()` and `__next__()` methods respectively.

> + *Iterators must implement both `__iter__()` and `__next__()`.*  
+ *Iterables implement only `__iter__()`, which returns an iterator.*
{: .prompt-info}

Let's go through some examples:

```python
nums = [1, 2, 3]          # nums is an iterable
nums_it = iter(nums)      # nums_it is an iterator 

print(type(nums))     # Outputs: <class 'list'>
print(type(nums_it))  # Outputs: <class 'list_iterator'>
```
{: .nolineno }

`nums` is a sequence (`list`) and it supports `__iter__()` method, it is an
iterable. Calling `iter(nums)` will return an iterator. Note that, `nums` it self is not an iterator.

```python
>>> print(next(nums))  # or nums.__iter__() will through an error

Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: 'list' object is not an iterator
```
{: .nolineno }

but calling `next()` on `nums_it` will return elements from the iterator

```python
>>> next(nums_it)  # or nums_it.__next__()
1
>>> next(nums_it)
2
>>> next(nums_it)
3
>>> next(nums_it)  # No further item left in the container
Traceback (most recent call last):
  File "<input>", line 1, in <module>
StopIteration
>>>
```
{: .nolineno }

*So, if `nums` is not an iterator then how the following snippet loop over all the elements in `nums`?*
```python
for num in nums:
    print(num)
```
{: .nolineno }


Behind the scene, python: 
1. calls `iter(nums)` to get an iterator. 
2. calls `next()` repeatedly on the iterator returned in step 1 to fetch elements until `StopIteration` is raised.


>Run `print(dir(nums_it))` and it will print all the methods it
defines
```python
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__length_hint__', '__lt__', '__ne__', '__new__', '__next__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setstate__', '__sizeof__', '__str__', '__subclasshook__']
```
{: .prompt-tip}
{: .nolineno }

*What happens when we do `for num in nums_it:` ?*  
 
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

>*Iterators are exhausted after one pass, unlike iterables which can be re-iterated by creating a new iterator*
{: .prompt-info}


### 5. Example of iterator and iterable class (Custom iterator and iterable)

Classes can exibit iterator behaviour by implementing both the `__iter__()` and  `__next__()` method of *iterator
protocol*. 

In the example below `ForwardCounter` and `BackwarCouneter` implement the *iterator protocol*, making them both iterators as
well as iterables.  
On the other hand, `Counter` is an iterable class which only supports `__iter__()` method. This method returns an
iterator object of either class `ForwardCounter` or `BackwarCouneter`.

Forward Counter:

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

# Example run:
fw_counter = ForwardCounter(5)

print(next(fw_counter))  # Outputs: 1

for num in fw_counter:
    print(num)  # Outputs: 2 3 4 5
```

Backward Counter:

```python
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


# Example run:
bw_counter = BackwardCounter(5)

print(next(bw_counter))  # Outputs: 5

for num in bw_counter:
    print(num)           # Outputs: 4 3 2 1
```

Counter:

```python
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


# Example run:
counter = Counter(1, 5) 

for num in counter:
    print(num)        # Outputs: 5 4 3 2 1


# print(next(counter))        # Will produce TypeError

counter_it = iter(counter)
print(next(counter_it))   # Outputs: 5
```

### 6. Why is `__iter__()` Important for Iterators? 

The example 

```python
for i in fw_it:     
    print(i)
```
{: .nolineno }

demonstrates why the `__iter__()` method is essential for an iterator (and why an iterator should also be iterable).

As already explained above, when you use a statement like `for i in fw_it`:, Python internally calls the `iter()` function on the `fw_it` object. Even if `fw_it` is already an iterator, the `iter()` function ensures compatibility with the iterable protocol by calling the object's `__iter__()` method.

If the `__iter__()` method is missing, the `fw_it` object will not be recognized as an iterable, and the `for` loop will raise a `TypeError`. This is because `for` loops require objects to support the iterable protocol, even for iterators.

To see this in action, try removing the `__iter__()` method from fw_counter and then running `for i in fw_it`:. You’ll encounter an error.

---------

<h3>References: </h3>  

+ [iterator types](https://docs.python.org/dev/library/stdtypes.html#iterator-types)  

+ [Iterators](https://docs.python.org/3/tutorial/classes.html#iterators)  

+ [Iterators](https://docs.python.org/dev/howto/functional.html#iterators)  
