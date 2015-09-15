###Ex 2.1
```scheme
(define (make-rat n d)
  (let ((g (gcd n d)))
    (if (< (/ n d) 0)
	(cons (- (abs n)) (abs d))
	(cons (abs n) (abs d)))))
```

We check if `n` divided by `d` is negative.  If it is, then by convention, we give `n` the negative sign, and take the absolute value of `d`.  Note that in this case we take the absolute value of `n` before negating it in case it's already negative.  If the result is postive, then again, we take the absolute value of each in case they're both negative.

###Ex 2.2

```scheme
(define (make-segment a b)
  (cons a b))

(define (start-segment s)
  (car s))

(define (end-segment s)
  (cdr s))

(define (make-point x y)
  (cons x y))

(define (x-point p)
  (car p))

(define (y-point p)
  (cdr p))

(define (midpoint-segment s)
  (let ((x-mid (average (x-point (start-segment s))
			(x-point (end-segment s))))
	(y-mid (average (y-point (start-segment s))
			(y-point (end-segment s)))))
    (make-point x-mid y-mid)))
```

###Ex 2.3
To ensure that our `perimeter` procedure will work with any representation w choose to use for a rectangle, we can begin by assuming that we have procedures `width-rect`, and `height-rect` that will tell us the width and height of the rectangle.

- `(width-rect <r>)`

  returns the width of the given rectangle `r`.

- `(height-rect <r>)`

  returns the height of the given rectangle `r`.

At this point, this is all we need to find the perimeter and area of the rectangle `<r>`:

```scheme
(define (perimeter-rect r)
  (+ (* 2 (width-rect2 r))
     (* 2 (height-rect2 r))))

(define (area-rect r)
  (* (width-rect2 r)
     (height-rect2 r)))

```

Since the definitions above make no assumptions about the representation fo the rectangle, they will work for any representation that we choose.

Here is one way, representing a rectangle $$ABCD$$ as four points:

```scheme
(define (make-rect a b c d)
  (cons (cons a b) (cons c d)))
```

We define the selectors for the four points:

```scheme
(define (a-rect r)
  (car (car r)))

(define (b-rect r)
  (cdr (car r)))

(define (c-rect r)
  (car (cdr r)))

(define (d-rect r)
  (cdr (cdr r)))
```

Finally, we can define `width-rect` and `height-rect` in terms of the above procedures:

```scheme
(define (height-rect r)
  (let ((d (d-rect r))
	(a (a-rect r)))
    (- (y-point d)
       (y-point a))))

(define (width-rect r)
  (let ((b (b-rect r))
	(a (a-rect r)))
    (- (x-point b)
       (x-point a))))
```

An alternative representation is to represent the rectangle as a point (its bottom left corner) and the two numbers to represent its width and height:

```scheme
(define (make-rect o w h)
  (cons o (make-point w h)))
```

then `width-rect` and `height-rect` are very straightforward:

```scheme
(define (height-rect r)
  (cdr (cdr r)))

(define (width-rect r)
  (car (cdr r)))
```

###Ex 2.4
The given example of `car` applies `z`, the procedure returned by `cons` to another `lambda` that simply returns its first parameter.  For `cdr`, we simply apply `z` to a procedure that returns the second formal parameter:

```scheme
(define (cdr z)
  (z (lambda (p q) q)))
```

###Ex 2.5
For nonnegative integers $$a$$ and $$b$$, the prime factorization of the product $$2^a \cdot 3^b$$ consists only of $$2$$'s and $$3$$'s.  Also, since both $$2$$ and $$3$$ are prime, any $$2$$ resulting in the prime factorization of $$2^a3^b$$ will belong to $$2^a$$ and likewise for $$3$$ and $$3^b$$.

For `cons`, we simply return the resulting product:

```scheme
(define (cons a b)
  (* (expt 2 a)
     (expt 3 b)))
```

For `car`, we find $$a$$ by repeatedly diving by $$2$$ as long as $$z$$ is divisible by $$2$$:

```scheme
(define (car z)
  (define (iter a z)
    (if (= (remainder z 2) 0)
	(iter (+ 1 a) (/ z 2))
	a))
  (iter 0 z))
```

`cdr` is the same, but with repeated division by $$3$$:

```scheme
(define (cdr z)
  (define (iter b z)
    (if (= (remainder z 3) 0)
	(iter (+ 1 b) (/ z 3))
	b))
  (iter 0 z))
```

###Ex 2.6
A Church numeral is represented by the number of times a function is applied:

```scheme
(define one (lambda (f) (lambda (x) (f x))))

(define two (lambda (f) (lambda (x) (f (f x)))))
```

To add two church numerals together, we simply apply the first numeral to the given argument, then apply the second to the result of the first application:

```scheme
(define (lambda+ a b)
  (lambda (f) (lambda (x) ((b f) ((a f) x)))))
```

###Ex 2.7

```scheme
(define (upper-bound x) (cdr x))
(define (lower-bound x) (car x))
```

###Ex 2.8

For two intervals $$x$$ and $$y$$ and their difference $$x-y$$, there are a few possibilities.  Clearly, for the lower bound, from the lower bound of $$x$$, we could subtract either the lower or upper-bound of $$y$$.  Since we're looking for the smallest possible value, we would want to subtract the upper bound of $$y$$ from lower bound of $$x$$.  Likewise, we want to subtract the smallest amount (lower-bound of $$y$$) from the upper bound $$x$$ for the upper-bound of the difference.

```scheme
(define (sub-interval x y)
  (make-interval (- (lower-bound x) (upper-bound y))
		 (- (upper-bound x) (lower-bound y))))
```

###Ex 2.9

For intervals $$a$$ and $$b$$, their difference is

\begin{equation}
a - b = [a_l - b_h, a_h - b_l]
\end{equation}

The width of $$a-b$$ is

\begin{equation}
  \end{aligned}
    \frac{(a_h - b_l) - (a_l - b_h)}{2} &= \frac{a_h - b_l - a_l + b_h}{2} \\
    &= \frac{a_h - a_l + b_h - b_l}{2} \\
    &= \frac{a_h - a_l}{2} + \frac{b_h - b_l}{2}
    \end{aligned}
\end{equation}


 The two quantities being added are just the widths of $$a$$ and $$b$$ respectively, so the width of the difference $$a-b$$ is just the sum of the widths of the two terms.

 The same can be shown for the sum:

 \begin{equation}
   a + b = [a_l + b_l, a_h + b_h]
 \end{equation}

whose width is equal to

\begin{equation}
  \begin{aligned}
    \frac{(a_h + b_h) - (a_l + b_l)}{2} &= \frac{a_h + b_h - a_l - b_l}{2} \\
    &= \frac{a_h - a_l + b_h - b_l}{2} \\
    &= \frac{a_h - a_l}{2} + \frac{b_h - b_l}{2}
  \end{aligned}
\end{equation}

Again, we see that it's just the sum of the widths of the individual terms.

We can show that this is not true for multiplication (and by extension, division).  Let $$a=[0,2]$$ and $$b=[0,2]$$.  The product $$a \cdot b$$ product is the interval $$[0,4]$$, which might lead us to believe that it's simply the sum of double their widths, but if we keep $$a$$ the same and let $$b=[1,3]$$, $$a \cdot b = [0,6]$$.  This is because the result of multiplying two intervals is found by finding the $$min$$ and $$max$$ of the four different ways to multiply the upper and lower bounds, which is directly affected by the value of the upper/lower bounds and not simply just the width of each interval.
