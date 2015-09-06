###Ex 2.17

```scheme
(define (last-pair list)
  (if (null? (cdr list))
      list
      (last-pair (cdr list))))
```

To find the last pair, we move through the list from left to right until the second element of the current list is `nil`, which indicates the end of the list, and so the `car` of the list is the last element.

###Ex 2.18

```scheme
(define (reverse list)
  (define (reverse-iter l1 l2)
    (if (null? l1)
        l2
        (reverse-iter (cdr l1) (cons (car l1) l2))))
  (reverse-iter list '()))
```

A simple way to think about reversing a list is to move from left to right through one list, taking the left-most element (using `car`), and adding it to the front of a second list (this is exactly how one would reverse a sequence using stacks).  `reverse-iter` does just that.

###Ex 2.19

```scheme
(define (cc amount coin-values)
  (define (first-denomination coins)
    (car coins))
  (define (except-first-denomination coins)
    (cdr coins))
  (define (no-more? coins)
    (null? coins))
  (cond ((= amount 0) 1)
        ((or (< amount 0) (no-more? coin-values)) 0)
        (else
         (+ (cc amount
                (except-first-denomination coin-values))
            (cc (- amount
                   (first-denomination coin-values))
                coin-values)))))
```

No, the order of the values in `coin-values` does not affect the answer produced by `cc`.  As we found in Excercise 1.14 in Section 1, `cc` essentially does a brute force search of all the possible ways to combine the given denominations to make up `amount`, and this search is independent of the order of the denominations (since it will check all of them).

###Ex 2.20

```scheme
(define (same-parity a . nums)
  (define (recur n l)
    (if (null? l)
        '()
        (let ((h (car l))
              (t (cdr l)))
          (if (= (remainder a 2) (remainder h 2))
              (cons h (recur n t))
              (recur n t)))))
  (cons a (recur a nums)))
```

The goal is to filter the elements of `nums` so that we keep only the ones that have same parity as `a`--that is, if `a` is even, we keep only the evens, and if it's odd, we keep only the `odd` ones.  To do this, we recursively build the resulting list, adding only elements that have the same parity.

Note how this is written as a recursive procedure, rather than iterative.  This is done to preserve the order in which the elements in `nums` appear.  It would be possible to do it in an iterative manner as well, but that would not be as efficient because it requires appending each element to the end of the list.  We could also build the list backwards, then `reverse` at the end, but that's another \\(O(n)\\) operations we can avoid.

###Ex 2.21

The first version:


```scheme
(define (square-list items)
  (if (null? items)
      '()
      (cons (square (car items)) (square-list (cdr items)))))
```

The second version:

```scheme
(define (square-list items)
  (map (lambda (x) (square x)) items))
```

###Ex 2.22

The reason the answer is in reverse order is because the process traverses the given list from left to right, prepending each result to the list, which results in the earlier computations appearing further to the right of the list than the most recently computed ones.

With Louis's second attempt, technically the results do appear in the correct order, but the structure of the result is not correct.  Rather than having the form head:[tail], it has the form [tail]:head.  Now the structure is reversed!  The reason for this is simple: Louis knows that results that have already been computed should appear earlier in the list, so he makes them the first element of the pair, putting the element just computed at the end of the resulting list.  In a way, this is correct, but violates the structure of a list, where the `car` of the list should be an element of the list, rather than a sublist.

###Ex 2.23

```scheme
(define (for-each f l)
  (define (iter items)
    (if (null? items)
        #t
        ((lambda ()
           (f (car items))
           (iter (cdr items))))))
  (iter l))
```


###Ex 2.24

```scheme
(list 1 (list 2 (list 3 4)))
```

as printed by the interpreter:

```scheme
(1 (2 (3 4)))
```

###Ex 2.25
I found that drawing the box-and-pointer representation of the list made this exercise a lot easier.  After creating the drawing, determining the sequence of `car`s and `cdr`s was just a matter of starting from desired box (i.e. the one that contained `7`), and seeing whether it was the first or second element in the pair to determine whether i needed a `car` or a `cdr`.  Then, we just follow the pointer back to the box that points to the pair that that contained the `7` and see whether we need a `car` or `cdr` to access it, and so on.  Note that by going backwards, this is also the exact sequence we need to type out; if we started at the beginning of the list and worked our way forward, we would need to reverse the sequence.

```scheme
(define l1 (list (1 3 (list 5 7) 9)))

l1
=> (1 3 (5 7) 9)

(car (cdr (car (cdr (cdr l1)))))
=> 7

(define l2 (list (list 7)))

l2
=> ((7))

(car (car l2))
=> 7

(define l3 (list 1 (list 2 (list 3 (list 4 (list 5 (list 6 7)))))))

l3
=> (1 (2 (3 (4 (5 (6 7))))))

(car (cdr (car (cdr (car (cdr (car (cdr (car (cdr (car (cdr l))))))))))))
=> 7
```

###Ex 2.26

```scheme
(define x (list 1 2 3))

x
=> (cons 1 (cons 2 (cons 3 '())))

(define y (list 4 5 6))

y
=> (cons 4 (cons 5 (cons 6 '())))

(append x y)
=> (1 2 3 4 5 6)

(cons x y)
=> (cons (cons 1 (cons 2 (const 3 '()))) (cons 4 (cons 5 (cons 6 '()))))

(list x y)
=> (cons (cons 1 (cons 2 (cons 3 '()))) (cons (cons 4 (cons 5 (cons 6 '()))) '()))
```

Note the difference between `(cons x y)` and `(list x y)`.  The first produces a cons cell whose `car` is `x` and `cdr` is `x`, whereas the second one produces a nesting of `cons`: `(cons x (cons y nil))`.


###Ex 2.27

```scheme
(define (deep-reverse list)
  (define (iter l1 l2)
    (cond ((null? l1) l2)
          ((pair? (car l1)) (iter (cdr l1) (cons (deep-reverse (car l1)) l2)))
          (else (iter (cdr l1) (cons (car l1) l2)))))
  (iter list '()))
```

The modification made is to detect when the `car` of the list is itself a list.  If it is, we recurse on the list to reverse it, before adding it to the resulting list.

###Ex 2.28

```scheme
(define (fringe t)
  (cond ((null? t) '())
        ((not (pair? t)) (list t))
        (else (append (fringe (car t))
                      (fringe (cdr t))))))
```

The implementation of `fringe` is very similar to the implementation given for `count-leaves`, both of which are just implementations of depth-first-search (DFS), where we move left to right.


###Ex 2.29

A.

```scheme
(define left-branch car)

(define (right-branch mobile)
  (car (cdr mobile)))

(define branch-length car)

(define (branch-structure branch)
  (car (cdr branch)))
```

`right-branch` and `branch-structure` can also be simplied by defining them simply as `cadr`.


B.

```scheme
(define (branch-weight b)
  (let ((s (branch-structure b)))
    (if (not (pair? s))
        s
        (total-weight s))))

(define (total-weight mobile)
  (let ((l (left-branch mobile))
        (r (right-branch mobile)))
    (+ (branch-weight l)
       (branch-weight r))))
```

To calculate the total weight of a mobile, I use two mutually recursive procedures: `branch-weight` and `total-weight`.  The total weight of mobile is the sum of the weight of its two branches, and the weight of any branch is either just the weight hanging off of it, or the weight of the (sub)mobile hanging off it.

C.

```scheme
(define (torque b)
  (* (branch-length b)
     (branch-weight b)))

(define (balanced mobile)
  (= (torque (left-branch mobile))
     (torque (right-branch mobile))))
```

D.

Only `right-branch` and `branch-structure` would need to be changed.  Both would need to be changed to be simply `define`s for `cdr`.  Everything else can be left as is, since only these procedures are affected by the change in structure.

###Ex 2.30

Implemented directly:


```scheme
(define (square-tree tree)
  (cond ((null? tree) '())
	((not (pair? tree)) (square tree))
	(else (cons (square-tree (car tree))
		    (square-tree (cdr tree))))))
```

Using `map`:

```scheme
(define (square-tree tree)
  (map (lambda (sub-tree)
         (if (pair? sub-tree)
             (square-tree sub-tree)
             (square sub-tree)))
       tree))
```

There really isn't much difference between the two, in fact, LOC-wise, the direct one is one line shorter.  Both follow the same structure: if the curent element in the list is a pair, it's a sub-tree, so we recurse on it, otherwise, we just square the value, and move on.


###Ex 2.31

```scheme
(define (tree-map f tree)
  (map (lambda (sub-tree)
         (if (pair? sub-tree)
             (tree-map f sub-tree)
             (f sub-tree)))
       tree))
```


###Ex 2.32

Here is the complete procedure that generates all of the subsets:

```scheme
(define (subsets s)
  (if (null? s)
      (list '())
      (let ((rest (subsets (cdr s))))
	(append rest (map (lambda (ss)
			    (cons (car s) ss))
			  rest)))))
```

Here is how it works: the procedure generates all of the subsets by taking a list, (recursively) generating all of the subsets of the tail of the list, prepending the head of the list to all of the returned subsets, then appending the subsets of the the tail (without the head) to that list.

Since all of those subsets were generated using a set that did not include the head, it is always safe to add it to all of the returned subsets.  Note also that for any set \\(A\\) which includes the element \\(a\\), there is a corresponding set \\(A'\\) that contains all of the elements of \\(A\\) except for \\(a\\) (precisely because at some point, we created \\(A\\) by adding \\(a\\) to \\(A'\\)). this guarantees that we have all of the subsets.

###Ex 2.33

```scheme
(define (map p sequence)
  (accumulate (lambda (x y) (cons (p x) y))
              '()
              sequence))
```

We define `map` using as our accumulator a `lambda` tht takes the result so far (`y`), and preppending to it the result of applying `p` to the next element in the list, `x`.  Note that this preserves the order of `sequence` because `accumulate` starts at the right side of the list and works its way left (we'll find out later in the chapter that another name for our `accumulate` method is `fold-right`, because it "folds" the list into the result from the right).

```scheme
(define (append seq1 seq2)
  (accumulate cons
              seq2
              seq1))
```

`append` is very simple, as we don't even need to write any new procedures: we simply use `cons` as our accumulator, and use `seq2` as our initial value.  Again, it's important to know that accumulate works its way from right to left of the input sequence, so we use `seq2` as the initial value and continuously preppend the rightmost element of the other list, which produces the effect of having all of elements of `seq1` followed by `seq2`.

```scheme
(define (length sequence)
  (accumulate (lambda (e c) (+ c 1))
              0
              sequence))
```

This one was the simplest.  Since `accumulate` will apply our accumulator once per element in the input sequence, counting the number of elements is the same as setting our initial value to 0, and incrementing it each time the accumulator is called.  We simply ignore the element that is also passed in.

###Ex 2.34

```scheme
(define (horner-eval x coefficient-sequence)
  (accumulate (lambda (this-coeff higher-terms) (+ (* x higher-terms)
                                                   this-coeff))
              0
              coefficient-sequence))
```

###Ex 2.35

```scheme
(define (count-leaves t)
  (accumulate +
              0
              (map (lambda (a) 1)
                   (fringe t))))
```

My solution makes use of the `fringe` procedure to get all of the leaves of the list, and using `map`, transform all of the elements of the list to 1's, then finally accumulate them using `+` to get the final count of the leaves.

###Ex 2.36

```scheme
(define (accumulate-n op init seqs)
  (if (null? (car seqs))
      '()
      (cons (accumulate op init (accumulate (lambda (seq next)
                                              (cons (car seq) next))
                                            '()
                                            seqs))
            (accumulate-n op init (accumulate (lambda (seq rest)
                                                (cons (cdr seq) rest))
                                              '()
                                              seqs)))))
```

This works by plucking off the first element of each of the sequences into another list using `accumulate`, and the element of the resulting sequence is computed using the `accumulate` with the user-provided `op`.  The rest of the sequence is computed using a recursive call to `accumulate-n` where the sequence of sequences given is created by using another call to `accumulate` to get the `cdr` of each of the sequences.


###Ex 2.37

```scheme
(define (dot-product v w)
  (accumulate +
              0
              (map * v w)))

(define (matrix-*-vector m v)
  (map (lambda (row)
         (dot-product row v))
       m))

(define (transpose mat)
  (accumulate-n cons
                '()
                mat))

(define (matrix-*-matrix m n)
  (let ((cols (transpose n)))
    (map (lambda (mrow)
           (accumulate (lambda (col prow)
                         (cons (dot-product col mrow) prow))
                       '()
                       cols))
         m)))
```

`matrix-*-vector` is pretty straightforward, the product of  matrix \\(m\\) and and vector \\(v\\) is simply a vector whose elements are the dot product of \\(v\\) an each row of the matrix \\(m\\).

`transpose` is also surprisingly simple, we simply want to turn the columns into the rows, so we use `accumulate-n` to take the first elements of each sequence (which are the rows), and create a list from them, which becomes a row in the resulting matrix.

`matrix-*-matrix` is a little tricky, but not too bad; the thing to notice is that we build the resulting matrix a row at a time, since we're mapping over `m`'s rows.  To build each row, we use `accumulate`, and compute each element of the row from left to right by using the given `cols` as the sequence, and computing the its dot product with the current row of `m`.

###Ex 2.38

```scheme
(fold-right / 1 (list 1 2 3))
```

`fold-right` starts with the right-most element, and works it way left.  For the above combination, the equivalent computation is `(/ (/ (/ 3 1) 2) 1)`, which has a value of `3/2`.

```scheme
(fold-left / 1 (list 1 2 3))
```

`fold-left` starts with the left-most element, and works it way left.  For the above combination, the equivalent computation is `(/ (/ (/ 1 1) 2) 3)`, which has a value of `1/6`.

```scheme
(fold-right list '() (list 1 2 3))
```

We're going from right to left, so we end up with `(list 1 (list 2 (list 3 '())))`.

```scheme
(fold-left list '() (list 1 2 3))
```

We start with the initial `'()`, and create a list it as the first element, and 1 as the second, `(list 1 '())`.  In general, as we move from left to right, the result from the previous iteration is the first element in the two element list constructed, so that it ends up being equivalent to `(list (list (list '() 1) 2) 3)`.

Note that it is not equivalent to the result from that of `fold-right`.

The reason that the `fold-left` results are not the same as the ones produced by `fold-right` and vice versa is because the result depends on the direction in which we traverse the input sequence.  Also note the difference in how the procedures pass the arguments pass arguments to the accumulator procedure: for `fold-left`, the result computed so far is the first argument, and the next element in the sequence is the second, and for `fold-right`, it's the reverse; the next element is the first argument, and the result computed so far is the second.  So not only are the elements in the sequence iterated over in different order, but the way the accumulator is ultimately "nested" is also different.  To guarantee that `fold-right` and `fold-left` produce the same results, `op` must be able to combine the items in the sequence indpendent of the order in which it is traversed, and regardless of the order which the result so far, and the next element are passed to it.

For example, `(fold-right + 0 (list 1 2 3))` and `(fold-left + 0 (list 1 2 3))` are equal because accumulation of numbers doesn't depend on the order in which they're added together.

###Ex 2.39

```scheme
(define (fold-right-reverse sequence)
  (fold-right (lambda (x y)
                (append y (list x)))
              '()
              sequence))

(define (fold-left-reverse sequence)
  (fold-left (lambda (x y)
               (cons y x))
             '()
             sequence))
```

The most natural one to use for this is `fold-left`, since by working our way from left to right, all we need to do is add the current element to the beginning of the list; it is also more efficient, since it does constant amount of work per element.  For the `fold-right` version, we begin at the right, so each element we encounter must be added to the *end* of the list, which requires work on the order of \\(O(n)\\) per `append`.

###Ex 2.40

```scheme
(define (unique-pairs n)
  (flatmap
   (lambda (j)
     (map (lambda (i) (list j i))
          (enumerate-interval (+ 1 j) n)))
   (enumerate-interval 1 (- n 1))))
```

To do this, we map over all possible values of `j`, that is \\(1 \leq j<n\\), and in an inner mapping, enumerate `i` from between \\(j<i \leq n\\).  This avoids any duplication (i.e. permutations of equivalent pairs such as \\((1,2)\\) and \\((2,1)\\)) because for any value of \\(j\\), we only look at values of \\(i\\) strictly larger than it.

###Ex 2.41

```scheme
(define (unique-triples n)
  (flatmap
   (lambda (i)
     (flatmap
      (lambda (j)
        (map (lambda (k) (list i j k))
             (enumerate-interval (+ j 1) n)))
      (enumerate-interval (+ i 1) (- n 1))))
   (enumerate-interval 1 (- n 2))))

(define (triple-sum n s)
  (define (sum-to-s? triple)
    (= (+ (car triple)
          (cadr triple)
          (car (cddr triple)))
       s))
  (define (make-triple-sum triple)
    (let ((i (car triple))
          (j (cadr triple))
          (k (car (cddr triple))))
      (list i j k (+ i j k))))
  (map make-triple-sum
       (filter sum-to-s? (unique-triples n))))
```

The approach is just like the one for Exercise 2.40.  One thing to note is the use of two `flatmaps`s.  This is because within the two innermost maps, we are generating lists of lists, so we want to use a `flatmap` to avoid such nesting.


###Ex 2.42

```scheme
(define (drop seq n)
  (if (= n 0)
      seq
      (drop (cdr seq) (- n 1))))

(define (take seq n)
  (if (= n 0)
      '()
      (cons (car seq) (take (cdr seq) (- n 1)))))

(define (queens board-size)
  (define empty-board (map (lambda (c)
                             (map (lambda (r) 0)
                                  (enumerate-interval 1 board-size)))
                           (enumerate-interval 1 board-size)))

  (define (safe? k positions)
    (define (occupied? row col)
      (if (or (< row 1) (> row board-size))
          #f
          (= (car (drop col (- row 1))) 1)))
    (define (find-queen col)
      (define (iter c col)
        (if (= (car col) 1)
            c
            (iter (+ c 1) (cdr col))))
      (iter 1 col))
    (define (safe?-iter anti-diag row diag cols)
      (cond ((null? cols) #t)
            ((or (occupied? anti-diag (car cols))
                 (occupied? row (car cols))
                 (occupied? diag (car cols))) #f)
            (else (safe?-iter (dec anti-diag) row (inc diag) (cdr cols)))))
    (if (= k 0)
        #t
        (let ((cols (drop positions (- board-size k))))
          (let ((queen-row (find-queen (car cols))))
            (safe?-iter (- queen-row 1) queen-row (+ queen-row 1) (cdr cols))))))

  (define (adjoin-position new-row k rest-of-queens)
    (define (place-queen col row)
      (append (take col (- row 1)) (list 1) (drop col row)))
    (append (take rest-of-queens (- board-size k))
            (list (place-queen (car empty-board) new-row))
            (drop rest-of-queens (- board-size (- k 1)))))

  (define (queen-cols k)
    (if (= k 0)
        (list empty-board)
        (filter
         (lambda (positions) (safe? k positions))
         (flatmap
          (lambda (rest-of-queens)
            (map (lambda (new-row)
                   (adjoin-position new-row k rest-of-queens))
                 (enumerate-interval 1 board-size)))
          (queen-cols (- k 1))))))
  (queen-cols board-size))
```

A surprising amount of code went into this, but I think it's actually pretty clear and easy to understand.

The board is represented as a sequence of sequences, where each sub sequence is a column of the board, with the right-most column appearing as first in the list, and the left-most column at the end. The rows for each column go from left (row 1) to right (row n).

The definition of `empty-board` is quite simple, and simply uses nested `map`s over sequences of length `board-size` to create a sequence of sequences.

My choice of how to represent the board made `safe?`, the trickiest one to implement (nicely), easier, since we can simply `drop` the columns we haven't looked at yet (everything beyond column `k`, which are towards the front of the list), and then simply iterate over the columns remaining, checking the diagonal, antidiagonal, and row during each iteration for a queen.

`adjoin-position` is quite simple; it works by `take`ing the last \\(board-size - k\\) columns--i.e. all the columns to the right of `k`--appending to this list a new column with a queen placed in `new-row`, and finally appending all the other columns before `k`.  The auxilary procedure `place-queen` works in an analogous manner.

I did implement `take` and `drop`, which both take a sequence and either returns the first `n` items in the given sequence, or returns the sequence with the first `n` items removed, respectively.

Note, the version of `append` used here is the one provided by the implementation (Neil van Dyke's in my case), which takes an abitrarily number of lists, instead of just 2, as in the case for the one given as an example in the book.

###Ex 2.43
*I initially thought it was \\(O(n^2)\\) extra work, but looking around at others' solutions to this problem (such as Eli Bendersky's), I realized that I had forgotten about the recursive call to `queens-cols`.

Here is the complete `queen-cols` procedure as Louis has it:

```scheme
(define (queen-cols k)
  (if (= k 0)
      (list empty-board)
      (filter
       (lambda (positions) (safe? k positions))
       (flatmap
        (lambda (new-row)
          (map (lambda (rest-of-queens)
                 (adjoin-position new-row k rest-of-queens))
               (queen-cols (- k 1))))
        (enumerate-interval 1 board-size)))))
		```

The reason his version runs very slowly is because we incur exponential amount of extra unnecessary work.  The outer `flatmap` now maps over the rows, while the inner `map` maps over solutions found so far for the preceding columns.  This is terrible for the running time because for each new row that we want to check, we incur a call to `queen-cols` (!!), all of which are a waste (other than one) because it will return the same set of solutions each time.  Contrast this to the book's version where `queen-cols` is only called \\(n\\) times, once for each column.

Each call to `queen-cols` for column \\(k\\), results in \\(n\\) more calls, this time for column \\(k - 1\\).  The call tree basically looks like a tree with branching factor \\(n\\), and height \\(n\\), so we end up with an exponential number of calls, instead of a linear number.

In conclusion, if the book's version solves the puzzle in time \\(T\\), then Louis version solves it in \\(T*n^n\\) time, \\(n^n\\) times slower.

###Ex 2.44

```scheme
(define (up-split painter n)
  (if (= n 0)
      painter
      (let ((smaller (up-split painter (- n 1))))
        (below painter (beside smaller smaller)))))
```

###Ex 2.45

In both split procedures, what they have in common is how the painter is "split", and how the given painter and its split counterpart are arranged.  In `right-split`, the two parts are arranged side-by-side, using `beside`, whereas in `up-split`, the two parts are arranged with the split painter above the original, using `above`.  For `right-split`, the painter is split by putting one copy of the painter below another using `below` and for `up-split`, the painter is split by putting two copies next to each other using `beside`.  This leads to the following generalization of both methods:

```scheme
(define (split arranger splitter)
  (lambda (painter n)
    (if (= n 0)
        painter
          (let ((smaller ((split arranger splitter) painter (- n 1))))
            (arranger painter (splitter smaller smaller))))))
```

###Ex 2.46

```scheme
(define (make-vect x y)
  (list x y))

(define xcor-vect car)
(define ycor-vect cadr)

(define (add-vect u v)
  (make-vect (+ (xcor-vect u)
		(xcor-vect v))
	     (+ (ycor-vect u)
		(ycor-vect v))))

(define (sub-vect u v)
  (make-vect (- (xcor-vect u)
		(xcor-vect v))
	     (- (ycor-vect u)
		(ycor-vect v))))

(define (scale-vect a u)
  (make-vect (* a (xcor-vect u))
	     (* a (ycor-vect u))))
```

Here, I chose to represent a vector as pair using `list`.

###Ex 2.47

Here is the first constructor and its matching selectors:

```scheme
(define (make-frame origin edge1 edge2)
  (list origin edge1 edge2))

(define (edge2-frame frame)
  (car (cddr frame)))

(define (edge1-frame frame)
  (car (cdr frame)))

(define (origin-frame frame)
  (car frame))
```

And here is the second constructor and its matching selectors:

```scheme
(define (make-frame origin edge1 edge2)
  (cons origin (cons edge1 edge2)))

(define (edge2-frame frame)
  (cddr frame))

(define (edge1-frame frame)
  (car (cdr frame)))

(define (origin-frame frame)
  (car frame))
```

Note that the selectors `origin-frame` and `edge2-frame` are exactly the same.  The only different selector is `edge2-frame`.  This is because both constructors produce very similar objects, but for the second one, the second edge is just the second element in the second `cons` cell, instead of being the first element in its own `cons` cell like the other two elements (meaning we don't need to `car`).

###Ex 2.48

```scheme
(define (make-segment a b)
  (list a b))

(define (start-segment s)
  (car s))

(define (end-segment s)
  (cadr s))
```


###Ex 2.49

Note: I wasn't actually able to run any of this code because my there was a problem with my implementation (Neil van Dyke's).  Hopefully these draw that they're supposed to!

A.

```scheme
(define outline (segments->painter
                 (list (make-segment (make-vect 0 0)
                                     (make-vect 0 1))
                       (make-segment (make-vect 0 0)
                                     (make-vect 1 0))
                       (make-segment (make-vect 0 1)
                                     (make-vect 1 1))
                       (make-segment (make-vect 0 1)
                                     (make-vect 1 1)))))
```

B.

```scheme
(define x (segments->painter
           (list (make-segment (make-vect 0 1)
                               (make-vect 1 0))
                 (make-segment (make-vect 0 0)
                               (make-vect 1 1)))))
```

C.

```scheme
(define x (segments->painter
           (list (make-segment (make-vect 0 1)
                               (make-vect 1 0))
                 (make-segment (make-vect 0 0)
                               (make-vect 1 1)))))
```


D.

For this part, (which I felt was a big time sink :\), I did my best to recreate the picture in MS Paint, then normalized the coordinates for each of the 17 segments.

```scheme
(define wave (segments->painter
              (list
               ;; 1
               (make-segment (make-vect 0 0.8)
                             (make-vect 0.1375 0.575))
               ;; 2
               (make-segment (make-vect 0 0.575)
                             (make-vect 0.121 0.4))
               ;; 3
               (make-segment (make-vect 0.1375 0.573)
                             (make-vect 0.231 0.635))
               ;; 4
               (make-segment (make-vect 0.121 0.398)
                             (make-vect 0.235 0.575))
               ;; 5
               (make-segment (make-vect 0.281 0.792)
                             (make-vect 0.388 1))
               ;; 6
               (make-segment (make-vect 0.281 0.792)
                             (make-vect 0.325 0.635))
               ;; 7
               (make-segment (make-vect 0.231 0.635)
                             (make-vect 0.325 0.635))
               ;; 8
               (make-segment (make-vect 0.235 0.575)
                             (make-vect 0.313 0.47))
               ;; 9
               (make-segment (make-vect 0.313 0.47)
                             (make-vect 0.177 0))
               ;; 10
               (make-segment (make-vect 0.39 0)
                             (make-vect 0.495 0.177))
               ;; 11
               (make-segment (make-vect 0.495 0.177)
                             (make-vect 0.552 0))
               ;; 12
               (make-segment (make-vect 0.554 1)
                             (make-vect 0.647 0.77))
               ;; 13
               (make-segment (make-vect 0.647 0.77)
                             (make-vect 0.579 0.633))
               ;; 14
               (make-segment (make-vect 0.579 0.633)
                             (make-vect 0.716 0.633))
               ;; 15
               (make-segment (make-vect 0.58 0.464)
                             (make-vect 0.668 0))
               ;; 16
               (make-segment (make-vect 0.716 0.633)
                             (make-vect 1 0.417))
               ;; 17
               (make-segment (make-vect 0.58 0.464)
                             (make-vect 1 0.212)))))
```


###Ex 2.50

```scheme
(define (flip-horiz painter)
  (transform-painter painter
                     (make-vect 1.0 0.0)
                     (make-vect 0.0 0.0)
                     (make-vect 1.0 1.0)))

(define (rotate180 painter)
  (transform-painter painter
                     (make-vect 1.0 1.0)
                     (make-vect 0.0 1.0)
                     (make-vect 1.0 0.0)))

(define (rotate270 painter)
  (transform-painter painter
                     (make-vect 0.0 1.0)
                     (make-vect 0.0 0.0)
                     (make-vect 1.0 1.0)))
```

###Ex 2.51

Here is the first way, analogous to `beside`:

```scheme
(define (below painter1 painter2)
  (let ((split-point (make-vect 0.0 0.5)))
    (let ((paint-bottom
          (transform-painter painter1
                             (make-vect 0.0 0.0)
                             (make-vect 1.0 0.0)
                             split-point))
          (paint-top
           (transform-painter painter2
                              split-point
                              (make-vect 1.0 0.5)
                              (make-vect 0.0 1.0))))
      (lambda (frame)
        (paint-bottom frame)
        (paint-top frame)))))
```

And the second, as a combination of rotations:

```scheme
(define (below painter1 painter2)
  (let ((bottom (rotate270 painter1))
        (top (rotate270 painter2)))
    (rotate90 (beside bottom top))))
```

The first way of defining `below` is analogous to the way beside was defined in the book.  We transform the two painters so that they paint to their respective halves (the upper and lower halves in this case), and then return a lambda accepts a frame and passes it along to the transformed painters.

For the second way, we use only `beside` coupled with rotations.  Obviously, we need to use `beside` to combine th two painters, but the image will be 90 degrees off, (the split is vertical), so we will nee to transform the result of `beside` using `rotate90`.  This implies that before we combine our two painters, we must also transform them so that after the 90 degree rotation at the end, it will be in the right orientation, so we rotate each input painter by 270 degrees using `rotate270`.


###Ex 2.52

For this exercise, I decided to skip part A because I couldn't verify it anyway, and takes too much time.

B.

```scheme
(define (corner-split-modified painter n)
  (if (= n 0)
      painter
      (let ((up (up-split painter (- n 1)))
            (right (right-split painter (- n 1))))
        (let ((smaller (corner-split painter (- n 1))))
          (below (beside painter right)
                 (beside up smaller))))))
```

C.

```scheme
(define (square-limit-modified painter n)
  (let ((combine4 (square-of-four flip-vert rotate180 identity flip-horiz)))
    (combine4 (corner-split painter n))))
```
