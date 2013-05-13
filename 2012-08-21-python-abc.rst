---
layout: post
title: "Python abc"
date: 2012-08-21 21:06
comments: true
categories: Python
tags: [abc, ABCMeta, python, metaclass]
keywords: abc, ABCMeta, python, metaclass
description: Notes on reading the source of abc module
---

Introduction
------------
This module provides the infrastructure for defining
`abstract base classes <http://docs.python.org/glossary.html#term-abstract-base-class>`_
(ABCs) in Python. The ABCs define a minimal set of methods that establish the
characteristic behavior of the type. For more details about this, see
`PEP 3119 <http://www.python.org/dev/peps/pep-3119/>`_. 

Highlights
----------
The module provides a `metaclass <http://docs.python.org/glossary.html#term-metaclass>`_
used to create ABCs. An ABC can be subclassed directly. The class also has a 'register'
method to register unrelated concrete classes (including built-in classes) and unrelated
ABCs as 'virtual subclasses'

Also there are two decorators *abstractmethod* and *abstractproperty*, which will set
the function object's attribute '__isabstractmethod__' to True. Only when all of the
abstract methods and abstract properties are overriden, can a class that has a metaclass
derived from ABCMeta be instantiated.


Code comments
-------------

ABCMeta.__new__
~~~~~~~~~~~~~~~
.. code-block:: python

    def __new__(mcls, name, bases, namespace):
        cls = super(ABCMeta, mcls).__new__(mcls, name, bases, namespace)
        # Compute set of abstract method names
        abstracts = set(name
                     for name, value in namespace.items()
                     if getattr(value, "__isabstractmethod__", False))
        for base in bases:
            for name in getattr(base, "__abstractmethods__", set()):
                value = getattr(cls, name, None)
                if getattr(value, "__isabstractmethod__", False):
                    abstracts.add(name)
        cls.__abstractmethods__ = frozenset(abstracts)

        # chaoc: caches are the typical usages of weak references

        # Set up inheritance registry
        cls._abc_registry = WeakSet()
        cls._abc_cache = WeakSet()
        cls._abc_negative_cache = WeakSet()
        cls._abc_negative_cache_version = ABCMeta._abc_invalidation_counter
        return cls

- It first creates a 'type' object cls. (super(ABCMeta, mcls) is 'type')
- Iterate through all the attributes (including all the attributes inherited
  from all the bases), if any of them have '__isabstractmethod__' set to true,
  add it to cls's __abstractmethods__.
- Initialize the attributes '_abc_registry', '_abc_cache', '_abc_negative_cache'
  and '_abc_negative_cache_version', which are used to speed up the check in
  __instancecheck__ and __subclasscheck__.

ABCMeta.register(cls, subclass)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
See the added comments in line

.. code-block:: python

    def register(cls, subclass):
        """Register a virtual subclass of an ABC."""
        # chaoc: sanity check to make sure that subclass is class type
        # either a type object (for new style classes) or the type is
        # the same as ClassType(for old style classes)
        if not isinstance(subclass, (type, types.ClassType)):
            raise TypeError("Can only register classes")
        if issubclass(subclass, cls):
            return  # Already a subclass
        # Subtle: test for cycles *after* testing for "already a subclass";
        # this means we allow X.register(X) and interpret it as a no-op.
        if issubclass(cls, subclass):
            # This would create a cycle, which is bad for the algorithm below
            raise RuntimeError("Refusing to create an inheritance cycle")
        cls._abc_registry.add(subclass)
        ABCMeta._abc_invalidation_counter += 1  # Invalidate negative cache


ABCMeta.__instancecheck__
~~~~~~~~~~~~~~~~~~~~~~~~~
See the added comments in line

.. code-block:: python

    def __instancecheck__(cls, instance):
        """Override for isinstance(instance, cls)."""
        # Inline the cache checking when it's simple.
        subclass = getattr(instance, '__class__', None)
        if subclass is not None and subclass in cls._abc_cache:
            return True
        subtype = type(instance)
        # Old-style instances
        if subtype is _InstanceType:
            subtype = subclass
        # chaoc: subtype will also be subclass for old style classes
        # as assigned in the above step
        if subtype is subclass or subclass is None:
            # chaoc: check if the negative cache is safe to use or not
            if (cls._abc_negative_cache_version ==
                ABCMeta._abc_invalidation_counter and
                subtype in cls._abc_negative_cache):
                return False
            # Fall back to the subclass check.
            return cls.__subclasscheck__(subtype)
        return (cls.__subclasscheck__(subclass) or
                cls.__subclasscheck__(subtype))


ABCMeta.__subclasscheck__
~~~~~~~~~~~~~~~~~~~~~~~~~
The code and comment in this function is very clear and straightforward.

Just make sure the different cases needed to check:

1. check the subclass hook
2. check if it's a direct subclass through __mro__
3. check if it's a subclass of a registered class (issubclass is called to do
   recursive check)
4. check if it's a subclass of a subclass (issubclass is called to do recursive
   check)


In this post, we only talk about the defitions of ABCMeta. We will see the
typical usages in the **collections** module.
