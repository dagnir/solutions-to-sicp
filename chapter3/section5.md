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
(define partial-sums (cons-stream 1 (add-streams partial-sums (stream-cdr integers))))
```

This is very similar to the fibonacci solution in Exercise 3.54.  We define
`partial-sums` as the stream starting with `1`, and the rest of the stream
computed by adding the `partial-sums` stream with the `cdr` of the `integers`
stream. Again, this works because when it comes time to compute the `cdr` of
the `partial-sums` stream, we have just enough information computed (i.e. the
current partial sum is at the head of the stream) to cmpute the next element.

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
$$2$$ and depth $$n$$ so that the number of operations is $$\approx 2^n$$.
