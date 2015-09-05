###Ex 1.9

```scheme
(define (+ a b)
    (if (= a 0)
	    b
        (inc (+ (dec a) b))))
```

Process evolution:

```scheme
(+ 4 5)
(inc (+ 3 5))
(inc (inc (+ 2 5)))
(inc (inc (inc (+ 1 5))))
(inc (inc (inc (inc (+ 0 5)))))
(inc (inc (inc (inc 5))))
(inc (inc (inc 6)))
(inc (inc 7))
(inc 8)
9
```

This process is clearly recursive.

```scheme
(define (+ a b)
    (if (= a 0)
        b
        (+ (dec a) (inc b))))
```

Process evolution:

```scheme
(+ 4 5)
(+ 3 6)
(+ 2 7)
(+ 1 8)
(+ 0 9)
9
```

This proces is iterative.

###Ex 1.10

```scheme
(define (A x y)
  (cond ((= y 0) 0)
	((= x 0) (* 2 y))
	((= y 1) 2)
	(else (A (- x 1)
		 (A x (- y 1))))))

(A 1 10) ;; evaluates to 1024
(A 2 4) ;; evaluates to 65536
(A 3 3) ;; evaluates to 65536
```

```scheme
(define (f n) (A 0 n))

(define (g n) (A 1 n))

(define (h n) (A 2 n))

(define (k n) (* 5 n n))
```

`f` is equivalent to \\(2n\\)

`g` is equivalent to \\(2^n\\)

`h` is equivalent to \\(2 \Uparrow n\\)

`k` is equivalent to \\(5n^2\\)

###Ex 1.11
Recursive process:

```scheme
(define (f n)
  (if (< n 3)
      n
      (+ (f (- n 1))
         (* 2 (f (- n 2)))
         (* 3 (f (- n 3))))))
```

Iterative version:

```scheme
(define (f n)
  (define (f-iter a b c n)
    (if (= n 0)
        c
        (f-iter b c (+ c (* 2 b) (* 3 a)) (- n 1))))
  (if (< n 3)
      n
      (f-iter 0 1 2 (- n 2))))
```

###Ex 1.12

It's a little easier to see if the triangle is drawn so that everything is left-aligned:

<pre>
1
1 1
1 2 1
1 3 3 1
1 4 6 4 1
</pre>

Then we can see that the any element of the triangle is the sum of the element directly above, and the element directly above and one over to the left.

```scheme
(define (pte r c)
  (define (at-edge?)
    (or (= c 0)
        (= c r)))
  (if (at-edge?)
      1
      (+ (pte (- r 1) c)
         (pte (- r 1) (- c 1)))))
```

###Ex 1.13

Prove:

$$ Fib(n) = \frac{\phi^n - \psi^n}{\sqrt{5}} $$

where \\(\phi\\) and \\(\psi\\) are roots of the golden ratio.

Some useful facts:

$$ 1 + \phi = \phi^2 $$

$$ 1 + \psi = \psi^2 $$

Proof by Induction:

Base Case:

\\(n = 0\\)

$$ Fib(0) = \frac{\phi^0 - \psi^0}{\sqrt{5}} = \frac{0}{\sqrt{5}} = 0 $$

Inductive Hypothesis:

Assume that \\(Fib(n) = \frac{\phi^k - \psi^k}{\sqrt{5}}\\)

Inductive Step:

\\(n = k + 1\\)

By definition of the Fibonacci series:

$$ Fib(k+1) = Fib(k) + F(k - 1) $$

Invoke Inductive Hypothesis:

$$ = \frac{\phi^k - \psi^k}{\sqrt{5}} - \frac{\phi^{k - 1} - \psi^{k - 1}}{\sqrt{5}} $$

Simplify:

$$ = \frac{\phi^k + \phi^{k - 1} + \psi^k + \psi^{k - 1}}{\sqrt{5}} $$

$$ = \frac{\phi^k(1 + \phi^{-1}) + \psi^k(1 + \psi^{-1})}{\sqrt{5}} $$

$$ = \frac{\phi^k(1 + \frac{1}{\phi}) + \psi^k(1 + \frac{1}{\psi})}{\sqrt{5}} $$

$$ = \frac{\phi^k(\frac{\phi}{\phi} + \frac{1}{\phi}) + \psi^k(\frac{\psi}{\psi} + \frac{1}{\psi})}{\sqrt{5}} $$

$$ = \frac{\phi^k(\frac{1 + \phi}{\phi}) + \psi^k(\frac{1 + \psi}{\psi})}{\sqrt{5}} $$

$$ = \frac{\phi^k(\frac{\phi^2}{\phi}) + \psi^k(\frac{\psi^2}{\psi})}{\sqrt{5}} $$

$$ = \frac{\phi^k(\phi) + \psi^k(\psi)}{\sqrt{5}} $$

$$ = \frac{\phi^{k + 1} + \psi^{k + 1}}{\sqrt{5}} $$


###Ex 1.14

For `count-change`, the space required is \\(O(n)\\) where \\(n\\) is the amount.  This is clear to see when drawing the process tree where the longest branch is when only pennies are used.

For the number of steps, `count-change` is \\(O(n^5)\\).  This can be seen most easily by limiting the kinds of coins to 1, and gradually increasing the kinds of coins.  With only a single type of coin (a penny), the number of steps is proportional to n because we substract its denomination from `n` at each step, and this can only happen at most `n` times.  For two kinds of coins, the number of times we use the larger coin is again proportional to `n`.  But at each one of these steps, we also choose not to use it, so we have a call to `cc` with 1 kind of coin.  So we have \\(O(n)\\) calls to `cc` with 2 kinds of coins, and during each of these calls, we have a call to `cc` using only 1 kind of coin, which we already know is \\(O(n)\\).  So \\(O(n)\\) calls each of which leads to \\(O(n)\\) creates \\(O(n^2)\\) calls all together.  for 3 kinds of coins, the pattern becomes clear.  We have \\(O(n)\\) calls to `cc` with `kinds-of-coins` equal to 2, and we know each of those is \\(O(n^2)\\), so with `kinds-of-coins` equal to 3, the time complexity is \\(O(n^3)\\).  For \\(n=4\\), it's \\(O(n^4)\\) and finally for \\(n=5\\), it's \\(O(n^5)\\).

It's also clear to see that `count-change` finds all the ways to combine each coin to equal the amount, which each step (leaf in the tree) corresponding to the use of one of the kinds of coins, so the number of steps would roughly be equal to

$$ \frac{n}{50} * \frac{n}{25} * \frac{n}{10} * \frac{n}{5} * \frac{n}{1} $$

which is clearly bounded above by \\(n^5\\).

###Ex 1.15
a. `p` is applied 5 times.

b. Since it divides `angle` by 3.0 each time until it's \\(<0.1\\), which is \\(O(\log_3{n})\\), where `n` is the angle.  Both space and the number of steps is \\(O(log_3{n})\\).


###Ex 1.16

This was a little tricky but the hints given are very helpful.  To make it work, we just need to make sure that whenever `n` is even, for the next iteration, we halve `n` and square `b`.  This maintains the invariant because \\(b^n = (b^2)^{n/2}\\).  When `n` is odd, rather than squaring b, we multiply it with `a` and subtract 1 from `n`.  Again, this maintains the invariant because \\(ab^n = (ab)b^{n - 1}\\).

```scheme
(define (fast-expt b n)
  (define (expt-iter a b n)
    (cond ((= n 0) a)
          ((even? n) (expt-iter a (square b) (/ n 2)))
          (else (expt-iter (* a b) b (- n 1)))))
  (expt-iter 1 b n))
```

###Ex 1.17

```scheme
(define (* a b)
  (cond ((= b 1) a)
        ((even? b) (double (* a (halve b))))
        (else (+ a (* a (- b 1))))))
```

Straightforward, when `b` is even, we double the result of \\(a(b/2)\\), and when `b` is odd, we add `a` to the result of \\(a(b -1)\\).

###Ex 1.18

```scheme
(define (* a b)
  (define (iter c a b)
    (cond ((= b 0) c)
          ((even? b) (iter c (double a) (halve b)))
          (else (iter (+ c a) a (- b 1)))))
  (iter 0 a b))
```

Again, the trick here is to maintain the invariant which is \\(c + ab\\).  The solution is very similar to the one for 1.16.  When `b` is even, we halve it and double a, and when it's odd, we subract 1 and add `a` to `c`.

###Ex 1.19

After first transformation:

$$ a' = bq + aq + ap $$
$$ b' = bp + aq $$

and after second transformation:

$$ a'' = (bp + aq)q + (bq + aq + ap)q + (bq + aq + ap) p $$
$$ b'' = (bp + aq)p + (bq + aq + ap)q $$

Now, it's a matter of gathering and simplifying terms so that \\(a''\\) is in the form

$$ bq' + aq' + ap' $$

and likewise, \\(b''\\) looks like

$$ bp' + aq' $$

Since \\(b''\\) only has one of each term, we'll start with it first:

$$ b'' = bp^2 + apq + bq^2 + aq^2 + apq $$

By rearranging terms, we get:

$$ b '' = b(p^2 + q^2) + a(q^2 + 2pq) $$

Doing the same thing for \\(a''\\), we see that we can do the same thing:

$$ a'' = (bp + aq)q + (bq + aq + ap)q + (bq + aq + ap) p $$
$$     = bpq + aq^2 + bq^2 + aq^2 + apq + bpq + apq + ap^2 $$
$$     = bq^2 + bpq + bpq + aq^2 + apq + apq + aq^2 + ap^2 $$
$$     = b(q^2 + 2pq) + a(q^2 + 2pq) + a(q^2 + p^2) $$

We can now see that

$$ p' = p^2 + q^2 $$

$$ q' = q^2 + 2pq $$

Now we can fill in the missing parts of the code:

```scheme
(define (fib-iter a b p q count)
  (cond ((= count 0) b)
        ((even? count)
         (fib-iter a
                   b
                   (+ (square p) (square q))
                   (+ (square q) (* 2 p q))
                   (/ count 2)))
        (else (fib-iter (+ (* b q) (* a q) (* a p))
                        (+ (* b p) (* a q))
                        p
                        q
                        (- count 1)))))
```

###Ex 1.20

For **normal order** evaluation, I found that it results in 18 `remainder` operations.

For **applicative order** evaluation, I found that it results in 4 `remainder` operations.

###Ex 1.21

```scheme
(smallest-divisor 199) ;; 199
(smallest-divisor 1999) ;; 1999
(smallest-divisor 19999) ;; 7
```

###Ex 1.22

First 3 primes \\(> 1000\\): 1009, 1013, 1019

First 3 primes \\(> 10000\\): 10007, 10009, 10037

First 3 primes \\(> 100000\\): 100003, 100019, 100043

First 3 primes \\(> 1000000\\): 1000003, 1000033, 1000037

To compute tmings, I actually needed to use much larger numbers: \\(1000000000\\), and \\(10000000000\\) to get better timings as smaller number resulted in execution times that were too small to measure accurately.  Using large enough values, it does seem to support the the notion that runtime on the machine are proportional to the steps required.  For example, `(timed-prime-test 1000000007)` took about `12ms` and `(timed-prime-test 10000000019)` took about `34ms` which is aboue \\(sqrt{10}\\) times longer, as predicted.

###Ex 1.23

Here are the timings using both methods: checking all numbers, and skipping even numbers.

|               | `(+1 test-divisor)` | `(next test-divisor)` | speedup |
|---------------|-------------------|---------------------|-------|
| 1000000007    | 6                 | 3                   | 2     |
| 1000000009    | 6                 | 3                   | 2     |
| 1000000021    | 7                 | 3.5                 | 2     |
| 10000000019   | 14                | 9.2                 | 1.5   |
| 10000000033   | 20                | 9.3                 | 2.15  |
| 10000000061   | 17.5              | 7.2                 | 2.36  |
| 100000000003  | 41.2              | 15.3                | 2.69  |
| 100000000019  | 45.3              | 18                  | 2.52  |
| 100000000057  | 34.3              | 17                  | 2.0   |
| 1000000000039 | 106.3             | 46.3                | 2.29  |
| 1000000000061 | 102               | 41.5                | 2.46  |
| 1000000000063 | 97.3              | 49.5                | 1.96  |

It does seem like when we skip checking even numbers, we do get about a ~2x speedup.  I would attribute any deviation from things outside of our control, such as the way the JIT interacs with out code, and the way the OS schedules our process.

###Ex 1.24

$$ log{1000000} = log{1000 * 1000} = log{1000} + log{1000} $$

I would expect them to differ by just \\(log{1000}\\).

Note, I can't actually test this because I need larger numbers to get good timings, but `random` in scheme only supports 32-bit values.

###Ex 1.25

No, she is not correct.  Although her version works in theory, it is very inefficient.  This is because it computes the actual value of \\(a^n\\) first, then computes the modulus.  Since we're dealing with very large exponents, this number blows up very quickly, and any operation that we perform ends up taking an extremely long time.

The other version is more efficient because it works by deferring remainder operations on the way down, and halving/decrementing `exp` until we end up with just \\(base \mod m\\).  Then on the way up, we only ever deal with the square of the previous `expmod` operation, or the result multiplied by `base`, so our number never gets much bigger than base, actually always \\(\leq m^2\\).

###Ex 1.26

The reason it is now \\(\Theta(n)\\) is that Louis' version evolves a tree recursive process.  The height of the tree is \\(log_2{n}\\) and each node has a maximum of 2 children, so it becomes a binary tree.  As a binary tree, the number of nodes in the tree is equal to \\(2^{h+1}-1\\), where \\(h\\) is the height, so that becomes \\(2^{log_2{n}+1}-1\\).

$$ 2^{log_2{n}+1}-1 = 2(2^{log_2{n}}) - 1 $$
$$ = 2(n) - 1 $$

So now there are about \\(2n-1\\) nodes in the tree, so it's bounded by \\(\Theta(n)\\).

###Ex 1.27

Using the six Carmichael numbers provided, `fast-prime?` returns `#t` for each of them, so it really does look like they fool the Fermat Test.
