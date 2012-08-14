---
layout: post
title: "Introduction on reading source of python standard libraries series"
date: 2012-08-14 21:11
comments: true
categories: Python
tags: python
---

I am going to start the series of posts on reading the source of python standard
modules. I will go with the pure python modules first, and maybe later I can
continue with C implementations of the modules. Let's see how far I could go.

What will be included
---------------------
- A brief introduction of the module. (It should be very short, people can go to
  the standard library doc for more information.)
- Special highlights about the important APIs, implementation details.
- Python features/idioms/tricks/gotchas that worth the whistle, especially those
  I was not familiar with
- Detail explanations about the tricky part of the code if any

What will not be included
-------------------------
- The example usage of the various APIs, for this kind of stuff,
  `Python module of the week <http://www.doughellmann.com/PyMOTW/>`_ is a better
  place to go


Also, alone the way, I may start another series on some specific 'advanced topics'
in python, like **descriptor**, **decorator**, **method resolution order(mro)**
and so on. Mainly about why they are introduced into python, how they are used and
the typical use cases. This is inspired by the blogs about
`python history <http://python-history.blogspot.com/>`_

This post will also be used to track my progress.



