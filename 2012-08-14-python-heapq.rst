---
layout: post
title: "Python heapq"
date: 2012-08-14 22:26
comments: true
categories: Python
tags: [python, heap, priority queue]
keywords: python, heapq, priority queue
description: Notes on reading the source of heapq
---

Introduction
------------
The module provides an implementation of heap queue algorithm, also known as
priority queue algorithm.

Highlights
----------
- Zero-based indexing is used, so the children's index of node with index k
  are (2*k + 1) and (2*k + 2) respectively.
- Internally a 'min heap' is maintained rather than 'max heap', which is more
  generally used in algorithm textbooks.
- Three general functions based on heaps are also provided:

.. code-block:: python

    # Merge multiple sorted inputs into a single sorted output
    # (for example, merge timestamped entries from multiple log files)
    heapq.merge(*iterables)

    # The following two functions are effectively like
    # sorted(iterable, key=key, reverse=True)[:n] and
    # sorted(iterable, key=key)[:n],
    # but they perform best with smaller values of n
    heapq.nlargest(n, iterable[, key])
    heapq.nsmallest(n, iterable[, key]))


Pythonic stuff
--------------

Rich comparison methods
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    def cmp_lt(x, y):
        # use __lt__ if available; otherwise, try __le__
        return x < y if hasattr(x, '__lt__') else (not y <= x)

In python, there are 6 so called "Rich Comparison" methods, x < y calls
x.__lt__(y) and others are similar (__le__ and <=; __gt__ and >=; __eq__ and ==;
__ne__ and <>). Arguments to rich comparison  methods are never coerced. see
`coercion <http://docs.python.org/glossary.html#term-coercion>`_

Code comments
-------------
Why heapify(x) is O(n)?
~~~~~~~~~~~~~~~~~~~~~~~
This is not obvious at first by seeing the code, given that there is a 'while'
loop in _siftup and also a while loop in _siftdown(called in _siftup). Let's
look into it further:

1) in the while loop of _siftup, it takes O(L) time for nodes that L levels
   above leaves.
2) and in the while loop of _siftdown called in _siftup, it takes at most L
   steps, so _siftdown is O(L).
3) since we have n/4 nodes in level 1, n/8 nodes in level 2, and finally one
   root node, which is lg(n) levels above leaf, so the total amount in the while
   loop of heapify is:

| **n/4 * c + n/8 * c + n/16 * 3c + ... + 1 * lg(n) * c**, and let n/4 = 2^k,
  after simplification, we get:
| **c * 2^k(1/2^0 + 2/2^1 + 3/2^2 + ... + (k+1)/2^k)**, as the limit of
  **(k+1)/2^k** is 0 when k is infinite, so the term in the brackets bound to
  a constant, from this we can conclude that heapify is O(2^k), which is O(n).
|

Why it continues to find the smaller child until a leaf is hit in _siftup?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
As explained in the comment by the module author, this is a ad hoc to reduce
the comparisons on the following operations on the heap.
