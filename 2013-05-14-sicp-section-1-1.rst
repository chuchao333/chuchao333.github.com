---
layout: post
title: "SICP Section 1.1"
date: 2013-05-14 00:39
comments: true
categories: SICP
tags: [SICP, Chapter 1, Section 1.1] 
keywords: SICP
---

.. highlights::

    Computational processes are abstract beings that inhabit computers. As they
    evolve, processes manipulate other abstract things called data. The evolution
    of a process is directed by pattern of rules called a program.


Why use Lisp for the book?

.. highlights::

    The most significant unique feature of Lisp is the fact that Lisp descriptions
    of processes, called procedures, can themselves be represented and manipulated
    as Lisp data. The importance of this is that there are powerful program-design
    techniques that rely on the ability to blur the traditional distinction between
    "passive" data and "active" processes.

The three mechanisms of combining simple ideas to form more complex ideas in any
powerful programming languages:

1. primitive expressions
#. means of combination
#. means of abstraction


Substitution model
~~~~~~~~~~~~~~~~~~

- Applicative order

  Evaluate the operator and operands first and then applies the resulting procedure
  to the resulting arguments. "Fully expand and then reduce"

- Normal order

  Don't evaluate the operands until their values are needed, instead, substitute
  operand expressions for parameters until it obtained an expression involving only
  primitive operators, and would then perform the evaluation. "Evaluate the
  arguments and then apply"

Example: Square Roots by Newton's Method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. highlights::

    The contrast between function and procedure is a reflection of the general
    distinction between describing properties of things and describing how to
    do things, or, as it is sometimes refered to, the distinction between
    declarative knowledge and imperative knowledge. In mathematics we are
    usually concerned with declarative (what is) descriptions, whereas in
    computer science we are usually concerned with imperative (how to)
    descriptions.

Exercises
~~~~~~~~~

**1.3** Define a procedure that takes three numbers as arguments and return the
sum of squares of the two larger numbers.

.. code-block:: scheme
    
    (define (square x) (* x x))

    (define (sum-of-squares a b) (+ (square a) (square b)))

    (define (min a b)
      (if (< a b)
         a
         b))

    (define (max a b)
      (if (< a b)
         b
         a))

    (define (sum-of-squares-of-2-largest a b c)
      (sum-of-squares (max a b) (max c (min a b))))

**1.4** Observe that our model of evaluation allows for combinations whose
operators are compound expressions. Use this observation to describe the
behavior of the following procedure:

.. code-block:: scheme

    (define (a-plus-abs-b a b)
      ((if (> b 0) + -) a b))

if b is greater than 0, the operator that will apply to a and b is +, or else
it will be -, so the a-plus-abs-b will always result in a plus abs of b.

**1.5**
Answer: the interpreter that uses applicative order evaluation will hang due
to the infinite recursive call of 'p', while an interpreter that uses normal
order evaluation will get 0.

*Note:* **p** is a function, **(p)** is a call to function p.

**1.6**
Answer: the interpreter will hang due to the infinite recursive call to
*sqrt-iter*. Since List uses applicative order evaluation, in the definition
of new-if, the else-clause will always be evaluated no matter the result of
the predicate, thus lead to infinite recursive call to sqrt-iter. That's why
**if** needs to be a special form, the predicate expression is evaluated first,
and the result determines whether to evaluate the consequent or the alternative
expression.

**1.7**
Answer: For small values, the absolute tolerance 0.001 is too large, so the
results become inaccurate. For example, (sqrt 0.001) gives 0.04124542607499115
on my machine. (ubuntu 12.10 x86_64); And for large values, due to the precision
limitation of float-point representation, the guess couldn't be refined to a
value that can be represented within the tolerance. In such cases, the program
can endlessly alternate between two guesses that are more than 0.001 away from
the true square root.

so, instead of using absolute tolerance, we changed to use the relative
tolerance of two continuous guess values. This can be demonstrated with the
below updated good-enough? procedure:

.. code-block:: scheme

    (define (good-enough? guess x)
      (< (abs (- (improve guess x) guess)) 0.001))

**1.8**

.. code-block:: scheme

    (define (cbrt-iter guess x)
      (if (good-enough? guess x)
        guess
        (cbrt-iter (improve guess x) x)))

    (define (improve guess x)
      (/ (+ (/ x (square guess)) (* 2 guess)) 3))

    (define (good-enough? guess x)
      (< (abs (- (improve guess x) guess)) 0.001))

    (define (square x)
      (* x x))

    (define (cbrt x)
      (cbrt-iter 1.0 x))

The only difference between cbrt and sqrt is the **improve** procedure.
