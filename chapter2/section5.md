###Ex 2.77

The reason that Louis was getting the error from `apply-generic` is because it could not find `real-part` procedure that takes objects tagged as `complex` as arguments, only as either `rectangular` or `polar`.  By adding entries for `'real-part`, `'imag-part`, `'magnitude`, `'angle` that accept `'complex` numbers in the table, `apply-generic` will no longer complain since it will now find an entry, furthermore, if we look at the the procedures that are actually put into the table for these entries, we see that it's actually the generic versions of the respective procedures!  It loops back onto itelf.  However, the next time we call the generic `magnitude`, the `'complex` tag will have been stripped off (by the first `apply-generic`), and this time the outermost (and only) tag `rectangular`.  This will cause `apply-generic` to dispatch to the `magnitude` method defined in the `rectangular` package.

For `(magnitude z)`, `apply-generic` will be invoked twice: the first time when the outer tag is `'complex` cause it to dispatch to `'complex` package version of `magnitude` (which is actually just the generic version), which in turn causes another invokation of `apply-generic`, this time with the outer tag being `rectangular`.

###Ex 2.78

The modifications to the procedures are simple:

```scheme
(define (attach-tag type-tag contents)
  (if (number? contents)
      contents
      (cons type-tag contents)))
```

for `attach-tag`, if `contents` is a number, we ignore the tag, and just return the number.

```scheme
(define (type-tag datum)
  (cond ((number? datum) 'scheme-number)
        ((pair? datum) (car datum))
        (else (error "Bad tagged datum -- TYPE-TAG" datum))))
```

for `type-tag`, if `datum` is a number, we return `scheme-number` rather than `car`ing; `apply-generic` still depends on type tags for the operation arguments, so we can't do away with having a specific type tag for numbers.

```scheme
(define (contents datum)
  (cond ((number? datum) datum)
        ((pair? datum) (cdr datum))
        (else (error "Bad tagged datum -- CONTENTS" datum))))
```

for `contents`, if it's a number, we just return in.

###Ex 2.79

Here is `equ?` implementation for `'scheme-number`:

```scheme
(put 'equ? '(scheme-number scheme-number)
     (lambda (x y) (= x y)))
```

We delegate to the built-in `=` procedure.  Simple.

For `'rational`:

```scheme
(put 'equ? '(rational rational)
     (lambda (x y) (and (equ? (numer x) (numer y))
                        (equ? (denom x) (denom y)))))
```

Notice here that we don't delegate to `=` like we did for `'scheme-number` and instead invoke the generic `equ?` procedure to compare the numerators and denominators.  This is probably unnecessary here, but it's valid, and could be useful to assume that that numerators and denominators could be something other than `'scheme-number`.

for `'complex`:

```scheme
(put 'equ? '(complex complex)
     (lambda (z1 z2) (and (equ? (real-part z1) (real-part z2))
                          (equ? (imag-part z1) (imag-part z2)))))
```

More of the same here.  Notice again that we invoke `equ?` to compare the real and imaginary parts.

###Ex 2.80

The obvious implementation for `'scheme-number`:

```scheme
(put '=zero? '(scheme-number)
     (lambda (x) (= x 0)))
```

For `'rational`:

```scheme
(put '=zero? '(rational)
     (lambda (x) (=zero? (numer x))))
```

Here, again, we invoke the generic `=zero?` method to check that the numerator is zero.

For `'complex`:

```scheme
(put '=zero? '(complex)
     (lambda (z) (and (=zero? (real-part z))
                      (=zero? (imag-part z)))))
```

Again, notice the invokation of `=zero?` within this procedure to test the real and imaginary parts.

###Ex 2.81
A.  This actually results in an infinite recursion.  When we attempt to apply `exp` to two complex numbers, finding no suitable entry in the table, `apply-generic` will attempt to coerce one of the two arguments to the other.  We assume in `apply-generic` that if we can coerce one of the arguments to the other, we get another chance at finding a method that will accept them.  However, by providing procedures that coerce types to their own type, we trick `apply-generic` into thinking that it's possible that with the new arguments, we can find the correct procedure, and thus recursing the "new" (but actually the same) types, but of course it won't find a procedure, and so forth.

B.  No, nothing needs to be changed, it works as is.

C.
```scheme
(define (apply-generic op . args)
  (let ((type-tags (map type-tag args)))
    (let ((proc (get op type-tags)))
      (if proc
          (apply proc (map contents args))
          (if (= (length args) 2)
              (let ((type1 (car type-tags))
                    (type2 (cadr type-tags))
                    (a1 (car args))
                    (a2 (cadr args)))
		(if (not (equal? type1 type2))
		    (let ((t1->t2 (get-coercion type1 type2))
			  (t2->t1 (get-coercion type2 type1)))
		      (cond (t1->t2
			     (apply-generic op (t1->t2 a1) a2))
			    (t2->t1
			     (apply-generic op a1 (t2->t1 a2)))
			    (else
			     (error "No method for these types"
				    (list op type-tags)))))
		    (error "No method for these types"
			   (list op type tags))))
	      ;; not 2 args
              (error "No method for these types"
                     (list op type-tags)))))))
```

###Ex 2.82

```scheme
(define (coerce-all type args)
  (define (iter coerced rest)
    (if (null? rest)
        coerced
        (let ((a (car rest)))
          (if (equal? (type-tag a) type)
              (iter (cons a coerced) (cdr rest))
              (let ((coerce-proc (get-coercion (type-tag a) type)))
                (if coerce-proc
                    (iter (cons (coerce-proc a) coerced) (cdr rest))
                    false))))))
  (let ((res (iter '() args)))
    (if res
        (reverse res)
       res)))

(define (apply-generic op . args)
  (define (try-coercions op types args)
    (define (recur t)
      (if (null? t)
          (error "No method for these types" (list op types))
          (let ((type-tags (repeat (car t) (length args))))
            (let ((proc (get op type-tags)))
              (if proc
                  (let ((coerced (coerce-all (car t) args)))
                    (if coerced
                        (apply proc coerced)
                        (recur (cdr t))))
                  (recur (cdr t)))))))
    (recur types))
  (let ((type-tags (map type-tag args)))
    (let ((proc (get op type-tags)))
      (if proc
          (apply proc (map contents args))
	  (try-coercions op type-tags args)))))
```

This version of `apply-generic` delegates most of the work to the inner `try-coercions` procedure.  It starts off with trying to find a suitable procedure with the unmodified types of the arguments.  If it fails to do so, it takes the types of the arguments, and iterates over them.  For every element, it builds a list of the same length as the arguments, and attempts to find a procedure that accepts the given type and arity.  If it does, it then attemps to coerce all of the arguments to the given type.  If thise coercion step succeeds, it can then apply the procedure to the given arguments.  If the coercion step or a procedure couldn't be find, we recurse on the rest of the list.

The shortcoming of this method is that the arguments are only coerced to the same type as one of the arguments.  If `'add` only worked for two `'complex` numbers, then `(add a b)` where `a` and `b` are any combination of `'scheme-number` and `'rational` would not work because they would never be coerced to `'complex`.  Also, since all arguments are coerced to the same type, we could miss some methods where arguments need to be reordered.  For example, if for some odd reason we only defined addition for `'(rational scheme-number)`, but we give it arguments in the reverse order `'(scheme-number rational)`, then this will also fail because we only try `'(scheme-number rational)`, `'(scheme-number scheme-number)` and `'(rational rational)`.

###Ex 2.83

First, to add to be able to go from `'integer` to `'real`, a few additions had to be made.

First we create a real number package:

```scheme
;; Real Number package
(define (install-real-package)
  (define (tag r)
    (attach-tag 'real r))
  (define epsilon 0.0000001)
  (put 'add '(real real)
       (lambda (r1 r2) (+ r1 r2)))
  (put 'sub '(real real)
       (lambda (r1 r2) (- r1 r2)))
  (put 'mul '(real real)
       (lambda (r1 r2) (* r1 r2)))
  (put 'div '(real real)
       (lambda (r1 r2) (/ r1 r2)))
  (put 'equ? '(real real)
       (lambda (r1 r2) (< (abs (- r1 r2) epsilon))))

  (put 'make 'real
       (lambda (r)
         (tag r)))
  'done)
```

Note that's almost exactly like the the `'scheme-number` package, which I've renamed `'integer`.  This is to allow us to tag numbers with `'real` (as a pair), and will be what allows us to "raise" `'integer`s to `'real`s.

The type tag related procedures have also been modified to check specifically for `integer?` rather then the more general `number?` because attempting to raise something like a rational number like \\(\frac{3}{1}\\) results (as you'll see) in the procedure application `(attach-tag 'real 3)` which, if we were checking for `number?`, causes the previous version to ignore the tag and just return the number.  So rather than being raised to a real number, it would go back to integer.

```scheme
(define (attach-tag type-tag contents)
  (if (equal? type-tag 'integer)
      contents
      (cons type-tag contents)))

(define (type-tag datum)
  (cond ((integer? datum) 'integer)
        ((pair? datum) (car datum))
        (else (error "Bad tagged datum -- TYPE-TAG" datum))))

(define (contents datum)
  (cond ((integer? datum) datum)
        ((pair? datum) (cdr datum))
        (else (error "Bad tagged datum -- CONTENTS" datum))))
```

With that done, we define the generic `raise` procedure:

```scheme
(define (raise x) (apply-generic 'raise x))
```

Then we implement for `'integer`:

```scheme
(put 'raise '(integer)
     (lambda (x) (make-rational x 1)))
```

for `'rational`:

```scheme
(put 'raise '(rational)
     (lambda (x) (make-real (div (numer x) (denom x)))))
```

and for `'real`:

```scheme
(put 'raise '(real)
     (lambda (r) (make-complex-from-real-imag r 0)))
```

### Ex 2.84

```scheme
(define (raise-until type x)
  (let ((r (get 'raise (list (type-tag x)))))
    (cond ((equal? (type-tag x) type) x)
          ((not r) false)
          (else (raise-until type (r (contents x)))))))

(define (apply-generic op . args)
  (let ((type-tags (map type-tag args)))
    (let ((proc (get op type-tags)))
      (if proc
          (apply proc (map contents args))
          (if (= (length args) 2)
              (let ((type1 (car type-tags))
                    (type2 (cadr type-tags))
                    (a1 (car args))
                    (a2 (cadr args)))
                (let ((raised1 (raise-until (type-tag a2) a1))
                      (raised2 (raise-until (type-tag a1) a2)))
                  (cond (raised1
                         (apply-generic op raised1 a2))
                        (raised2
                         (apply-generic op a1 raised2))
                        (else
                         (error "No method for these types"
                                (list op type-tags))))))
              (error "No method for these types"
                     (list op type-tags)))))))
```

This uses the 2-argument version of `apply-generic` for simplicity.

We do something very similar to the version that used explicit coercion procedures.  In this version of `apply-generic`, if no suitable method is found, then we attempt to `raise` one of th arguments to the type of the other, using `raise-until`.  If `raise-until` is can't raise the argument to the given type, it returns `#f`.  If one we're able to raise one argument to the type of the other, then we attempt to apply the generic method to these transformed arguments.

###Ex 2.85

As always, we start by defining the generic `project` operation:

```scheme
(define (project x)
  (let ((p (get 'project (list (type-tag x)))))
    (if p
	(p (contents x))
	false)))
```

Here we query the table for a suitable `'project` implementation, and if one is found, we apply it to the contents of `x`.

Now we can define the type-specific implementations:

for `'rational`:
```scheme
(define (project x) (make-integer (round (/ (numer x) (denom x)))))
(put 'project '(rational) project)
```

for `'real`:
```scheme
(define (project r) (make-rational (round r) 1))
(put 'project '(real) project)
```

for `'complex`:
```scheme
(define (project z) (real-part z))
(put 'project '(complex) project)
```

Obviously, we don't don't define one for the `'integer` package, which is why we need the check in `project`.

Now we can define `drop` by implementing it as described in the book: project the argument, then raise it, and check for equality between the result and the original.  If they're equal, try to project further.

```scheme
(define (drop x)
  (let ((projected (project x)))
    (if projected
	(if (equ? x (raise projected))
	    (drop projected)
	    x)
	x)))
```

###Ex 2.86

The solution is to define the operations on complex numbers in terms of generic operations like `add` and `mul`.  This way, the calculations involved in the procedures like `real-part` and `angle` are made independent of the type fo the numbers they're operating on, as long as the operation is defined for that particular type.

Here are the modified complex number packages that make use only of the generic arithmetic operations:

the `'rectangular` package:

```scheme
(define (install-rectangular-package)
  ;; internal procedures
  (define (real-part z) (car z))
  (define (imag-part z) (cdr z))
  (define (make-from-real-imag x y) (cons x y))
  (define (magnitude z)
    (square-root (add (square (real-part z))
		      (square (imag-part z)))))
  (define (angle z)
    (arctan (imag-part z) (real-part z)))
  (define (make-from-mag-ang r a)
    (cons (mul r (cos a)) (mul r (sin a))))
  ;; interface to the rest of the system
  (define (tag x) (attach-tag 'rectangular x))
  (put 'real-part '(rectangular) real-part)
  (put 'imag-part '(rectangular) imag-part)
  (put 'magnitude '(rectangular) magnitude)
  (put 'angle '(rectangular) angle)
  (put 'make-from-real-imag 'rectangular
       (lambda (x y) (tag (make-from-real-imag x y))))
  (put 'make-from-mag-ang 'rectangular
       (lambda (r a) (tag (make-from-mag-ang r a))))
  'done)
  ```

the `'polar` package:

```scheme
(define (install-polar-package)
  ;; internal procedures
  (define (magnitude z) (car z))
  (define (angle z) (cdr z))
  (define (make-from-mag-ang r a) (cons r a))
  (define (real-part z)
    (mul (magnitude z) (cosine (angle z))))
  (define (imag-part z)
    (mul (magnitude z) (sine (angle z))))
  (define (make-from-real-imag x y)
    (cons (square-root (add (square x) (square y)))
          (arctan y x)))
  ;; interface to the rest of the system
  (define (tag x) (attach-tag 'polar x))
  (put 'real-part '(polar) real-part)
  (put 'imag-part '(polar) imag-part)
  (put 'magnitude '(polar) magnitude)
  (put 'angle '(polar) angle)
  (put 'make-from-real-imag 'polar
       (lambda (x y) (tag (make-from-real-imag x y))))
  (put 'make-from-mag-ang 'polar
       (lambda (r a) (tag (make-from-mag-ang r a))))
  'done)
```

and finally the `'complex` package:

```scheme
(define (install-complex-package)
  ;; imported procedures from rectangular and polar packages
  (define (make-from-real-imag x y)
    ((get 'make-from-real-imag 'rectangular) x y))
  (define (make-from-mag-ang r a)
    ((get 'make-from-mag-ang 'polar) r a))
  ;; internal procedures
  (define (add-complex z1 z2)
    (make-from-real-imag (add (real-part z1) (real-part z2))
                         (add (imag-part z1) (imag-part z2))))
  (define (sub-complex z1 z2)
    (make-from-real-imag (sub (real-part z1) (real-part z2))
                         (sub (imag-part z1) (imag-part z2))))
  (define (mul-complex z1 z2)
    (make-from-mag-ang (mul (magnitude z1) (magnitude z2))
                       (add (angle z1) (angle z2))))
  (define (div-complex z1 z2)
    (make-from-mag-ang (div (magnitude z1) (magnitude z2))
                       (sub (angle z1) (angle z2))))
  (define (project z) (make-real (real-part z)))
  ;; interface to rest of the system
  (define (tag z) (attach-tag 'complex z))
  (put 'add '(complex complex)
       (lambda (z1 z2) (tag (add-complex z1 z2))))
  (put 'sub '(complex complex)
       (lambda (z1 z2) (tag (sub-complex z1 z2))))
  (put 'mul '(complex complex)
       (lambda (z1 z2) (tag (mul-complex z1 z2))))
  (put 'div '(complex complex)
       (lambda (z1 z2) (tag (div-complex z1 z2))))
  (put 'equ? '(complex complex)
       (lambda (z1 z2) (and (equ? (real-part z1) (real-part z2))
                            (equ? (imag-part z1) (imag-part z2)))))
  (put 'project '(complex) project)
  (put 'make-from-real-imag 'complex
       (lambda (x y) (tag (make-from-real-imag x y))))
  (put 'make-from-mag-ang 'complex
       (lambda (r a) (tag (make-from-mag-ang r a))))
  ;; selectors
  (put 'real-part '(complex) real-part)
  (put 'imag-part '(complex) imag-part)
  (put 'magnitude '(complex) magnitude)
  (put 'angle '(complex) angle)
  'done)
```

In addition to `add`, `sub`, `mul`, and `div`, we also define the following additional generic procedures: `sine`, `cosine`, `arctan`, `square`, and `square-root`.

additional procedures for `'integer`:

```scheme
(put 'square '(integer)
     (lambda (x) (tag (* x x))))
(put 'sine '(integer)
     (lambda (x) (make-real (sin x))))
(put 'cosine '(integer)
     (lambda (x) (make-real (cos x))))
(put 'arctan '(integer integer)
     (lambda (a b) (make-real (atan a b))))
(put 'square-root '(integer)
     (lambda (x) (make-real (sqrt x))))
```

additional procedures for `'rational`:

```scheme
  (define (square-rat x) (make-rat (* (numer x) (numer x))
				   (* (denom x) (denom x))))
  (define (sine-rat x) (sin (/ (numer x)
			       (denom x))))
  (define (cosine-rat x) (cos (/ (numer x)
				 (denom x))))
  (define (arctan-rat a b) (atan (/ (numer a)
				    (denom a))
				 (/ (numer b)
				    (denom b))))
  (define (square-root-rat x) (make-rat (sqrt (numer x))
					(sqrt (denom x))))

  (put 'square '(rational)
       (lambda (x) (tag (square-rat x))))
  (put 'sine '(rational)
       (lambda (x) (make-real (sine-rat x))))
  (put 'cosine '(rational)
       (lambda (x) (make-real (cosine-rat x))))
  (put 'arctan '(rational rational)
       (lambda (x y) (make-real (arctan-rat x y))))
  (put 'square-root '(rational)
       (lambda (x) (tag (square-root-rat x))))
```

and for `'real`:

```scheme
(put 'square '(real)
     (lambda (x) (tag (* x x))))
(put 'sine '(real)
     (lambda (x) (tag (sin x))))
(put 'cosine '(real)
     (lambda (x) (tag (cos x))))
(put 'arctan '(real real)
     (lambda (a b) (tag (atan a b))))
(put 'square-root '(real)
     (lambda (x) (tag (sqrt x))))
 ```

###Ex 2.87
Inside the `'polynomial` package:

```scheme
  (define (=zero?-poly L)
    (if (=zero? (coeff (first-term L)))
	(=zero?-poly (rest-terms L))
	false))

  (put '=zero? 'polynomial =zero?-poly)
```

###Ex 2.88
The problem hints at the answer: by defining a generic `negate` procedure, we can implement `sub-poly` in terms of `add-poly` by adding negation of the augend to the addend.  We need to define `negate` for all of our packages, as well as a generic one:

for `'integer`:

```scheme
 (put 'negate '(integer)
      (lambda (x) (tag (- x))))
```

for `'rational`:

```scheme
(define (negate-rat x) (make-rat (- (numer x))
				   (denom x)))
(put 'negate '(rational)
     (lambda (x) (tag (negate-rat x))))
```

for `'real`:

```scheme
(put 'negate '(real)
     (lambda (x) (tag (- x))))
```

for `'complex`:

```scheme
(define (negate-complex z)
  (make-from-real-imag (negate (real-part z))
			 (negate (imag-part z))))
(put 'negate '(complex)
     (lambda (z) (tag (negate-complex z))))
```

for `'polynomial`, we walk the terms list, and negate each one:

```scheme
(define (negate-terms termlist)
  (if (empty-termlist? termlist)
	termlist
	(adjoin-term (negate-term (first-term termlist))
		     (negate-terms (rest-terms termlist)))))
(define (negate-poly p)
  (make-poly (variable p) (negate-terms (term-list p))))
```

Finally, we implement `sub` for the `'polynomial` package:

```scheme
(define (sub-poly p1 p2)
  (if (same-variable? (variable p1) (variable p2))
	(add-poly p1 (negate-poly p2))
	(error "Polys not in same var -- SUB POLY")))
(put 'sub '(polynomial polynomial)
     (lambda (p1 p2) (tag (sub-poly p1 p2))))
```

###Ex 2.89

```scheme
(define (dense-coeff L)
  (car L))

(define (dense-order L)
  (- (length L) 1))
```

For the dense representations, the terms list consists of only of the coeffiecients, and the order of the current term--which is at the head of the list--is the 1 less than the length of the list.  So for `coeff`, we simply return the car of the list, and and for `order` we return one less than the length of the list.

###Ex 2.90
To make this work, we need to add two new packages, one for each of the types of terms lists we can have: `'dense` and `'sparse`.  Luckily, the `'polynomial` package is written in such a way that whenever it must perform an operation on a terms list, it's always done through a procedure; that it, it never accesses any element of the terms list directly via something like `car`.  This means that to make the `'polynomial` package work with either of the representations we need only to implement these methods for each package, then define generic methods that dispatch to these representation-specific implementations.

Here is the `'dense` package, implements the term list operations for dense polynomials:

```scheme
(define (install-dense-package)
  (define (tag terms-list)
    (attach-tag 'dense terms-list))
  (define (coeff termslist)
    (car termslist))
  (define (order termslist)
    (- (length termslist) 1))
  (define (first-term termslist)
    (list (order termslist) (coeff termslist)))
  (define (rest-terms termslist) (cdr termslist))
  (define (adjoin-term-dense term termslist)
    (if (=zero? (cadr term))
	termslist
	(let ((ord (car term)))
	  (if (= ord (+ 1 (order termslist)))
	      (cons (cadr term) termslist)
	      (adjoin-term-dense term (cons 0 termslist))))))
  (put 'first-term '(dense) first-term)
  (put 'rest-terms '(dense)
       (lambda (L) (tag (rest-terms L))))
  (put 'adjoin-term 'dense
       (lambda (term L) (tag (adjoin-term-dense term L))))
  (put 'empty-termlist 'dense
       (lambda (termslist) (null? termslist)))
  (put 'make 'dense
       (lambda (terms) (tag terms)))
  'done)
```

Most of the procedures here are pretty straightforward.  The most interesting is the `adjoin-term-dense` procedure, which is the implementation of `adjoin-term` for dense term lists.  Since the order of a term is implicit in its position in the list, we need to make sure that before `cons`ing to the terms list, it will be in the right place.  The "right place" is when the length of list which we're adjoing to is 1 *less* than the order of the term being adjoined.  If not, we "pad" the list with a 0, and recurse.

Note that we use `dense` and `coeff` procedures that we defined in Exercise 2.89.


Here is `'sparse` package:

```scheme
(define (install-sparse-package)
  (define (tag terms-list)
    (attach-tag 'sparse terms-list))
  (define (make-term order coeff) (list order coeff))
  (define (order term)
    (car term))
  (define (coeff term)
    (cadr term))
  (define (first-term L)
    (car L))
  (define (rest-terms L)
    (cdr L))
  (define (adjoin-term term term-list)
    (if (=zero? (coeff term))
	term-list
	(cons term term-list)))
  (put 'first-term '(sparse) first-term)
  (put 'rest-terms '(sparse)
       (lambda (L) (tag (rest-terms L))))
  (put 'adjoin-term 'sparse
       (lambda (term L) (tag (adjoin-term term L))))
  (put 'empty-termlist 'sparse
       (lambda (termslist) (null? termslist)))
  (put 'make 'sparse
       (lambda (terms) (tag terms)))
  'done)
```

This is even simpler than the `'dense` package, and all of the procedures here are the same as from the original polynomial package.

Then we define the generic procedures that the `'polynomial` package will use:

```scheme
(define (make-polynomial-from-dense-termslist var terms)
  (let ((termslist ((get 'make 'dense) terms)))
    ((get 'make 'polynomial) var termslist)))

(define (make-polynomial-from-sparse-termslist var terms)
  (let ((termslist ((get 'make 'sparse) terms)))
    ((get 'make 'polynomial) var termslist)))

(define (first-term termslist) (apply-generic 'first-term termslist))
(define (rest-terms termslist) (apply-generic 'rest-terms termslist))
(define (adjoin-term term L) ((get 'adjoin-term (type-tag L)) term (contents L)))
(define (empty-termlist? x) ((get 'empty-termlist (type-tag x)) (contents x)))
(define (variable x) ((get 'variable (type-tag x)) (contents x)))
```

A note about `the-empty-termlist`: Since we have two representations of term lists now, we can't simply define it as `'()`.  That won't work because it doesn't have a type tag.  So instead we redefine it in the `'polynomial` package as an empty list of type `'parse`, though we could easily just define it as a `'dense` list.

###Ex 2.91

```scheme
  (define (div-terms L1 L2)
    (if (empty-termlist? L1)
	(list (the-empty-termlist) (the-empty-termlist))
	(let ((t1 (first-term L1))
	      (t2 (first-term L2)))
	  (if (> (order t2) (order t1))
	      (list (the-empty-termlist) L1)
	      (let ((new-c (div (coeff t1) (coeff t2)))
		    (new-o (- (order t1) (order t2))))
		(let ((rest-of-result
		       (div-terms (sub-terms L1 (mul-terms L2
							   (adjoin-term (make-term new-o new-c)
									(the-empty-termlist))))
				  L2)))
		  (list (adjoin-term (make-term new-o new-c) (car rest-of-result))
			(cadr rest-of-result))
		))))))
```

It's just a direct implementation of the description of the algorithm described in the book.  We have to remember that `rest-of-result` is a pair containing the quotient and remainder, so when we're adding term for the quotient (comprising of `new-o` and `new-c`), we have to "unwrap" the result, stick the term in the first list, then return the result.

###Ex 2.92

I chose to skip this after thinking about it for about half an hour.  To be honest, I'm still not very sure about how I would tackle this problem.  Maybe I'll come back to it at a later time.

###Ex 2.93

Here is the modified rational package.  To make this and the remaining exercises easier, I decided to remove the `drop` step in `apply-generic`, because it's not clear how to `project` a rational number when we can't assume that it's regular number.

```scheme
(define (install-rational-package)
  ;; internal procedures
  (define (numer x) (car x))
  (define (denom x) (cdr x))
  (define (make-rat n d)
    (cons n d))
  (define (add-rat x y)
    (make-rat (add (mul (numer x) (denom y))
		   (mul (numer y) (denom x)))
              (mul (denom x) (denom y))))
  (define (sub-rat x y)
    (make-rat (sub (mul (numer x) (denom y))
		   (mul (numer y) (denom x)))
              (mul (denom x) (denom y))))
  (define (mul-rat x y)
    (make-rat (mul (numer x) (numer y))
              (mul (denom x) (denom y))))
  (define (div-rat x y)
    (make-rat (mul (numer x) (denom y))
              (mul (denom x) (numer y))))
  (define (square-rat x) (make-rat (mul (numer x) (numer x))
				   (mul (denom x) (denom x))))
  (define (sine-rat x) (sine (div (numer x)
				  (denom x))))
  (define (cosine-rat x) (cosine (div (numer x)
				      (denom x))))
  (define (arctan-rat a b) (arctan (div (numer a)
				      (denom a))
				 (div (numer b)
				      (denom b))))
  (define (square-root-rat x) (make-rat (square-root (numer x))
					(square-root (denom x))))
  (define (negate-rat x) (make-rat (sub 0 (numer x))
				   (denom x)))
  (define (=zero?-rat x) (equ? (numer x) 0))

;; interface to rest of the system
  (define (tag x) (attach-tag 'rational x))
  (put 'add '(rational rational)
       (lambda (x y) (tag (add-rat x y))))
  (put 'sub '(rational rational)
       (lambda (x y) (tag (sub-rat x y))))
  (put 'mul '(rational rational)
       (lambda (x y) (tag (mul-rat x y))))
  (put 'div '(rational rational)
       (lambda (x y) (tag (div-rat x y))))
  (put 'square '(rational)
       (lambda (x) (tag (square-rat x))))
  (put 'sine '(rational)
       (lambda (x) (sine-rat x)))
  (put 'cosine '(rational)
       (lambda (x) (cosine-rat x)))
  (put 'arctan '(rational rational)
       (lambda (x y) (arctan-rat x y)))
  (put 'square-root '(rational)
       (lambda (x) (tag (square-root-rat x))))
  (put 'negate '(rational)
       (lambda (x) (tag (negate-rat x))))
  (put 'equ? '(rational rational)
       (lambda (x y) (and (equ? (numer x) (numer y))
                          (equ? (denom x) (denom y)))))
  (put '=zero? 'rational =zero?-rat)
  (put 'make 'rational
       (lambda (n d) (tag (make-rat n d))))
  'done)
```

With these changes, we can execute the following and get the expected results:

```scheme
(define p1 (make-polynomial-from-sparse-termslist 'x '((2 1)(0 1))))
(define p2 (make-polynomial-from-sparse-termslist 'x '((3 1)(0 1))))
(define rf (make-rational p2 p1))

(add rf rf)
```

###Ex 2.94
Here is the implementation of `greatest-common-divisor`:

```scheme
(define (greatest-common-divisor a b)
  (if (and (equal? 'polynomial (type-tag a))
	   (equal? 'polynomial (type-tag b)))
      (apply-generic 'greatest-common-divisor a b)
      (gcd a b)))
```

The operation for `'polynomial` is defined in the `'polynomial` package:

```scheme
  (define (gcd-terms a b)
    (if (empty-termlist? b)
	a
	(gcd-terms b (remainder-terms a b))))
  (define (remainder-terms a b)
    (cadr (div-terms a b)))
  (define (gcd-poly p1 p2)
    (if (same-variable? (variable p1) (variable p2))
	(make-poly (variable p1)
		   (gcd-terms (term-list p1)
			      (term-list p2)))
	(error "Polys not in same var -- GCD-POLY"
	       (list p1 p2))))

  (put 'greatest-common-divisor '(polynomial polynomial)
       (lambda (p1 p2) (tag (gcd-poly p1 p2))))
```

Executing the following, we do indeed get the correct GCD:

```scheme
(define p1 (make-polynomial-from-sparse-termslist 'x '((4 1) (3 -1) (2 -2) (1 2))))
(define p2 (make-polynomial-from-sparse-termslist 'x '((3 1) (1 -1))))
(greatest-common-divisor p1 p2)
```

###Ex 2.95

Sure enough, evaluating the expressions gives us a result that is not equal to `p1`, but seems to be off by some constant.  As the problem states, the reason we don't get integer coefficients like we would hope for is that in the process of dividing to find the GCD, the intermediate divisions produce rational numbers, that progate to the result.

```scheme
(define p1 (make-polynomial-from-sparse-termslist 'x '((2 1) (1 -2) (0 1))))
(define p2 (make-polynomial-from-sparse-termslist 'x '((2 11) (0 7))))
(define p3 (make-polynomial-from-sparse-termslist 'x '((1 13) (0 5))))
(define q1 (mul p1 p2))
(define q2 (mul p1 p3))

(greatest-common-divisor q1 q2)
```

###Ex 2.96

A.  Here's the implementation of `psuedoremainder-terms`:

```scheme
  (define (pseudoremainder-terms p q)
    (let ((o1 (order (first-term p)))
	  (o2 (order (first-term q)))
	  (c (coeff (first-term q))))
      (let ((f (expt c (+ 1 o1 (- o2)))))
	(cadr (div-terms (mul-terms p (adjoin-term (make-term 0 f) (the-empty-termlist)))
			 q)))))
```

Sure enough, by switching out `remainder-terms` with `pseudoremainder-terms`, we now get integer coefficients:

```scheme
(greatest-common-divisor q1 q2)
=> (polynomial x sparse (2 1458) (1 -2916) (0 1458))
```

B.  By finding the GCD of all the (assumed to be integer) coefficients, we can reduce the quotient to lowest terms by diving each term by the GCD:

```scheme
  (define (coeff-gcd termlist)
    (define (iter c l)
      (if (empty-termlist? l)
	  c
	  (iter (gcd c (coeff (first-term l)))
		(rest-terms l))))
    (iter (coeff (first-term termlist))
	  (rest-terms termlist)))

  (define (gcd-terms a b)
    (if (empty-termlist? b)
	(let ((d (coeff-gcd a)))
	  (car (div-terms a (adjoin-term (make-term 0 d) (the-empty-termlist)))))
	(gcd-terms b (pseudoremainder-terms a b))))
  (define (remainder-terms a b)
    (cadr (div-terms a b)))
```

Now, finally, `(greatest-common-divisor q1 q2)` will evaulate to exactly `p1`:

```scheme
(greatest-common-divisor q1 q2)
=> (polynomial x sparse (2 1) (1 -2) (0 1))
```

###Ex 2.97

A.  Here is the implementation of `reduce-terms`:

```scheme
  (define (reduce-terms n d)
    (let ((the-gcd (gcd-terms n d))
	  (f (integerizing-factor n d)))
      (let ((n2 (mul-terms-by-term n (make-term 0 f)))
	    (d2 (mul-terms-by-term d (make-term 0 f))))
	(let ((n3 (car (div-terms n2 the-gcd)))
	      (d3 (car (div-terms d2 the-gcd))))
	  (list (reduce-coeffs n3)
		(reduce-coeffs d3))))))
```

It's pretty much a step-by-step translation of the algorithm outlined in the book.  First we compute the GCD of `n` and `d`, then the integerizing factor `f`, multiply `n` and `d` by `f` to get `n2` and `d2`, divide both of them by `the-gcd`, and then finally reduce their coefficients using `reduce-coeffs`.

```scheme
  (define (integerizing-factor n d)
    (let ((o1 (order (first-term n)))
	  (o2 (order (first-term d)))
	  (c (coeff (first-term d))))
      (expt c (+ 1 o1 (- o2)))))

  (define (reduce-coeffs a)
    (let ((c-gcd (coeff-gcd a)))
      (car (div-terms-by-term a (make-term 0 c-gcd)))))

  (define (reduce-poly p1 p2)
    (if (same-variable? (variable p1) (variable p2))
	(let ((reduced (reduce-terms (term-list p1)
				     (term-list p2))))
	  (list (make-poly (variable p1) (car reduced))
		(make-poly (variable p1) (cadr reduced))))
	(error "Polys not in same var -- REDUCE-POLY"
	       (list p1 p2))))
```

`mul-terms-by-term` is just a helper function to multiply a terms list by a single term:

```scheme
  (define (mul-terms-by-term termlist term)
    (mul-terms termlist (adjoin-term term (the-empty-termlist))))
```

B. The generic `reduce`:

```scheme
(define (reduce n d)
  (if (and (equal? 'polynomial (type-tag n))
	   (equal? 'polynomial (type-tag d)))
      (apply-generic 'reduce n d)
      (reduce-integers n d)))
```

installing `reduce-poly` for reducing polynomials:

```scheme
  (put 'reduce '(polynomial polynomial)
       (lambda (p1 p2)
	 (let ((result (reduce-poly p1 p2)))
	   (list (tag (car result)) (tag (cadr result))))))
```

using `reduce` within `make-rat`:

```scheme
  (define (make-rat n d)
    (let ((reduced (reduce n d)))
      (cons (car reduced) (cadr reduced))))
```

With these changes in place, we do indeed get the result reduced to lowest terms when executing the following:

```scheme
(define p1 (make-polynomial-from-sparse-termslist 'x '((1 1)(0 1))))
(define p2 (make-polynomial-from-sparse-termslist 'x '((3 1)(0 -1))))
(define p3 (make-polynomial-from-sparse-termslist 'x '((1 1))))
(define p4 (make-polynomial-from-sparse-termslist 'x '((2 1)(0 -1))))

(define rf1 (make-rational p1 p2))

(define rf2 (make-rational p3 p4))

(add rf1 rf2)
=> (rational (polynomial x sparse (3 1) (2 2) (1 3) (0 1)) polynomial x sparse (4 -1) (3 -1) (1 1) (0 1))
```
