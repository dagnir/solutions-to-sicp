###Ex 1.1

```scheme
10
12
8
3
6
a ;; value 3
b ;; value 4
(+ a b (* a b))
(+ a b (* 3 4))
(+ a b 12)
(+ 3 4 12)
19
#f

(if (and (< b (* a b)) (> b a))
    b
    a)
(if (and #t #t)
    b
    a)
b
4

(cond ((= a 4) 6)
      ((= b 4) (+ 6 7 a))
	  (else 25))
(+ 6 7 a)
(+ 6 7 3)
16
(+ 2 (if (> b a) b a))
(+ 2 (if #t b a))
(+ 2 b)
(+ 2 4)
6

(* (cond ((> a b) a)
         ((< a b) b)
	     (else -1))
   (+ a 1))
;; the cond evalues to b
(* b (+ a 1))
(* 4 (+ a 1))
(* 4 (+ 3 1))
(* 4 4)
16
```

###Ex 1.2

```scheme
(/ (+ 5 4 (- 2 (- 3 (+ 6 (/ 4 5)))))
   (* 3 (- 6 2) (- 2 7)))
```

###Ex 1.3

```scheme
(define (ex1-3 a b c)
  (cond ((and (> a b) (> a c))
         (if (> b c)
             (sos a b)
             (sos a c)))
        ((and (> b a) (> b c))
         (if (> a c)
             (sos b a)
             (sos b c)))
        (else (if (> a b)
                  (sos c a)
                  (sos c b)))))
```

###Ex 1.4

```scheme
(define (a-plus-abs-b a b)
    ((if (> b 0) + -) a b))
```

A combination with the procedure above for the operator looks like this:

```scheme
(a-plus-abs-b a b)
```

Using substitution to substitute the arguments into the body:

```scheme
((if (> b 0) + -) a b)
```

Now it's easy to to see that the operator in this combination is a compound procedure which must be evaluated.  Assuming `b > 0` we see that the `if` evaluates to the `+` operator:

```scheme
(+ a b)
```

then we just proceed with normal evaluation.

This comes straight from the rules of evaluation which states that when evaluating a combination, the operator and the operands must be evaluated first.  We simply follow that rule here, and since the thing appearing in the position of an operator is a combination, we proceed to evaluate it, which yields another operator.

###Ex 1.5

```scheme
(define (p) (p))

(define (test x y)
    (if (= x 0)
        0
        y))

(test 0 (p))
```
In **normal order** evaluation, the expression would return 0.

In **applicative order** evaluation, the expression would never terminate.  The reason for this is because during evaluation, the interpreter evaluates the operands first to determine their value before substituting them for the formal parameters in the body of the procedure.  When interpreter attempts to evaluate `(p)` to get at its value, it will enter an infinite loop because p is defined in terms of itself or "recursively", and has no terminating condition.

In **normal order** evaluation, this is not a problem because operands are substituted first, and only evaluated when their value is actually needed for the computation to proceed.  However, in this case, the value of `y` (whose value is `(p)`) is never actually referenced in the body of the procedure because the predicate of the `if` evaluates to `#t` and the consequent is evaluated and the intepreter never has to evaluate the alternative.

###Ex 1.6

The problem here is related to Ex 1.5.  The interpreter performs applicative order evaluation, so it evaluates the procedure's operands first.  This becomes an issue because the `else-clause` clause parameter in this case is a call to `sqrt-iter`.  The interpreter by design must evaluate this before moving on, and i doing so, ends up evaluating another call to `sqrt-iter`.  This will continue to happend forever because the interpreter will never actually enter the body of the procedure.

###Ex 1.7

The problem for very small numbers is that the value of `0.001` in `good-enough?` is too large.  For values that are small enough, a difference of `0.001` between `(square guess)` and `x` could still be too far from the actual root.

For example:

```scheme
(sqrt 0.000001)
```

return `0.03` but the actual root is `0.003`.

For large numbers, the limited precision becomes an issue.  On the one hand, we could get a guess that when we square it, we end up with a number that has more significant digits than we can represent (in this case with a double precision floating point value) so the less significant ones are lost.  In this case, it's possible that `good-enough?` returns true because it sees that the difference is less tahan `epsilon`--the digits that would have made it greater than epsilon have been dropped.

example:

```scheme
(sqrt 9007199254760990)
```

returns `94906265.62435691` but square of that and `x` differs by `0.0999`.

Another problem with `good-enough?` with large numbers occurs when `good-enough?` says a guess isn't good enough, but we cannot represent a better guess (via `improve`) because of precision issues.  This causes an infinite loop to occur because `improve` could reach a point successive calls will begin return the same value, but `good-enough?` will not allow the procedure to terminate.

The book suggests an alternative way to implement `good-enough?` by watching how `guess` changes between iterations, and stopping when the change is very small:

```scheme
(define (good-enough? oldguess newguess)
  (< (/ (abs (- newguess oldguess)) oldguess) 0.001))
```

This new version of `good-enough?` works better for both small and large values.  For small values, we avoid situations where we prematurely terminate because we now check how the guess changes between successive iterations, rather than stopping at some threshhold.  For large values, we avoid situations with infinite loops because when we can no longer improve the value (i.e. `improve` starts return the same value), we simply terminate because we see that the guess did not change.

###Ex 1.8

```scheme
(define (improve-curt-guess guess x)
  (/ (+ (/ x (square guess)) (* 2 guess))
     3))

(define (curt-iter oldguess newguess x)
  (if (good-enough? oldguess newguess)
      newguess
      (curt-iter newguess (improve-curt-guess newguess x) x)))

(define (curt x)
  (curt-iter 0.0 1.0 x))

(define (improve-cube-root-guess guess x)
```
