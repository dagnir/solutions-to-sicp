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

Finally, we can define `width-rect` and `height-rect` in terms of the above procedure:

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
