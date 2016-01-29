### Ex 4.1

To ensure that the expression returned by `first-operand` is evaluated first,
we use a `let` binding:

```scheme
(define (list-of-values exps env)
  (if (no-operands? exps)
      '()
      (let ((val (eval (first-operand exps) env)))
        (cons val
              (list-of-values (rest-of-operands exps)
                              env)))))
```

To go the other way, from right to left, we can again use a `let` binding, but
using the `let` to first evaluate the tail of the list, then at the end
`cons`ing the value of the head of the list.

```scheme
(define (list-of-values exps env)
  (if (no-operands? exps)
      '()
      (let ((rest (list-of-values (rest-of-opearands exps)
                                   env)))
        (cons (eval (first-operand exps) env)
              rest))))
```

###Ex 4.2

1. This will not work because the test for what is an application can apply to
   things that really aren't applications:

   ```scheme (define (application? exp) (pair? exp)) ```

   Since all expressions in our language are represented as lists, and list in
   turn are implemented inters of `cons` cells, this test will past for many
   things that are not applications.

   As the book hints, `(define x 3)` would be identified as an application,
   this expression is actually a special form rather than a procedure
   application.

2. We can support the new syntax `(call <op> <arg 1> ... <arg n>)` with just a
   few changes to the application-related procedures:

   ```scheme
   (define (application? exp)
     (tagged-list? exp 'call))

   (define (operator exp) (cadr exp))
   (define (operands exp) (cddr exp))
   (define (no-operands? ops) (null? (cdr exp)))
   (define (first-operand ops) (caddr exp))
   (define (rest-operands ops) (cdddr exp))
   ```

   Now that procedure calls are explicit in the language, we can distinguish
   procedure applications by checking if the tagged list begins with `'call`.
   After that, the selectors are change up a bit to account for the extra
   element at the front of the list.

###Ex 4.3

First, the modified `eval` procedure:

```scheme
(define (eval exp env)
  (cond ((self-evaluating? exp) exp)
        ((variable? exp) (lookup-variable-value exp env))
        (else
         (let ((eval-proc (get 'eval (car exp))))
           (cond (eval-proc (apply eval-proc (list (cdr exp) env)))
                 ((application? exp) (mce-apply (eval (operator exp) env)
                                                (list-of-values (operands exp) env)))
                 (else (error "Unknown expression type -- EVAL" exp)))))))

```

We leave `self-evaluating?` and `variable?` as before because they don't have
tags and it's more cumbersome to make them have them.  Otherwise, we first
check if it has an `'eval` procedure defined in teh global table.  If it does,
then we `apply` the procedure to the arguments and environment.  Also notice we
use `apply` here and not `mce-apply` because the procedure returned is from the
underlying Scheme, not from our own metacircular language.  If we don't find an
`'eval` procedure, the last thing we check is it's a application, again, as
before.

Then, we actually "install" the different evaluation procedures:

```scheme
(define (install-assignment-package)
  (define (assignment-variable args) (car args))
  (define (assignment-value args) (cadr args))
  (put 'eval 'set! (lambda (args env)
                        (set-variable-value! (assignment-variable args)
                                             (eval (assignment-value args) env)
                                             env)))
  'ok)

(define (install-definition-package)
  (define (definition-variable args)
    (if (symbol? (car args))
        (car args)
        (caar args)))
  (define (definition-value args)
    (if (symbol? (car args))
        (cadr args)
        (make-lambda (cdar args) ;; formal parameters
                     (cdr args)))) ;; body
  (define (eval-definition args env)
    (define-variable! (definition-variable args)
                      (definition-value args)
                      env))
  (put 'eval 'define eval-definition)
  'ok)

(define (install-if-package)
  (define (predicate args)
    (car args))
  (define (consequent args)
    (cadr args))
  (define (alternative args)
    (cddr args))
  (define (eval-if args env)
    (if (true? (eval (predicate args) env))
        (eval (consequent args) env)
        (eval (alternative args) env)))
  (put 'eval 'if eval-if)
  'ok)

(define (install-begin-package)
  (define (eval-begin exps env)
    (cond ((last-exp? exps) (eval (first-exp exps) env))
          (else (eval (first-exp exps) env)
                (eval-sequence (rest-exps exps) env))))
  (put 'eval 'begin eval-begin)
  'ok)

(define (install-quote-package)
  (define (eval-quote arg env)
    arg)
  (put 'eval 'quote eval-quote)
  'ok)

(define (install-lambda-package)
  (define (make-procedure parameters body env)
    (list 'procedure parameters body env))
  (define (eval-lambda args env)
    (make-procedure (car args)
                    (cdr args)
                    env))
  (put 'eval 'lambda eval-lambda)
  'ok)
```
