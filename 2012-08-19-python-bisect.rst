---
layout: post
title: "Python bisect"
date: 2012-08-19 20:01
comments: true
categories: Python
tags: [bisect, binary search, python]
keywords: bisect, python, binary search
description: Notes on reading the source of bisect module
---

Introduction
------------
This module provides support for maintaining a list in sorted order without
having to sort the list after each insertion. It uses a basic bisection
algorithm similar with the classic binary search.

Review of binary search
-----------------------
According to `'Programming Pearls'
<http://www.amazon.com/Programming-Pearls-2nd-Edition-Bentley/dp/0201657880/ref=sr_1_1?ie=UTF8&qid=1345379618&sr=8-1&keywords=programming+pearls>`_
by Jon Bentley, It's very hard to write a correct binary search(Unbelievable, Uh?).
Below is the note from the book:

.. highlights::

    I've assigned [binary search] in courses at Bell Labs and IBM.
    Professional programmers had a couple of hours to convert [its] description
    into a program in the language of their choice; a high-level pseudocode 
    was fine. At the end of the speci?ed time, almost all the programmers
    reported that they had correct code for the task. We would then take thirty
    minutes to examine their code, which the programmers did with test cases.
    In several classes and with over a hundred programmers, the results varied
    little: ninety percent of the programmers found bugs in their programs (and
    I wasn't always convinced of the correctness of the code in which no bugs were
    found). I was amazed: given ample time, only about ten percent of professional
    programmers were able to get this small program right. But they aren't the only
    ones to ?nd this task difficult: in the history in Section 6.2.1 of his
    'Sorting and Searching', Knuth points out that while the first binary search
    was published in 1946, the first published binary search without bugs did not
    appear until 1962.
	
Understanding how to use loop invariants in composing a program is very important,
so let's first implement a binary search and prove the correctness utilizing this.
Here we use a simplified specification compared to the bsearch function in the C
standard library.

.. code-block:: cpp

    int binsearch(int x, int* A, int n)
    // @require 0 <= n && n <= length(A)
    // @require is_sorted(A, n)
    /* @ensures (-1 == result && !is_in(x, A, n))
                || ((0 <= result && result < n) && A[result] == x)
    */
    {
        int lower = 0, higher = n;
        int mid;
        
        while (lower < higher)
        // @loop_invariant 0 <= lower && lower <= higher && higher <= n
        // @loop_invariant (lower == 0 || A[lower - 1] < x)
        // @loop_invariant (higher == n || A[higher] > x)
        {
            mid = lower + (higher - lower) / 2;
            // @assert lower <= mid && mid < higher
            if (A[mid] == x)
                return mid;
            else if (A[mid] < x)
                lower = mid + 1;
            else
                higher = mid;
        }
        return -1;
    }


It's easy to prove that the loop invariants are strong enough to imple the
postcodition of the function(@ensures). Does this function terminate? In the
loop body, we have lower < higher, so lower <= mid && mid < higher, then the
intervals from *lower* to *mid* and from *(mid+1)* to *higher* are strictly
smaller than the original interval (lower, higher), unless we find the element,
the difference between higher and lower will eventually become 0 and we will
exit the loop.

**The bisect() functions provided in this module are useful for finding the
insertion points but can be tricky or awkward to use for common searching
tasks.**

Code comments
-------------
The above binsearch function will return the first of the elements that equals
to the target if there are more than one. Sometimes it's required to find the
left most one or the right most one. We can achieve this by using the bisect()
functions. Let's examine the pre and post conditions, and also loop invariants.

.. code-block:: python

    def bisect_right(a, x, lo=0, hi=None):

        # Let l be the original lo passed in, and h be hi. In order to differentiate.
        
        # @requires 0 <= l && l < h && h <= len(a)
        # @requires is_sorted(a[l:h])
        # @ensures all e in a[l:result] have e <= x, and all e in a[result:h] have e > x
        
        while lo < hi:
            # @loop_invariant l <= lo && lo <= hi && hi <= h
            # @loop_invariant lo == l || a[lo - 1] <= x
            # @loop_invariant hi == h || a[hi] > x
            mid = (lo + hi) // 2
            # @assert lower <= mid && mid < higher
            if x < a[mid]:
                hi = mid
            else:
                lo = mid + 1
        return lo
	
After exiting the loop, we have lo == hi. Now we have to distinguish some cases:

- If lo == l, from the third conjunct, we know that a[hi] > x, and since lo == hi,
  we have a[lo] > x. In this case, all e in a[l:h] > x
- If hi == h, from the second conjunct, we know that a[lo - 1] < x, in this case,
  all e in a[l:h] <= x
- if lo != l and hi != h, from the second and third conjuncts, we know that
  a[lo - 1] <= x, a[hi] > x. The post condition still holds.
  

We can do the same analysis on the function bisect_left(). 


Note: In the pre condition I explicitly written down that lo < hi is required, the
code will directly return lo when hi is less than or equal to lo, but that's simply
meaningless, if this is the case, the sorted order of 'a' after we insert element
before the index returned by the function.

About the function prototype
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In the doc of the module, it's **bisect.bisect_right(a, x, lo=0, hi=len(a))**
while in the source code, it's **def bisect_left(a, x, lo=0, hi=None)**

**bisect.bisect_right(a, x, lo=0, hi=len(a))** is not valid python code, you will
get error like this: **NameError: name 'a' is not defined.** This is because default
values are computed and bound at the function definition time rather than when you
call the function. This means that you can't have a default which is dependent on
something that is not known until the function is called.

Pythonic stuff
--------------
Override function definitions in Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Overwrite above definitions with a fast C implementation
    try:
        from _bisect import *
    except ImportError:
        pass

Gotcha -- Mutable default arguments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See `here <http://stackoverflow.com/questions/1132941/least-astonishment-in-python-the-mutable-default-argument>`_
for more details.
