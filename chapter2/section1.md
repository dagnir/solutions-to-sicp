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

$$
\begin{equation}
a - b = [a_l - b_h, a_h - b_l]
\end{equation}
$$

The width of $$a-b$$ is

$$
\begin{equation}
  \begin{aligned}
    \frac{(a_h - b_l) - (a_l - b_h)}{2} &= \frac{a_h - b_l - a_l + b_h}{2} \\
    &= \frac{a_h - a_l + b_h - b_l}{2} \\
    &= \frac{a_h - a_l}{2} + \frac{b_h - b_l}{2} \\
    \end{aligned}
\end{equation}
$$

 The two quantities being added are just the widths of $$a$$ and $$b$$ respectively, so the width of the difference $$a-b$$ is just the sum of the widths of the two terms.

 The same can be shown for the sum:

 $$
 \begin{equation}
   a + b = [a_l + b_l, a_h + b_h]
 \end{equation}
 $$

whose width is equal to

$$
\begin{equation}
  \begin{aligned}
    \frac{(a_h + b_h) - (a_l + b_l)}{2} &= \frac{a_h + b_h - a_l - b_l}{2} \\
    &= \frac{a_h - a_l + b_h - b_l}{2} \\
    &= \frac{a_h - a_l}{2} + \frac{b_h - b_l}{2} \\
  \end{aligned}
\end{equation}
$$

Again, we see that it's just the sum of the widths of the individual terms.

We can show that this is not true for multiplication (and by extension, division).  Let $$a=[0,2]$$ and $$b=[0,2]$$.  The product $$a \cdot b$$ product is the interval $$[0,4]$$, which might lead us to believe that it's simply the sum of double their widths, but if we keep $$a$$ the same and let $$b=[1,3]$$, $$a \cdot b = [0,6]$$.  This is because the result of multiplying two intervals is found by finding the $$min$$ and $$max$$ of the four different ways to multiply the upper and lower bounds, which is directly affected by the value of the upper/lower bounds and not simply just the width of each interval.

###Ex 2.10

```scheme
(define (div-interval x y)
  (if (= 0 (width-interval y))
      (error "Division by interval spanning 0 -- DIV-INTERVAL")
      (mul-interval x
		    (make-interval (/ 1.0 (upper-bound y))
				   (/ 1.0 (lower-bound y))))))
```

###Ex 2.11

```scheme
(define (mul-interval a b)
  (let ((al (lower-bound a))
	(ah (upper-bound a))
	(bl (lower-bound b))
	(bh (upper-bound b)))
    (cond ((and (<= al 0) (>= ah 0) (<= bl 0) (>= bh 0)) (make-interval (min (* al bh) (* ah bl)) (max (* al bl) (* ah bh))))
	  ((and (> al 0) (> ah 0) (>= bl 0) (> bh 0)) (make-interval (* al bl) (* ah bh)))
	  ((and (>= al 0) (> ah 0) (< bl 0) (<= bh 0)) (make-interval (* ah bl) (* al bh)))
	  ((and (> al 0) (> ah 0) (< bl 0) (> bh 0)) (make-interval (* ah bl) (* ah bh)))
	  ((and (< al 0) (> ah 0) (> bl 0) (> bh 0)) (make-interval (* al bh) (* ah bh)))
	  ((and (< al 0) (> ah 0) (< bl 0) (< bh 0)) (make-interval (* ah bl) (* al bl)))
	  ((and (< al 0) (<= ah 0) (>= bl 0) (> bh 0)) (make-interval (* al bh) (* ah bl)))
	  ((and (< al 0) (<= ah 0) (< bl 0) (<= bh 0)) (make-interval (* ah bh) (* al bl)))
	  (else (make-interval (* al bh) (* al bl))))))
```

This was a little tricky, but I'm pretty sure I got all the cases right...

###Ex 2.12

```scheme
(define (make-center-percent c t)
  (make-interval (- c (* c t)) (+ c (* c t))))

(define (percent i)
  (* 1.0 (/ (width i) (center i))))
```

###Ex 2.13

By playing around with some values, I noticed that the tolerance of the product can be approximated by the sum of the tolerances of each factor.

By assuming that both $$a$$ and $$b$$ are positive, we know that the product $$c = a \cdot b = [a_l \cdot b_l, a_h \cdot b_h]$$.

The exact tolerance of this interval is then

$$
\begin{equation}
  \begin{aligned}
    T(c) &= \frac{W(c)}{C(c)} \\
    &= \frac{a_hb_h - a_lb_l}{2} \cdot \frac{2}{a_lb_l - a_hb_h} \\
    &= \frac{a_hb_h - a_lb_l}{a_lb_l + a_hb_h} \\
  \end{aligned}
\end{equation}
$$

The sum of each factor's tolerance is:

$$
\begin{equation}
  \begin{aligned}
    T(a) + T(b) &= \frac{W(a)}{C(a)} + \frac{W(b)}{C(b)} \\
    &= \frac{a_h - a_l}{a_l + a_h} + \frac{b_h - b_l}{b_l + b_h} \\
    &= \frac{2a_hb_h - 2a_lb_l}{a_lb_l + a_lb_h + a_hb_l + a_hb_h} \\
    &\approx \frac{2(a_hb_h - a_lb_l)}{2(a_lb_l + a_hb_h)} \\
  \end{aligned}
\end{equation}
$$

The last line works because we assume that the tolerances are very small, meaning that $$a_l \approx a_h$$, so we can substitute accordingly.

###Ex 2.14

```scheme
(define a (make-center-percent 2 0.05))
(define b (make-center-percent 3 0.03))
(define c a)

(percent (div-interval a a))
=> 0.09975062344139651

(percent (div-interval a b))
=> 0.07988017973040436

(percent (div-interval a c))
=> 0.09975062344139651
```

I think this illustrates the problem properly.  We know from the previous exercise that for intervals with very small tolerances, the percentage of the product of quotient of two intervals is very close to the sum of their tolerances.  This can be shown from $$\frac{A}{B}$$.  However for $$\frac{A}{A}$$, the percent is $$\approx 0.1$$, twice the percent of `a`.  However, $$\frac{A}{A}$$ can simply be reduced to 1 mathematically with percent 0.  A related problem is the third application, $$\frac{A}{C}$$, where $$C=A$$.  Even though the names of the values are different here, they are equal, and so we sould expect the result to be 1 also.

###Ex 2.15
Yes, Eva is right: `par2` produces a result that has a lower percent.  This can be seen when comparing `(percent (par1 a b))` and `(percent (par2 a b))`, where

```scheme
(define a (make-center-percent 3 0.1))
(define b (make-center-percent 4 0.05))
```
In particular, `par2` will produce an interval with tighter percentage tolerance.  The reason this happens is because each time we perform an operation with intervals with width > 0, they will affect final result's percentage tolerance. In `par1`, we see that there are three operations in total that affect the resulting tolerance: $$R1 \cdot R2$$, $$R1 + R2$$, and the division of the results of those two operations.  In conrast, `par2` only results in *one* operation involving two intervals with width > 0: $$\frac{1}{R1} + \frac{1}{R2}$$.  Intuitively, we can think of it this way: It does not make sense for an operation involving two intervals a and b to result in an interval with *tighter* bounds than the terms used to compute it.

###Ex 2.16
This answer is related to the one given for 2.15.  In general, the reason that different algebraic expressions, although equivalent, can lead to different results is because each operation involving two intervals carried out directly (i.e. no simplication) increases the percentage tolerance.  For example, `(a + b) - b` is equivalent to just `a`, but computing the first expression directly will result in an interval that has a width greater than `a`'s, despite having the same center, which means it also has a higher percentage.
