###Ex 2.73
A. Instead of using predicates like `sum?` and `product?`, we create different entries in the dispatch table under the `'deriv` operation, and indexed by the different types of operations we intend to support; i.e. there's a row in the table for the `'deriv` operation, and in that row would be entries for the rules of differentiation that we choose to implement.  We don't have `number?` and `same-variable?` because we don't need to dispatch to anything, they simply inspect the data and return 0 or 1.

B. Here are the implementations for sums and products:

```scheme
(define (deriv-product operands var)
  (let ((multiplier (car operands))
        (multiplicand (cadr operands)))
    (make-sum
     (make-product multiplier (deriv multiplicand var))
     (make-product (deriv multiplier var) multiplicand))))

(define (deriv-sum operands var)
  (let ((addend (car operands))
        (augend (cadr operands)))
    (make-sum (deriv addend var)
              (deriv augend var))))
```

And here are the respective `install` methods:

```scheme
(define (install-product-deriv)
  (put '* 'deriv deriv-product)
  'done)

(define (install-sum-deriv)
  (put '+ 'deriv deriv-sum)
  'done)
```

C.

```scheme
(define (deriv-exponentiation operands var)
  (make-product (make-product (exponent operands)
                              (make-exponentiation (base operands)
                                                   (- (exponent operands) 1)))
                (deriv (base operands) var)))

(define (install-exponentiation-deriv)
  (put '** 'deriv deriv-exponentiation)
  'done)
```

D. Since the indices into the table are inverted, we need to modify the `install-` methods so that the rows in the table are the different operations, and the columns are the operation-specific `deriv` implementations.

###Ex 2.74

A.

```scheme
(define (get-record name file)
  ((get 'get-record (get-file-division file)) name file))
```


The only requirement for each divison's file (assumed to be implemented as a data structure in Scheme) is that they are type-tagged by division, e.g. `'marketing`; we can assume that the tags are the first members of the list.

The `get-record` implementation assumes that each division has an entry in the dispatch table where we can find the division-supplied `get-record` implementation that will work their unique data structure.  Their implementation must also implement this specific interface, which is `(get-record <employee name> <file>)`.

B.

```scheme
(define (get-salary emp)
  ((get 'get-salary (get-employee-division emp)) emp))
```

This is very similar to `get-record`.  Since employee records are themselves sets whose representation varies between divisions, we have to rely on each division to provide their own implentations of `get-salary`.  To make this work, we need records to also be type-tagged by division, possibly by making sure that their implementations of `get-record` add a type tag before returning the record.  All implementations of `get-salary` would need to implement the interface `(get-salary <record>)`.

C.

```scheme
(define (find-employee-record name files)
  (if (null? files)
      (error "Could not find record for employee -- FIND-EMPLOYEE-RECORD" name files)
      (let ((record (get-record name (car files))))
	(if (not (null? record))
	    record
	    (find-employee-record name (cdr files))))))
```

We simply iterate over all the files, and use the generic `get-record` procedure to search for the employee's record in the current file.

D.

Assuming that the new company already has all of the records in a file and has their own implementations of `get-record`, and selectors for their records like `get-salary`, we need to make sure that the file data structure is type-tagged, and all records returned by their `get-record` implentations are also type-tagged.  Finally, they would need to supply all of these methods in a self-contained package so that someone working on the central system can easily install it.

###Ex 2.75

```scheme
(define (make-from-mag-ang r a)
  (define (dispatch op)
    (cond ((eq? op 'real-part) (* r (cos a)))
          ((eq? op 'imag-part) (* r (sin a)))
          ((eq? op 'magnitude) r)
          ((eq? op 'angle) a)
          (else (error "Unknown op -- MAKE-FROM-REAL-IMAG" op))))
  dispatch)
```

###Ex 2.76

####Generic Operations w/ Explicit dispatch
When adding a new type, all of the existing operations must be modified so that they are "aware" of the new type, most likely with the use of a new predicate to identify the new type.

When a new operation is added, only one procedure needs to be created, but again, it needs to be aware of all the possible types that could be passed into it.

####Data-directed style
When adding a new type, the existing methods need not be modified.  The developer(s) creating the new type are responsible for providing their package that has implementations of the "public" interface. Data-directed style also requires that types be type-tagged since the type information is used to lookup the appropriate procedure in the dispatch table.

When adding a new operation, all of the packages must be modified to provide their own specific implementations, but this can be done independently, without the developers needing to worry about types other than their own.  In addition to the type-specific implementations, the generic version of this method must also be created that dispatches to the correct procedure based on the types of the data objects given to it.

####Message-passing style
When adding a new type, no existing types or procedures need to be modified (just like with data-directed style).  The developers need only to supply the constructor, and ensure that the data object implements the needed interfaces.  Note that for message-passing style, we can do away with type-tagging because the data objects created by the constructor already maintain the set of procedures that work their representation.

When adding new operations, the constructor of each type must be modified to support the new operation, but just like in data-directed style, it can be done independently.


In a system where new types are added often, I would message-passing has a slight advantage over data-directed style. It has an advantage because it doesn't require that the types need to be type-tagged, which could cause issues in a large system where different developers could still possibly end up tagging their types with the same tag.  Of course a system could be worked out to prevent this, but it would still require work.  With message-passing style, there is no need for type-tagging because the data object itself maintains the set of procedures that work on it.  Adding a new type is also less work overall for message passing because it doesn't require creating some sort of install procedure to add the new procedures to the dispatch table.

In a system where new procedures are added often, generic operations with explicit dispatch might be the best approach, if the amount of different types is relatively small.  Having to define only a single procedure for an operation could help to ensure that it's available for all possible types.  With data-directed style or message-passing, one would need to modify every package or constructor respectively to make sure all the types support the new procedure.  This task could be made difficult if not one person/team has access to the code for all the packages/types, and so adding the procedure must be coordinated among different teams before the entire system can support a new operation.  In addition, for the data-directed style, a generic implementation of new operation that performs the lookup and dispatch must also be created.
