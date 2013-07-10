---
layout: post
title: "SICP Section 1.3"
date: 2013-06-27 01:06
comments: true
categories: SICP
tags: [SICP, Chapter 1, Section 1.3]
keywords: SICP
---

Procedures as Arguments
~~~~~~~~~~~~~~~~~~~~~~~

Why 'high order procedures' is useful?

.. highlights::

    Often the same programming pattern will be used with a number of different
    procedures. To express such patterns as concepts, we will need to construct
    procedures that can accept procedures as arguments or return procedures as
    values. Procedures that manipulate procedures are called high-order
    procedures, they can serve as powerful abstraction mechanisms, vastly
    increaseing the expressive power of our language.

Lambda
~~~~~~

- Using lambda to create anonymous functions:

  - **(define (plus4 x) (+ x 4))** is *equivalent to* **(define plus4 (lambda (x) (+ x 4)))**

- Using lambda(let) to create local variables

.. highlights::

    Sometimes we can use internal definitions to get the same effect as with let.

And in which cases we couldn't get the same effect?

Abstractions and first-class procedures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. highlights::

    As programmers, we should be alert to oppotunities to identify the underlying
    abstractions in our programs and to build upon then and generalize them to
    create more powerful abstractions. This is not to say one should always write
    programs in the most abstract way possible; expert programmers know how to
    choose the level of abstraction appropriate to their task. But it's important
    to be able to think in terms of these abstractions, so that we can be ready
    to apply them in new contexts.

Exercises
~~~~~~~~~

**1.29**
Answer:

This is not very hard, just simply translate the simpson rule definition to code:

.. code-block:: scheme

    (define (simpson-rule f a b n)
      ; still use 'define' since we haven't learned 'let' till now
      (define h (/ (- b a) n))
      ; the procedure to compute 'yk', the helper method used by 'term'
      (define (y k) (f (+ a (* k h))))
      ; get the 'next and 'term for the sum procedure
      (define (next k) (+ k 1))
      (define (term k)
        (* (cond ((or (= k 1) (= k n)) 1)
                 ((even? k) 2)
                 ((odd? k) 4))
           (y k)))
      (/ (* h (sum term 0 next n)) 3))

    (define (sum term a next b)
      (if (> a b)
        0
        (+ (term a)
           (sum term (next a) next b))))

    (define (integral f a b dx)
      (define (add-dx x) (+ x dx))
      (* (sum f (+ a (/ dx 2.0)) add-dx b) dx))

    (define (cube x) (* x x x))

    (simpson-rule cube 0 1 100.0)
    (simpson-rule cube 0 1 1000.0)
    (integral cube 0 1 0.01)
    (integral cube 0 1 0.001)

    ;; the output:
    ; 0.24999998999999987
    ; 0.24999999999900027
    ; 0.24998750000000042
    ; 0.249999875000001

**1.30**
Answer:

Put both the recursive and iterative versions here to compare:

.. code-block:: scheme

    (define (sum term a next b)
      (if (> a b)
        0
        (+ (term a)
           (sum term (next a) next b))))

    (define (sum-iter term a next b)
      (define (iter a result)
        (if (> a b)
            result
            (iter (next a) (+ (term a) result))))
      (iter a 0))


    (sum (lambda (x) x) 1 (lambda (x) (+ x 1)) 10)
    (sum-iter (lambda (x) x) 1 (lambda (x) (+ x 1)) 10)

    (sum (lambda (x) (* x x)) 1 (lambda (x) (+ x 1)) 5)
    (sum-iter (lambda (x) (* x x)) 1 (lambda (x) (+ x 1)) 5)

They produce the same results as expected.


**1.31**
Answer:

.. code-block:: scheme

    (define (product term a next b)
      (if (> a b)
        1
        (* (term a)
           (product term (next a) next b))))

    (define (product-iter term a next b)
      (define (iter cur result)
        (if (> cur b)
          result
          (iter (next cur) (* (term cur) result))))
      (iter a 1))

    (define (factorial n)
      (define (identity x) x)
      (product identity 1 inc n))

    (define (inc x) (+ x 1))


    (product (lambda (x) x) 1 (lambda (x) (+ x 1)) 6)
    (product-iter (lambda (x) x) 1 (lambda (x) (+ x 1)) 6)
    (factorial 6)
    (factorial 10)
    (product (lambda (x) (* x x)) 1 (lambda (x) (+ x 1)) 5)
    (product-iter (lambda (x) (* x x)) 1 (lambda (x) (+ x 1)) 5)

    ;; compute the approximations to PI using the formula:
    ;; PI   2*4*4*6*6*8...
    ;; -- = --------------
    ;; 4    3*3*5*5*7*7...

    ;; here, term is (2n/(2n + 1) * 2(n+1) /(2n + 1)), n = 1, 2, 3, ...
    (/
      (product-iter
        (lambda (n)
          (* (* 2 n) (* 2 (+ n 1.0))))
        1
        inc
        50)
      (product-iter
        (lambda (n)
          (* (+ (* 2 n) 1.0) (+ (* 2 n) 1.0)))
        1
        inc
        50)
    )

There might be better ways to translate the formula. The solution above the
is not veru accurate and if the upper bound is too large (say 100), it will
yield to 'nan+'.

**1.32**
Answer:

.. code-block:: scheme

    (define (accumulate combiner null-value term a next b)
      (if (> a b)
        null-value
        (combiner (term a)
                  (accumulate combiner null-value term (next a) next b))))

    (define (accumulate-iter combiner null-value term a next b)
      (define (iter cur result)
        (if (> cur b)
          result
          (iter (next cur) (combiner (term cur) result))))
      (iter a null-value))

    (define (sum term a next b)
      ; (accumulate + 0 term a next b))
      (accumulate-iter + 0 term a next b))

    (define (product term a next b)
      ; (accumulate * 1 term a next b))
      (accumulate-iter * 1 term a next b))


    (sum (lambda (x) x) 1 (lambda (x) (+ x 1)) 6)
    (product (lambda (x) x) 1 (lambda (x) (+ x 1)) 6)
    (sum (lambda (x) (* x x)) 1 (lambda (x) (+ x 1)) 5)
    (product (lambda (x) (* x x)) 1 (lambda (x) (+ x 1)) 5)


**1.33**
Answer:

.. code-block:: scheme

    (define (filtered-accumulate combiner null-value term a next b filter)
      (if (> a b)
        null-value
        (if (filter a)
          (combiner (term a)
                    (filtered-accumulate combiner null-value term (next a) next b filter))
          (filtered-accumulate combiner null-value term (next a) next b filter))))


    (define (prime? n)
      (= n (smallest-divisor n)))

    (define (smallest-divisor n)
      (find-divisor n 2))

    (define (find-divisor n test-divisor)
      (cond ((> (square test-divisor) n) n)
            ((divides? test-divisor n) test-divisor)
      (else (find-divisor n (+ test-divisor 1)))))

    (define (square x) (* x x))

    (define (divides? a b)
      (= (remainder b a) 0))

    (define (inc x) (+ x 1))

    (define (identity x) x)

    ; return a procedure Integer -> Boolean
    ; that check if the given integer is relative prime with n
    (define (relative-prime? n)
      (define (f a) (= (gcd n a) 1))
      f)

    (define (gcd a b)
      (if (= b 0)
        a
        (gcd b (remainder a b))))

    (filtered-accumulate + 0 square 2 inc 10 prime?)
    (filtered-accumulate * 1 identity 1 inc 10 (relative-prime? 10))


**1.34**
Answer:

Apply (f f) with get (f 2), while tring to apply (f 2), error will be issued
since a procedure is expected but an integer parameter 2 is given.

**1.35**
Answer:

.. code-block:: scheme

    (define tolerance 0.00001)

    (define (fixed-point f first-guess)
      (define (close-enough x y)
        (< (abs (- x y)) tolerance))
      (define (try guess)
        (let ((next (f guess)))
          (if (close-enough next guess)
            next
            (try next))))
      (try first-guess))


    ; since the golden-ratio x satisfies that x^2 = x + 1
    (define (golden-ratio)
      (fixed-point (lambda (x) (+ 1 (/ 1 x))) 1.0))


    (golden-ratio)


**1.36**
Answer:

.. code-block:: scheme

    (define tolerance 0.00001)

    (define (fixed-point f first-guess)
      (define (close-enough x y)
        (< (abs (- x y)) tolerance))
      (define (try guess)
        (display "current guess is: " )
        (display guess)
        (newline)
        (let ((next (f guess)))
          (if (close-enough next guess)
            next
            (try next))))
      (try first-guess))


    ; without average damping
    (fixed-point (lambda (x) (/ (log 1000) (log x))) 2.0)

    ; output below:
    ; current guess is: 2.0
    ; current guess is: 9.965784284662087
    ; current guess is: 3.004472209841214
    ; current guess is: 6.279195757507157
    ; current guess is: 3.759850702401539
    ; current guess is: 5.215843784925895
    ; current guess is: 4.182207192401397
    ; current guess is: 4.8277650983445906
    ; current guess is: 4.387593384662677
    ; current guess is: 4.671250085763899
    ; current guess is: 4.481403616895052
    ; current guess is: 4.6053657460929
    ; current guess is: 4.5230849678718865
    ; current guess is: 4.577114682047341
    ; current guess is: 4.541382480151454
    ; current guess is: 4.564903245230833
    ; current guess is: 4.549372679303342
    ; current guess is: 4.559606491913287
    ; current guess is: 4.552853875788271
    ; current guess is: 4.557305529748263
    ; current guess is: 4.554369064436181
    ; current guess is: 4.556305311532999
    ; current guess is: 4.555028263573554
    ; current guess is: 4.555870396702851
    ; current guess is: 4.555315001192079
    ; current guess is: 4.5556812635433275
    ; current guess is: 4.555439715736846
    ; current guess is: 4.555599009998291
    ; current guess is: 4.555493957531389
    ; current guess is: 4.555563237292884
    ; current guess is: 4.555517548417651
    ; current guess is: 4.555547679306398
    ; current guess is: 4.555527808516254
    ; current guess is: 4.555540912917957
    ; 4.555532270803653

    ; with average damping
    (fixed-point (lambda (x) (/ (+ x (/ (log 1000) (log x))) 2)) 2.0)

    ; output below:
    ; current guess is: 2.0
    ; current guess is: 5.9828921423310435
    ; current guess is: 4.922168721308343
    ; current guess is: 4.628224318195455
    ; current guess is: 4.568346513136242
    ; current guess is: 4.5577305909237005
    ; current guess is: 4.555909809045131
    ; current guess is: 4.555599411610624
    ; current guess is: 4.5555465521473675
    ; 4.555537551999825


**1.37**
Answer:

.. code-block:: scheme

    ; k-term finite continued fraction
    (define (cont-frac n d k)
      (define (frac i)
        (if (= i k)
          (/ (n i) (d i))
          (/ (n i) (+ (d i) (frac (+ i 1))))))
      (frac 1))

    (define (cont-frac-iter n d k)
      (define (iter i result)
        (if (= i 0)
          result
          (iter (- i 1) (/ (n i) (+ (d i) result)))))
      (iter k 0))


    ; k = 10 produce 0.6179775280898876
    (cont-frac (lambda (i) 1.0)
               (lambda (i) 1.0)
               10)

    (cont-frac-iter (lambda (i) 1.0)
                    (lambda (i) 1.0)
                    10)

    ; k = 11 produce 0.6180555555555556
    (cont-frac (lambda (i) 1.0)
               (lambda (i) 1.0)
               11)

    (cont-frac-iter (lambda (i) 1.0)
                    (lambda (i) 1.0)
                    11)


    ; TODO: implement a procedure to retry different k until the first
    ; one that produce the value accurate to 4 decimal.

**1.38**
Answer:

.. code-block:: scheme

    (define (cont-frac n d k)
      (define (iter i result)
        (if (= i 0)
          result
          (iter (- i 1) (/ (n i) (+ (d i) result)))))
      (iter k 0))


    ; if    i % 3 == 2, (d i) = 2 * ((i + 1) / 3)
    ; else  (d i) = 1
    (+ (cont-frac (lambda (i) 1.0)
                  (lambda (i) (let ((r (remainder i 3)))
                                (if (= r 2)
                                  (* 2.0 (/ (+ i 1) 3))
                                  1.0)))
                  10)
       2)

**1.39**
Answer:

.. code-block:: scheme

    ; if i = 1, (n i) = -x else (n i) = -x^2
    ; (d i) = 2 * i - 1
    (define (tan-cf x k)
      (cont-frac (lambda (i) (if (= i 1)
                               x
                               (- 0(* x x))))
                 (lambda (i) (- (* 2 i) 1))
                 k))

    (define (cont-frac n d k)
      (define (iter i result)
        (if (= i 0)
          result
          (iter (- i 1) (/ (n i) (+ (d i) result)))))
      (iter k 0))


    ; tan (pi / 8)
    (tan-cf 0.4 10)

    ; tan (pi / 4)
    (tan-cf 0.7853981633974483 10)


**1.40**
Answer:

.. code-block:: scheme

    (define (average-damp f)
      (define (average a b)
        (/ (+ a b) 2))
      (lambda (x) (average x (f x))))

    (define (square x) (* x x))

    (define tolerance 0.00001)

    (define (fixed-point f first-guess)
      (define (close-enough x y)
        (< (abs (- x y)) tolerance))
      (define (try guess)
        (let ((next (f guess)))
          (if (close-enough next guess)
            next
            (try next))))
      (try first-guess))

    ;; re-write the square-root procedure
    (define (sqrt x)
      (fixed-point (average-damp (lambda (y) (/ x y))) 1.0))

    ;; cube root, similar with above
    (define (cube-root x)
      (fixed-point (average-damp (lambda (y) (/ x (square y)))) 1.0))


    (define dx 0.00001)

    (define (deriv g)
      (lambda (x) (/ (- (g (+ x dx)) (g x)) dx)))

    (define (cube x)
      (* x x x))

    (define (newton-transform g)
      (lambda (x)
        (- x (/ (g x) ((deriv g) x)))))

    (define (newton-method g guess)
      (fixed-point (newton-transform g) guess))

    (define (cubic a b c)
      (lambda (x)
        (+ (* x x x) (* a (square x)) (* b x) c)))

    ; zero of x^3 + x^2 + x + 1
    ; should be -1
    (newton-method (cubic 1 1 1) 1.0)


**1.41**
Answer:

.. code-block:: scheme

    ;; Note: it's a little confusing at first, 'double' is not apply the function
    ;; 'f' and double the result, it's 'DOUBLE' the application of the function
    ;; 'f'.
    (define (double f)
      (lambda (x) (f (f x))))

    (define (inc x)
      (+ x 1))

    ((double inc) 1)
    (((double double) inc) 1)

    ; this will get 21
    (((double (double double)) inc) 5)


**1.42**
Answer:

.. code-block:: scheme

    (define (compose f g)
      (lambda (x) (f (g x))))

    (define (square x) (* x x))

    (define (inc x) (+ x 1))


    ((compose square inc) 6)


**1.43**
Answer:

.. code-block:: scheme

    (define (repeated f n)
      (if (= n 1)
        f
        (compose f (repeated f (- n 1)))))

    (define (compose f g)
      (lambda (x) (f (g x))))

    (define (square x) (* x x))

    (define (inc x) (+ x 1))


    ((repeated square 2) 5)


**1.44**
Answer:

.. code-block:: scheme

    (define (n-fold-smooth f n)
      (repeated (smooth f) n))

    (define (smooth f)
      (let ((dx 0.0001))
        (lambda (x)
          (/ (+ (f (- x dx)) (f x) (f (+ x dx))) 3))))

    (define (repeated f n)
      (if (= n 1)
        f
        (compose f (repeated f (- n 1)))))

    (define (compose f g)
      (lambda (x) (f (g x))))

    ((n-fold-smooth (lambda (x) (/ 1.0 x)) 3) 6)
    ((lambda (x) (/ 1.0 x)) 6)

**1.45**
Answer:

.. code-block:: scheme

    (define tolerance 0.00001)

    ;; from the experiments, the nth root requires at least log2(n) average damps
    (define (nth-root x n)
      (fixed-point
        ((repeated average-damp (floor (/ (log n) (log 2))))
        (lambda (y) (/ x (expt y (- n 1)))))
        1.0))

    (define (fixed-point f first-guess)
      (define (close-enough? x y)
        (< (abs (- (/ x y) 1)) tolerance))
      (define (try guess)
        (let ((next (f guess)))
          ; (display next)
          ; (newline)
          (if (close-enough? next guess)
            next
            (try next))))
      (try first-guess))

    (define (average-damp f)
      (define (average x y)
        (/ (+ x y) 2))
      (lambda (x) (average x (f x))))

    (define (repeated f n)
      (if (= n 1)
        f
        (compose f (repeated f (- n 1)))))

    (define (compose f g)
      (lambda (x) (f (g x))))


    (nth-root 16 4)
    (nth-root 32 5)
    (nth-root 64 6)
    (nth-root 128 7)
    (nth-root 256 8)
    (nth-root 10000000000000000000000000000000000000000000000000000000000000000 64)

**1.46**
Answer:

.. code-block:: scheme

    (define tolerance 0.00001)

    (define (iterative-improve enough? improve)
      (define (try guess)
        (let ((next-guess (improve guess)))
          (if (enough? guess next-guess)
            guess
            (try next-guess))))
      try)

    (define (fixed-point f first-guess)
      ((iterative-improve close-enough? f) first-guess))

    (define (sqrt x)
      ((iterative-improve
          close-enough?
          (lambda (y) (average y (/ x y))))
          1.0))

    (define (average x y)
      (/ (+ x y) 2))

    (define (close-enough? x y)
      (< (abs (/ (- x y) y)) tolerance))


    (sqrt 4)
    (sqrt 6)
    (sqrt 9)

    (fixed-point cos 1.0)
