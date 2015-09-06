###Ex 1.29

```scheme
(define (simpsons-rule f a b n)
  (define h (/ (- b a) n))
  (define (term k)
    (define c (cond ((or (= k 0)
			 (= k n)) 1)
		    ((even? k) 2)
		    (else 4)))
    (* c (f (+ a
	       (* k h)))))
  (define (next k) (+ k 1))
  (* (/ h 3.0)
     (sum term 0 next n)))
```

###Ex 1.30

Iterative version of `sum`:

```scheme
(define (sum term a next b)
  (define (iter a result)
    (if (> a b)
	result
	(iter (next a)
	      (+ result
		 (term a)))))
  (iter a 0))
```

###Ex 1.31

Iterative version of `product` analogous to `sum`:

```scheme
(define (product term a next b)
  (define (iter a result)
    (if (> a b)
	result
	(iter (next a)  (* result (term a)))))
  (iter a 1))
```

`factorial` in terms of `product`:

```scheme
(define (factorial n)
  (product-recur (lambda (i) i)
		 1
		 (lambda (i) (+ 1 i))
		 n))
```

Approximation of $$\pi$$ using `product` (Wallis Product):

```scheme
(define (pi-approx n)
  (define (n-term a)
    (if (even? a)
	(+ a 2)
	(+ a 1)))
  (define (d-term a)
    (if (even? a)
	(+ a 1)
	(+ a 2)))
  (define (term a)
    (/ (n-term a)
       (d-term a)))
  (define (next a)
    (+ 1 a))
  (* 1.0 (product term 1 next n)))
```

b. Recursive version of `product`:

```scheme
(define (product term a next b)
  (if (> a b)
      1
      (* (term a) (product term (next a) next b))))
```

###Ex 1.32

```scheme
(define (accumulate combiner null-value term a next b)
  (define (iter i a)
    (if (> i b)
	a
	(iter (next i) (combiner a (term i)))))
  (iter a null-value))
```

`sum` in terms of `accumulate`:

```scheme
(define (sum term a next b)
  (accumulate + 0 term a next b))
```

`product` in terms of `accumulate`:

```scheme
(define (product term a next b)
  (accumulate * 1 term a next b))
```

b. recursive version of `accumulate`

```scheme
(define (accumulate combiner null-value term a next b)
  (if (> a b)
      null-value
      (combiner (term a) (accumulate combiner null-value term (next a) next b))))
```

###Ex 1.33

```scheme
(define (filtered-accumulate combiner null-value predicate term a next b)
  (define (iter i a)
    (if (> i b)
	a
	(if (predicate i)
	    (iter (next i) (combiner (term i) a))
	    (iter (next i) a))))
  (iter a null-value))
```

a.

```scheme
(define (sum-of-squares-of-primes a b)
  (define (term x) (square x))
  (define (next i) (+ 1 i))
  (filtered-accumulate + 0 prime? term a next b))
```

b.

```scheme
(define (product-ints-relatively-prime-to-n n)
  (define (pred i)
    (= 1 (gcd i n)))
  (define (term x) x)
  (define (next i) (+ 1 i))
  (filtered-accumulate * 1 pred term 1 next n))
```

###Ex 1.35

The golden ratio $$x$$ satisfies the equation

$$ x^2 = x + 1 $$

We can rewrite this as

$$ x = 1 + \frac{1}{x} $$

or

$$ f(x) = 1 + \frac{1}{x} $$

Therefore, to find the golden ratio, we need to find the fixed point of $$f(x)$$:

```scheme
(define golden-ratio (fixed-point (lambda (x) (+ 1 (/ 1 x)))
				  1.0))
```

###Ex 1.36

```scheme
(define (fixed-point f first-guess)
  (define (close-enough? v1 v2)
    (< (abs (- v1 v2)) tolerance))
  (define (try guess)
    (let ((next (f guess)))
      (display "approximation: ")
      (display next)
      (newline)
      (if (close-enough? guess next)
	  next
	  (try next))))
  (try first-guess))
```

To find the solution to

$$ x^x = 1000 $$

we find the fixed point of the function

$$ f(x) = \frac{\log{1000}}{log{x}} $$

for the non average damped version:

```scheme
(fixed-point (lambda (x) (/ (log 1000)
			    (log x))))
```

for the average damped version:

```scheme
(fixed-point (lambda (x) (average x
				  (/ (log 1000)
				     (log x)))))
```

The non average damped version takes 34 steps, while the average damped version take 9 steps.

###Ex 1.37

```scheme
(define (cont-frac n d k)
  (define (recur i)
    (if (= i k)
	(/ (n k)
	   (d k))
	(/ (n i)
	   (+ (d i)
	      (recur (+ i 1))))))
  (recur 1))
```

iterative version:

```scheme
(define (cont-frac n d k)
  (define (iter i result)
    (if (= i 0)
	result
	(iter (- i 1)
	      (/ (n i)
		 (+ (d i) result)))))
  (iter (- k 1) (/ (n k)
		   (d k))))
```

To get a result accurate to 4 decimal places, `k` must be at least 11.

###Ex 1.38

Approximation of $$e$$:

```scheme
(define (e-approx k)
  (define (n i) 1.0)
  (define (d i)
    (let ((j (- i 2)))
      (if (= (remainder j 3) 0)
	  (- i (/ j 3))
	  1)))
  (+ 2 (cont-frac n d k)))
```

###Ex 1.39

Approximation of $$tan$$:

```scheme
(define (tan-cf x k)
  (define (n i)
    (if (= i 1)
	x
	(- (square x))))
  (define (d i)
    (let ((j (- i 1)))
      (+ (* 2 j) 1)))
  (* 1.0 (cont-frac n d k)))
```


###Ex 1.40

```scheme
(define (cubic a b c)
  (lambda (x)
    (+ (cube x) (* a (square x)) (* b x) c)))
```

###Ex 1.41

```scheme
(define (double f)
  (lambda (x) (f (f x))))
```

```scheme
(((double (double double) inc) 5)
=> 21
```

###Ex 1.42

```scheme
(define (compose f g)
  (lambda (x)
    (f (g x))))
```

###Ex 1.43

```scheme
(define (repeated f n)
  (if (= n 1)
      f
      (compose f (repeated f (- n 1)))))
```

###Ex 1.44

```scheme
(define (smooth f)
  (lambda (x)
    (/ (+ (f (- x dx))
	  (f x)
	  (f (+ x dx)))
       3)))
```

###Ex 1.45
After doing some experiments, the number of repeated `average-damp`s required seems to grow as the floor of the $$log_2(n)$$ where $$n$$ is the root we're looking for (more succinctly, $$\lfloor log_2{n} \rfloor$$). By increasing the value of $$n$$, I found that the 5th, 6th, and 7th roots required only two repeated `average-damp`s, but finding the 8th root required a third in order for `fixed-point` to converge.  An additional application is not needed until we get to the 16th root.

```scheme
(define (floor-log2 x)
  (floor (/ (log x)
	    (log 2))))

(define (nth-root x n)
  (define avg (repeated average-damp
			(floor-log2 n)))
  (define f (lambda (y) (/ x
			   (expt y (- n 1)))))
  (fixed-point (avg f)
	       1.0))
```

###Ex 1.46

```scheme
(define (iterative-improve good-enough? improve)
  (define (iter guess)
    (let ((next (improve guess)))
      (if (good-enough? guess next)
	  next
	  (iter next))))
  iter)
```

`sqrt` in terms of `iterative-improve`:

```scheme
(define (sqrt x)
  (define (improve y)
    (average y (/ x y)))
  ((iterative-improve close-enough? improve) 1.0))
```

`fixed-point` in terms of `iterative-improve`:

```scheme
(define (fixed-point f guess)
  (define (improve y)
    (f y))
  ((iterative-improve close-enough? improve) guess))
```
