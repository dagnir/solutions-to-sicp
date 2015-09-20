###Ex 3.1

```scheme
(define (make-accumulator initial-value)
  (lambda (n)
    (begin (set! initial-value (+ initial-value n))
	   initial-value)))
```

###Ex 3.2

```scheme
(define (make-monitored f)
  (let ((calls 0))
    (lambda (m)
      (cond ((and (symbol? m) (eq? m 'how-many-calls?)) calls)
	    ((and (symbol? m) (eq? m 'reset-count)) (set! calls 0))
	    (else (begin (set! calls (+ calls 1))
			 (f m)))))))
```

###Ex 3.3
```scheme
(define (make-account balance password)
  (define (withdraw amount)
    (if (>= balance amount)
	(begin (set! balance (- balance amount))
	       balance)
	"Insufficient funds"))
  (define (deposit amount)
    (set! balance (+ balance amount))
    balance)
  (define (dispatch pw m)
    (if (eq? pw password)
	(cond ((eq? m 'withdraw) withdraw)
	      ((eq? m 'deposit) deposit)
	      (else (error "Unknown request -- MAKE-ACCOUNT"
			   m)))
	(lambda (ignored) "Incorrect password")))
  dispatch)
```

The problem wasn't specific about wether we should use `error` to raise the complaint, so I just return a `lambda` that ignores the argument and returns the "Incorrect password" message in the case when the password is wrong.  That way, if we try to do `((acc 'the-wrong-password deposit) 10)`, we just get returned the string, instead of some message about not being able to apply the arguments to the given procedure if we just returned the string.

###Ex 3.4

```scheme
(define (make-account balance password)
  (let ((consecutive-incorrect-pws 0))
    (define (withdraw amount)
      (if (>= balance amount)
	  (begin (set! balance (- balance amount))
		 balance)
	  "Insufficient funds"))
    (define (deposit amount)
      (set! balance (+ balance amount))
      balance)

    (define (call-the-cops)
      (error "Seven consecutive failures to provide correct password.  Calling the cops!  WEE-OOO-WEE-OOO!"))

    (define (inc-incorrect-pws)
      (set! consecutive-incorrect-pws (+ 1 consecutive-incorrect-pws)))

    (define (reset-incorrect-pws)
      (set! consecutive-incorrect-pws 0))

    (define (dispatch pw m)
      (if (eq? pw password)
	  (begin
	    (reset-incorrect-pws)
	    (cond ((eq? m 'withdraw) withdraw)
		  ((eq? m 'deposit) deposit)
		  (else (error "Unknown request -- MAKE-ACCOUNT"
			       m))))
	  (begin
	    (inc-incorrect-pws)
	    (if (> consecutive-incorrect-pws 7)
		(call-the-cops)
		(lambda (ingored) "Incorrect password")))))

    dispatch))
```

Sidenote, it would probably be "secure" if it didn't allow furher access to the account after 7 failures.

###Ex 3.5
```scheme
(define (random-in-range low high)
  (let ((range (- high low)))
    (+ low (random range))))

(define (estimate-integral P x1 x2 y1 y2 trials)
  (define (experiment)
    (let ((x (random-in-range x1 x2))
	  (y (random-in-range y1 y2)))
      (P x y)))
  (let ((portion (monte-carlo trials experiment))
	(rect-area (* (- y2 y1) (- x2 x1))))
    (* 1.0 portion rect-area)))

(define (estimate-pi)
  (define (P x y)
    (<= (+ (square x) (square y)) 1))
  (estimate-integral P -1 1 -1 1 5000))
```

Here we just estimate the area of the circle centered around $$(0,0)$$.  We do indeed get results that are close to $$\pi$$.

###Ex 3.6

```scheme
(define (rand)
  (let ((x random-init))
    (lambda (m)
      (cond ((eq? m 'generate)
	     (set! x (rand-update x))
	     x)
	    ((eq? m 'reset)
	     (lambda (val) (set! x val)))
	    (else (error "Unkown message -- RAND"
			 m))))))
```

One of the changes is easy to miss, so I'll point it out: we define `rand` as a procedure that takes no arguments, and returns a `lambda`.  This is in contrast to the original definition in the book, which defines `rand` *as* a lambda.  The returned `lambda` checks the message that is passed in, and either returns the next value in the sequence (for `'generate`), or returns a 1-arg lambda for `'reset`, that resets the value of `x`.


###Ex 3.7

```scheme
(define (make-joint acc pass 2ndpass)
  (lambda (pw m)
    (if (eq? pw 2ndpass)
	(acc pass m)
	(lambda (ignored) "Incorrect password to joint account"))))
```

`make-account` is unchanged from 3.3.  Here we return a `lambda` that checks if the given password matches the second one given to `make-joint` (the new password), and if it does, passes the message to the `acc` with its password.


###Ex 3.8

```scheme
(define f-calls 0)

(define (f n)
  (if (<= f-calls 0)
      (begin (set! f-calls (+ f-calls 1))
	     n)
      0))
```

Here `f` makes use of globally-defined variable `f-calls` to track how many times it's been called previously.  If it's the first time, it returns `n`, otherwise, it returns `0`.  We can check that it meets the condition in the question with the following:

```scheme
(define (reset) (set! f-calls 0))

(let ((n1 (f 0))
      (n2 (f 1)))
  (+ n1 n2))
=> 0

(reset)

(let ((n1 (f 1))
      (n2 (f 0)))
	  (+ n1 n2))
=> 1
```

On a sidenote, my particular implementation (DrRacket using `#lang scheme`) seems to evalute them from left to right as the expression `(+ (f 0) (f 1))` returns `0`.
