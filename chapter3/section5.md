###Ex 3.50

```scheme
(define (stream-map proc . argstreams)
  (if (stream-null? (car argstreams))
      the-empty-stream
      (cons-stream
       (apply proc (map car argstreams))
       (apply stream-map
              (cons proc (map stream-cdr argstreams))))))
```

This looks (unsurprisingly) very similar to the list version, but with stream
specific procedures where you'd expect to find the normal list versions, e.g.
`stream-cdr` instead of `cdr`.

###Ex 3.51

We're asked to state what would be printed by the evaluation of the following
expressions:

```scheme
(define (show x)
  (display-line x)
  x)

(define x (stream-map show (stream-enumerate-interval 0 10)))
(stream-ref x 5)
(stream-ref x 7)
```

The first line, where `x` is defined, prints `0`.  This is because in
evaluating the `stream-map`, `cons-stream` is called, and `stream-map` must
apply `show` to the head of the input stream.  However, no more than that is
printed because the rest of the enumeration is delayed.

For `(stream-ref x 5)`, it will iterate through the elements of the stream
until the 6th element is found.  Since only the head of `x` has been computed
so far, the remaining 5 elements must be computed (by applying `show`), so the
following is printed:

```
1
2
3
4
5
5 5 <-- result from stream-ref
```

Now, we evaluate `(stream-ref x 7)`.  We have to remember now that delayed
computations are actually memoized here.  That is, `show` is only applied to
each element in the enumeration stream only once, and then the result is saved.
So when `stream-ref` goes over the first 6 elements again as in `(stream-ref x
5)`, nothing is printed because `show` is not applied, but rather the saved
result is returned immediately.  However, the 7th and 8th elements have not yet
been computed, so we apply `show` here so that the interpreter prints

```
6
7 7 <-- result from stream-ref
```

Note the double `5` and double `7`; the second values are the results of the
two `stream-ref`s.

###Ex 3.52

```scheme
(define sum 0)

(define (accum x)
  (set! sum (+ x sum))
  sum)

(define seq (stream-map accum (stream-enumerate-interval 1 20)))
; sum = 1
```

After this expression, `sum` is $$1$$ since `stream-map` must apply `accum`
once to create the head of the stream.  It's also important to see exactly the
elements of `seq` are here.  Each element of the stream is the sum of all the
elements that came before it, including itself.  This sequence is known as the
[triangular numbers](https://oeis.org/A000217).  In particular, this sequece
consists of the following elements in order: $$0, 1, 3, 6, 10, 15, 21, 28, 36,
45, 55, 66, 78, 91, 105, 120, 136, 153, 171, 190, 210$$.

```scheme
(define y (stream-filter even? seq))
; sum = 6
```

After this line, `sum` is $$3$$ because `stream-filter` will continue walking
down the stream until it encounters the first element that satsfies the
predicate ($$6$$ in this case), after wich the rest of the computation will be
delayed.

```scheme
(define z (stream-filter (lambda (x) (= (remainder x 5) 0)) seq))
; sum = 15
```

This is similar to filter on `even?` but this time it continues to walk down
the stream until the first element that is divisible by 5 is encountered, which
happens to be 10.  At that point, `sum` will have had the elements $$3,4$$
added to it; the earlier values in the stream are not added to `sum` again
because of memoization.

```scheme
(stream-ref y 7)
; sum = 136
```

`y` are the even numbers in `seq`, including $$0$$.  The 8th element in the
sequence is $$136$$.

```scheme
(display-stream z)
; sum = 210
```

First, for the value of `sum`, it is now $$210$$ since displaying the entirety
of `z` will cause `accum` to be applied all the elements in the interval
$$[1,20]$$ that were not yet encountered up this point.

As for what's printed, that's all the triangular numbers that are multiples of
five up to $$210$$ which are:

```
10
15
45
55
105
120
190
210
```

The book also asks if the responses would have been different if `delay` were
implemented without `memo-proc`.  The answer is yes.  `y` and `z` are
filterings of `seq`, which maps `accum` to an interval stream from $$1$$ to
$$20$$.  Without memoizing the results of forcing `delay`s, `accum` will be
applied more than once for each element in the interval stream so that the
elements of `seq` will no longer be the triangular numbers.  Basically, each
time an element from either `y` or `z` is computed, an element from `seq` must
be computed, requiring a call to `accum` and thus changing the value of `sum`,
and also what we see as the head of `seq`.

###Ex 3.53

I'll repeat the code given in the book:

```scheme
(define s (cons-stream 1 (add-streams s s)))
```

We can see that `s` is a stream that starts with $$1$$, and the rest of the
elements are the result of adding `s` to itself, effectively doubling it, so
just like the `doubles` stream given as an example in the book, elements of `s`
are powers of two.  This can be verified by evaluating the line, and then using
`display-stream` or `stream-ref`.


###Ex 3.54

Here is the definition of `mul-streams`:

```scheme
(define (mul-streams s1 s2)
  (stream-map * s1 s2))
```

The book asks us to use `mul-stream` and the `integers` stream to complete the
definition of `factorials`.  To generate the next element in the sequence, we
need to the current element, and multiply it by the next integer in the
sequence of integers, which gives us the following defintion:

```scheme
(define factorials (cons-stream 1 (mul-streams factorials integers))
```

So factorials is defined as the stream starting with `1` with the rest being
computed by multipling the $$n^{\textrm{th}}$$ `factorial` with the
corresponding $$n^{\textrm{th}}$$ integer.

###Ex 3.55

```scheme
(define (partial-sums s)
  (define sums (cons-stream (stream-car s) (add-streams sums (stream-cdr s))))
  sums)
```

This is very similar to the fibonacci solution in Exercise 3.54.  We define
`partial-sums` as the stream starting with the first element in the stream `s`,
and the rest of the stream computed by adding the `partial-sums` stream with
the `cdr` of the `s` stream. Again, this works because when it comes time to
compute the `cdr` of the `partial-sums` stream, we have just enough information
computed (i.e. the current partial sum is at the head of the stream) to cmpute
the next element.

###Ex 3.56

For this problem, we end up with three streams because `S` must be scaled by
$$2$$,$$3$$,and $$5$$, so we first merge the the streams produced by scaling
`S` by $$2$$ and $$3$$, and then merge that stream with the stream produced by
scaling `S` by $$5$$:

```scheme
(define S (cons-stream 1 (merge (merge (scale-stream S 2) (scale-stream S 3)) (scale-stream S 5))))
```

###Ex 3.57
Here is the definition of `fibs` repeated:

```scheme
(define fibs
  (cons-stream 0
               (cons-stream 1
                            (add-streams (stream-cdr fibs) fibs))))
```

The number of addition operations is $$O(n)$$ for the definition of `fibs`
based on `add-streams` because to compute the $$n+1$$ element in the stream we
need only have the $$n$$ and $$n-1$$ element, both of wich are available and
memoized for us so that accessing them is only an $$O(1)$$ operation.

If `delay` were implemented without `memo-proc`, then the access of the $$n$$
and $$n-1$$ are no longer $$O(1)$$ because the value must be recomputed on each
access.  Computing the $$n^{\textrm{th}}$$ now requires computing the $$n$$ and
$$n-1$$ element, so in effect, the recursion tree now has a branching factor of
$$2$$ and depth $$n$$ so that the number of operations is $$\approx O(2^n)$$.

###Ex 3.58

```scheme
(define (expand num den radix)
  (cons-stream
   (quotient (* num radix) den)
   (expand (remainder (* num radix) den) den radix)))
```

After inspection, we can see that this actually provides radix expansion of the
given rational number, $$\frac{\textrm{num}}{\textrm{den}}$$.  We can see this
by thinking carefully about the structure of th stream: the first element is
the quotient of `num` and `den` (multiplied by `radix`), and the rest of the
stream is the expansion of the remainder over `den` (multiplied by `radix`).

So, `(expand 1 7 10)` gives the *decimal* expansion of $$\frac{1}{7}$$, and
`(expand 3 8 10)` gives the decimal expansion of $$\frac{3}{8}$$.

$$\frac{1}{7}$$ repeats so the elements of the stream `(expand 1 7 10)`
consists of infinitely many sequences of $$1,4,2,8,5,7$$, whereas
$$\frac{3}{8}$$ does not, and the stream consists of $$3,7,5$$, followed by
zeroes.

###Ex 3.59
First, here is the definition of `integrate-series`:

```scheme
(define (integrate-series s)
  (define (recur d s)
    (if (stream-null? s)
        s
        (cons-stream (* (/ 1 d) (stream-car s))
                     (recur (+ d 1) (stream-cdr s)))))
  (recur 1 s))
```

We simply construct the stream recursively, multiplying each element by the
appropriate coefficient.

Here are `cosine-series` and `sine-series`:
```scheme
(define cosine-series (cons-stream 1 (integrate-series (scale-stream sine-series -1))))

(define sine-series (cons-stream 0 (integrate-series cosine-series)))
```

like the mathematical definition given in the book, the definitions of the
streams refer to each other.  Interestingly, this works because there is always
enough data already computed for either stream to compute the next element in
each stream.  It's certainly a little mind bending to think in depth about!

###Ex 3.60

To multiply two series $$A$$ and $$B$$, we simply take the first element in
$$A$$, (call it $$A_0$$), and scale $$B$$ by $$A_0$$, and add to this the
result of multiplying the rest of $$A$$ with $$B$$.

```scheme
(define (mul-series s1 s2)
  (cons-stream (* (stream-car s1)
                  (stream-car s2))
               (add-streams (scale-stream (stream-cdr s2) (stream-car s1))
                            (mul-series (stream-cdr s1) s2))))
```

Then, checking our work:

```scheme
(define cos-sq-series (mul-series cosine-series cosine-series))
(define sin-sq-series (mul-series sine-series sine-series))
(define cos-sq-plus-sin-sq (add-streams cos-sq-series sin-sq-series))
```

Inspecting the elements of `cos-sq-plus-sin-sq`, it's $$1$$ followed by $$0$$
it looks like we've got it correct.  Of course we can also check the elements
of `cos-sq-series` and `sin-sq-series` and compare them against the Taylor
expansions coefficients.

###Ex 3.61

We can more or less translate the definition direcly for `invert-unit-series`:

```scheme
(define (invert-unit-series s)
  (cons-stream 1 (mul-series (invert-unit-series s)
                             (scale-stream (stream-cdr s) -1))))
```

_Note_: We are computing the *multiplicative* inverse of the series, so if we
have

```scheme
(define x (invert-unit-series cosine-series))
```

`x` is the series expansion of $$\sec$$ and NOT $$\arccos$$.  I missed this
when I initially did the problem because I was comparing the values in the
stream to those in the series expansion of $$\arccos$$ rather than $$\sec$$.

###Ex 3.62

As the problem suggests, we already have all the pieces we need from the
previous exercises.  To implmenet division, we multiply the numerator series
with the inverse of the denominator series:

```scheme
(define (div-series n d)
  (if (= (stream-car d) 0)
      (error "Division by ZERO -- DIV-SERIES")
      (mul-series n (invert-unit-series d))))
```

$$\tan x = \frac{\sin x}{\cos x}$$, so we define `tan-series` as:

```scheme
(define tan-series (div-series sine-series cosine-series))
```

###Ex 3.63

Here is Louis' version:

```scheme
(define (sqrt-stream x)
  (cons-stream 1.0
               (stream-map (lambda (guess)
                             (sqrt-improve guess x))
                           (sqrt-stream x))))
```

We can see that the reason this is less efficient is that instead of defining
the stream recursively by `cons`ing it onto itself, the `cons` of the stream is
instead a completely new stream.  Since each each element is computed by
`force`ing the `cons` of the stream, and the `cons` of the stream will always
be a delayed computation that we haven't seen before, we effectively get no
memoization.  This means that this version would be identical to the first
version if `delay` were implemented as a simple `lambda` without the
optimization from `memo-proc`.

###Ex 3.64

```scheme
(define (stream-limit s t)
  (let ((e1 (stream-car s))
        (e2 (stream-car (stream-cdr s))))
    (if (< (abs (- e1 e2)) t)
        e2
        (stream-limit (stream-cdr s) t))))
```

No surprises for this solution, a simple iterative process does the trick.

###Ex 3.65

First, we define the summands of the $$\ln 2$$ stream:

```scheme
(define (ln2-summands n)
  (cons-stream (/ 1.0 n)
               (stream-map - (ln2-summands (+ n 1)))))

(define ln2-series (ln2-summands 1))
```

Just like for $$\pi$$, we can simply get successively better approximations by
using `partial-sums`:

```scheme
(define ln2-stream-1 (partial-sums ln2-series))
```

Again, the downside to this method is that it can take a while for it to get
accurate enough.

Next, we can use `euler-transform` to get better approximations faster:

```scheme
(define ln2-stream-2 (euler-transform ln2-stream-1))
```

Finally, we can do even better bo using `accelerated-sequence`:

```scheme
(define ln2-stream-3 (accelerated-sequence euler-transform ln2-stream-1))
```

###Ex 3.67

Here is the table that the book  presents that contains all the elements of the
`pairs` stream:

| | | | | |
|-|-|-|-|-|
|(S0, T0)|(S0, T1)|(S0, T2)|...
|(S1, T0)|(S1, T1)|(S1, T2)|...
|(S2, T0)|(S2, T1)|(S2, T2)|...
|(S3, T0)|(S3, T1)|(S3, T2)|...

For the original version of `pairs`, we excluded all the elements in the first
column, that is, the elements $$(S_1, T_0), (S_2, T_0), (S_3, T_0)$$, and so
on.  To get all pairs of integers, we need to add this extra stream:

```scheme
(define (pairs-all s t)
  (cons-stream
   (list (stream-car s) (stream-car t))
   (interleave
    (stream-map (lambda (x) (list (stream-car s) x))
                (stream-cdr t))
    (interleave
     (stream-map (lambda (x) (list x (stream-car t)))
                 (stream-cdr s))
     (pairs-all (stream-cdr s) (stream-cdr t))))))
```

Here we make use of two applications of `interleave`: In the innermost
application, we interleave the rest of the first column with the recursive call
to `pairs-all`. Then we interleave the resulting stream with the the rest of
the first-row.

###Ex 3.68

No this doesn't work.  Trying to run Louis' version will result in an infinite
loop.

Here is Louis' version:

```scheme
(define (pairs s t)
  (interleave
   (stream-map (lambda (x) (list (stream-car s) x))
               t)
   (pairs (stream-cdr s)
          (stream-cdr t))))
```

It seems resonable because it technically does what we want, which is
interleave the first row pairs with elements from the rest of the rows.
However, what's really wrong with this is that `interleave` is *not* like
`cons-stream`.  That is, its arguments will be evaluated right away.  Of course
this is a problem because the second argument is the recursive call to `pairs`,
and since a stream like `integers` is infinite, the recursion will never
terminate.

###Ex 3.69

```scheme
(define (triples s t u)
  (let ((a (stream-car s))
        (b (stream-car t))
        (c (stream-car u)))
    (cons-stream
     (list a b c)
     (interleave
      (stream-map (lambda (x) (list a b x))
                  (stream-cdr u))
      (interleave
       (stream-map (lambda (x) (cons a x))
                   (pairs (stream-cdr t) (stream-cdr u)))
       (triples (stream-cdr s) (stream-cdr t) (stream-cdr u)))))))
```

Here I have an interleaving of three streams.  The first element in the stream
consists of the heads of the three streams, $$(S_0, T_0, U_0)$$.  Next, we have
the stream consisting of elements of the form $$(S_0, T_0, U_n)$$, where
$$n>0$$.  For the second stream, we need the elements $$T_i$$ and $$U_j$$ such
that they are greater than $$S_0$$ and $$T_i \leq U_j$$, which is exactly what
`pairs` will give us.  Finally, the third stream is the recursive call to
`triples`.

Having defined `triples`, `pythagorean-triples` becomes a filtering of the
stream:

```scheme
(define pythagorean-triples
  (stream-filter
   (lambda (t)
     (let ((i (car t))
           (j (cadr t))
           (k (caddr t)))
       (= (+ (* i i) (* j j)) (* k k))))
   (triples integers integers integers)))
```

###Ex 3.70

```scheme
(define (merge-weighted s1 s2 weight)
  (cond ((stream-null? s1) s2)
        ((stream-null? s2) s1)
        (else
         (let ((s1car (stream-car s1))
               (s2car (stream-car s2)))
           (cond ((<= (weight s1car) (weight s2car))
                  (cons-stream s1car (merge-weighted (stream-cdr s1) s2 weight)))
                 ((> (weight s1car) (weight s2car))
                  (cons-stream s2car (merge-weighted s1 (stream-cdr s2) weight)))
                 (else (cons-stream s1car (merge-weighted (stream-cdr s1) (stream-cdr s2) weight))))))))
```

For `merge-weighted`, it's a small modification to the original.  Instead of
comparing the elements directly, we compare the values after applying `weight`
to them.  Also for equal weight pairs, we don't get rid of one of them as in
the original `merge`, instead they both make it into the stream.

```scheme
(define (pairs-weighted s t weight)
  (cons-stream
   (list (stream-car s) (stream-car t))
   (merge-weighted
    (stream-map (lambda (x) (list (stream-car s) x))
                (stream-cdr t))
    (pairs-weighted (stream-cdr s) (stream-cdr t) weight)
    weight)))
```

Here we replace `interleave` with `merge-weighted`.


Finally, we define the two streams as described in the book

```scheme
(define s1 (pairs-weighted integers integers (lambda (p) (+ (car p) (cadr p)))))
(define s2 (stream-filter (lambda (p)
                            (let ((i (car p))
                                  (j (cadr p)))
                              (not (or (= (remainder (* i j) 2) 0)
                                       (= (remainder (* i j) 3) 0)
                                       (= (remainder (* i j) 5) 0)))))
                          (pairs-weighted integers integers
                                          (lambda (p)
                                            (let ((i (car p))
                                                  (j (cadr p)))
                                              (+ (* 2 i) (* 3 j) (* 5 i j)))))))
```

###Ex 3.71

```scheme
(define (w1 p)
  (let ((i (car p))
        (j (cadr p)))
    (+ (* i i i) (* j j j))))

(define (rn)
  (define s (pairs-weighted integers integers w1))
  (define (iter s)
    (let ((e1 (stream-car s))
          (e2 (stream-car (stream-cdr s))))
        (if (= (w1 e1) (w1 e2))
            (cons-stream (w1 e1) (iter (stream-cdr (stream-cdr s))))
            (iter (stream-cdr s)))))
  (iter s))
```

Pretty straightforward definition.  Using `rn`, we can inspect the generated
stream to see that the next five Ramanujan numbers are: $$4104, 20683, 32832,
64232, 65728$$.

###Ex 3.72

```scheme
  (define (w2 p)
    (let ((i (car p))
          (j (cadr p)))
      (+ (* i i) (* j j))))
(define (sum-of-2-squares)
  (define s (pairs-weighted integers integers w2))
  (define (iter s)
    (let ((p1 (stream-car s))
          (p2 (stream-car (stream-cdr s)))
          (p3 (stream-car (stream-cdr (stream-cdr s)))))
      (if (= (w2 p1) (w2 p2) (w2 p3))
          (cons-stream (cons (w2 p1) (list p1 p2 p3)) (iter (stream-cdr (stream-cdr (stream-cdr s)))))
          (iter (stream-cdr s)))))
  (iter s))
```

Very similar to to 3.71.  Note that the string consists of pairs of the number
and the three different pairs.

###Ex 3.73

```scheme
(define (RC R C dt)
  (lambda (i v0)
    (add-streams (scale-stream i R)
                 ((integral (scale-stream i (/ 1 C)) v0 dt)))))
```

The diagram was very helpful for this exercise and just involved a direct
translation to code.
