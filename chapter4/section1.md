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
