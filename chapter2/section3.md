###Ex 2.53

```scheme
(list 'a 'b 'c)
=> ('a 'b 'c)

(cdr '((x1 x2) (y1 y2)))
=> (('y1 'y2))

(cadr '((x1 x2) (y1 y2)))
=> ('y1 'y2)

(pair? (car 'a short list))
=> #f

(memq 'red '((red shoes) (blue socks)))
=> #f

(memq 'red '(red shoes blue socks))
=> #t

```

###Ex 2.54

```scheme
(define (equal? a b)
  (cond ((and (null? a) (null? b)) true)
        ((and (null? a) (not (null? b))) false)
        ((and (not (null? a)) (null? b)) false)
        (else (if (eq? (car a) (car b))
                  (equal? (cdr a) (cdr b))
                  false))))
```

We simply walk down the lists, and compare the elements one by one.

###Ex 2.55

As stated in footnote 34, the `'` single quote is merely syntactic sugar, and is shorthand for the `quote` procedure.  Therefore, `(car ''abracadabra)` is actually `(quote (quote abracadabra))`.  The inner "quote" is treated as a symbol, rather than being evaluated by the interpreter.  This is further proved by the fact that `(cadr ''abracadabra)` returns `'abracadabra`.

###Ex 2.56

Here is how exponentations are represented:

```scheme
(define (exponentiation? x)
  (and (pair? x) (eq? (car x) '**)))

(define (base e) (cadr e))
(define (exponent e) (caddr e))

(define (make-exponentiation base exponent)
  (cond ((= exponent 0) 1)
        ((= exponent 1) base)
        (else (list '** base exponent))))
```

The implementation is pretty straightforward, I think, no surprises.  Here is the modified `deriv` program that accounts for exponentations:

```scheme
(define (deriv exp var)
  (cond ((number? exp) 0)
        ((variable? exp)
         (if (same-variable? exp var) 1 0))
        ((sum? exp)
         (make-sum (deriv (addend exp) var)
                   (deriv (augend exp) var)))
        ((product? exp)
         (make-sum
          (make-product (multiplier exp)
                        (deriv (multiplicand exp) var))
          (make-product (deriv (multiplier exp) var)
                        (multiplicand exp))))
        ((exponentiation? exp)
         (make-product (make-product (exponent exp)
                                     (make-exponentiation (base exp)
                                                          (- (exponent exp) 1)))
                       (deriv (base exp) var)))

        (else
         (error "unknown expression type -- DERIV" exp))))
```

This simply implements the differentiation rule for exponents shown in the book: the derivative of an exponentaiton is the product of the exponent, $$u$$ raised to one less than its exponent and the derivative of $$u$$ with respect to the given variable.

###Ex 2.57
Here are the relevant modifications:

```scheme
(define (augend s)
  (let ((rest (cddr s)))
    (if (not (= (length rest) 1))
        (cons '+ rest)
        (car rest))))p

(define (multiplicand p)
  (let ((rest (cddr p)))
    (if (not (= (length rest) 1))
        (cons '* rest)
        (car rest))))
```

Both work in the same way.  It treats everything after the addend/multiplier as the the augend/multiplicand.  If the length of the remaining list is 1 then that means that the expression only has 1 term left, and simply returns it.  If it isn't it creates a new sum or product expression by prepending the appropriate symbol to the list; i.e. `'+` or `'*`.

###Ex 2.58

A. Being able to assume that `+` and `*` both take two arguments and all epxressions are fully parenthesized makes this exercise relatively straightforward.  Here are the changes made to the original code:

First, we change `make-product` to put the `'*` symbol between the two terms:

```scheme
(define (make-product m1 m2)
  (cond ((or (=number? m1 0) (=number? m2 0)) 0)
        ((=number? m1 1) m2)
        ((=number? m2 1) m1)
        ((and (number? m1) (number? m2)) (* m1 m2))
        (else (list m1 '* m2))))
```

Then the selectors `multiplier` and `multiplicand` are modified to return the correct pieces of the structure:

```scheme
(define (multiplier p) (car p))
(define (multiplicand p) (caddr p))
```

We also change `product?` to check the second element rather than the first for `'*`:

```scheme
(define (product? x)
  (and (pair? x) (eq? (cadr x) '*)))
```

We do the exact same thing for sums:

for `make-sum`:

```scheme
(define (make-sum a1 a2)
  (cond ((=number? a1 0) a2)
        ((=number? a2 0) a1)
        ((and (number? a1) (number? a2)) (+ a1 a2))
        (else (list a1 '+ a2))))
```

for `addend` and `augend`:

```scheme
(define (addend s) (car s))
(define (augend s) (caddr s))
```

for `sum?`:

```scheme
(define (sum? x)
  (and (pair? x) (eq? (cadr x) '+)))
```

B.  *Check stackedit on V3 (Linux), should be there I think*.

###Ex 2.59

```scheme
(define (union-set set1 set2)
  (cond ((null? set1) set2)
        ((not (element-of-set? (car set1) set2))
         (cons (car set1)
               (union-set (cdr set1) set2)))
        (else (union-set (cdr set1) set2))))
```

The approach is very similar to `intersection-set`.  If `set1` is empty, we just return `set2`.  To create the union of two nonempty sets, we take an element from `set1`, and if it's not already in `set2`, we add to the recursively computed union of the tail of `set1` and `set2`.  Just like `intersection-set`, this procedure also runs in $$O(n^2)$$ with the choice to represent sets as lists because for every element of `set1`, we use `element-of-set?` to check if it's in `set2`.


###Ex 2.60

`element-of-set?` remains the same, and still runs in $$O(n)$$ time, since we still need to search through the list to see if the given element is in it.

`intersection-set` also remains the same.  We can't do better than $$O(n^2)$$ because for every element of `set1`, we must check that it's in `set2` in order to decide whether it belongs in the result or not, which means an invocation to `element-of-set?`.

`adjoin-set` can be simplified to an unconditional `cons`:

```scheme
(define (adjoin-set x set)
  (cons x set))
```

Since we're allowing duplicates, we don't have to check first that `x` isn't already a member of the set.  This is now an $$O(1)$$ operation, down from $$O(n)$$.

Likewise, `union-set` can also be simplified:

```scheme
(define (union-set set1 set2)
  (append set1 set2))
```

Duplicates are fine, so can simply append the `set2` to `set1`.  This is now an $$O(n)$$ operation, down from $$O(n^2)$$.

Obviously, choosing to allow duplicates in our representation of sets is advantagous if the operations we perform are dominated by unions and adjoins.  Certainly allowing duplicates can't really hurt (unless our sets contain massive amounts of duplicates), since the asymptotic complexity of the operations either go down or remain the same.

###Ex 2.61

```scheme
(define (adjoin-set x set)
  (cond ((null? set) (cons x '()))
        ((= x (car set)) set)
        ((< x (car set)) (cons x set))
        (else (cons (car set) (adjoin-set x (cdr set))))))
```

Here we build the set recursively.  If the given set is `null`, then we just return a list with `x` as the singular element.  If the next element in the set is equal to `x`, then it's already in the set so we return it.  If the next element is greater, then we know it's not in the set so then we add it.  Finally, if the next element is less, then the result is the cons of this element of the set and the result of adjoining `x` to the `cdr` of the set.  Just like for the modified version of `element-of-set?`, on average we will only need to search through the first half of the list.

###Ex 2.62

```scheme
(define (union-set set1 set2)
  (cond ((null? set1) set2)
        ((null? set2) set1)
        (else
         (let ((x1 (car set1)) (x2 (car set2)))
           (if (= x1 x2)
               (cons x1 (union-set (cdr set1) (cdr set2)))
               (cons x1 (cons x2 (union-set (cdr set1) (cdr set2)))))))))
```

If one of the sets is null, then we just return the other.  Otherwise, we compare the first element of each set.  If they are the same, the we add just one copy of the element to the resulting set.  If `x1` is less than `x2`, then that means `x1` does not appear in `set2` so we add it to the union of the tail fo `set1` and `set2`.  Similarly, if `x2` is less thatn `x1`, then `x2` does not appear in `set1`, so we add it to the union of `set1` and the tail of `set2`.  Just like for `intersection-set`, either one or both of the lists shrink by on element in each iteration, so this procedure is $$O(n)$$.

###Ex 2.63


A. Both procedures produce the same list.  By looking at the code, we see that their recursive plans are identical: for `tree->list-1`, we append to the transformation of the left subtree the list created by preppending the entry of the current tree to the transformation of the right subtree.  For `tree->list-2`, we work our way from right to left of the tree, adding the current entry to the start of the list as we go (it's basically a DFS from right to left).


B. No, `tree->list-1` is slower because it uses `append` which runs in $$O(n)$$ time.  By drawing out a call tree for a full binary tree with $$n$$ elements, we see that any level $$j$$ (where $$0 \leq j \leq \log{n}$$), we make $$2^j$$ calls to append, where each list we append has length $$\frac{1}{2^{j+1}}n$$:

\begin{equation}
\sum_{j=0}^{\log{n}} 2^j \cdot \frac{1}{2^{j+1}}n = \frac{1}{2}n\log{n}
\end{equation}

This isn't exact because each subtree doesn't have exactly half of the elements; instead the parent node has one element, and the two subtrees each have half of the remaining elements; so more like
$$\lfloor\frac{n}{2}\rfloor$$.  Regardless, the running time is still asymptotically $$O(n\log{n})$$.  Anyone who's taken an algorithms course will recognize this as being very similar to the analysis of mergesort.

We can also use the Master Theorem to show the running time $$O(n\log{n})$$.  By looking again at the call tree, we can see that we always recurse on problems that are half the size, and do work that is $$O(\frac{1}{2}n)$$ at each node, so we get the following recurrence:

\begin{equation}
T(n) = 2T(\frac{n}{2}) + \frac{1}{2}n
\end{equation}

which the Master Theorem will tell us is $$O(n\log{n})$$.

`tree->list-1` on the other hand, performs only constant amount of work at each node, and visits each node only once, so for the entire operation, it's $$O(n)$$.

###Ex. 2.64

A. `partial-tree` is (unsurprisingly) a tree-recursive process.  It begins by recursively building the left subtree from the first $$\frac{n-1}{2}$$ elements.  Note that it's $$n-1$$ and not merely $$n$$ because we want the middle element to be entry for the current node.  The process then builds the right subtree in the same manner.  At this point, the first recursive call to partial-tree has returned the elements that were not put into left subtree.  The length of this list is at most 1 more than the number of nodes in the left subtree, and we take the first element in the list as the entry for the current node.  Another recursive call to `partial-tree` is made, giving it the remaining elements to form the right subtree.  Finally, having created both the left and right sub-trees, we return the `cons` cell containing as its first element the tree we just computed, and the list of elements that were left out as the second element.

Here is the tree that it creates:

<pre>
    5
   / \
  /   \
 /     \
1       9
 \     / \
  3   7   11
</pre>

Notice that it's balanced, and it's easy to see why: every time it recurses, it builds the sub-trees from half of the elements in the list.

B. As mentioned in part A, this is a tree recursive process with a branching factor of 2 (we branch for the left and right subtree).  Each recursive call is on half the list, we always do a constant amount of work during each call.  Putting it all together:

\begin{equation}
2^{\log{n}} \cdot O(1) = O(n)
\end{equation}

So we end up doing $$O(n)$$ in total to build the tree.

###Ex 2.65
The problem statement gives a nice hint as how to do it, which is to first transform the two trees to lists.  Because they are binary search trees, transforming them to lists in the manner that `tree->list-2` does will preserve the ordering.  We know that this process takes $$O(n)$$ time.  We also already have procedures to find the union of two ordered sets in $$O(n)$$ time, and finally, we have a procedure that transforms an ordered list to a binary search tree in $$O(n)$$ time.  All together, the entire procedure takes $$O(n)$$, though probably with bigger constants than our previous union implementations.

We do the same exact thing for `intersection-set`.

```scheme
(define (union-set set1 set2)
  (let ((s1-list (tree->list-2 set1))
        (s2-list (tree->list-2 set2)))
    (list->tree (union-set-list s1-list s2-list))))

(define (intersection-set set1 set2)
  (let ((s1-list (tree->list-2 set1))
        (s2-list (tree->list-2 set2)))
    (list->tree (intersection-set-list s1-list s2-list))))
```


`union-set-list` and `intersection-set-list` are the union and intersection methods we created for ordered sets.

###Ex 2.66

```scheme
(define (lookup given-key set-of-records)
  (if (null? set-of-records)
      false
      (let ((record (entry set-of-records)))
        (cond ((= given-key (key record)) record)
              ((< given-key (key record)) (lookup given-key (left-branch set-of-records)))
              ((> given-key (key record)) (lookup given-key (right-branch set-of-records)))))))
```

This employs the same recursive plan that we had for testing for set membership in the tree set.

### Ex 2.67

The decoded message is `'(a d a b b c a)`.

### Ex 2.68

```scheme
(define (encode-symbol s tree)
  (define (element-of-set? x set)
    (cond ((null? set) #f)
          ((equal? x (car set)) true)
          (else (element-of-set? x (cdr set)))))
  (if (leaf? tree)
      '()
      (let ((l (left-branch-huffman tree))
            (r (right-branch-huffman tree)))
        (cond ((element-of-set? s (symbols l)) (cons 0 (encode-symbol s l)))
              ((element-of-set? s (symbols r)) (cons 1 (encode-symbol s r)))
              (else (error "bad symbol -- ENCODE-SYMBOL" s tree))))))
```

Encoding a symbol is simple: we essentially do a search for the symbol in the tree, appending a `0` whenever we have to move left, and a `1` whenever we move right.  `element-of-set?` is the linear list search we started of with when we began talking about sets.

Indeed, encoding the result we got from the previous exercise (`'(a d a b b c a)`), we get back the same bit string.

###Ex 2.69

```scheme
(define (successive-merge leaves)
  (cond ((null? leaves) '())
        ((null? (cdr leaves)) (car leaves))
        (else
         (let ((n1 (car leaves))
               (n2 (cadr leaves)))
           (successive-merge (adjoin-set-huffman (make-code-tree n1 n2)
                                                 (cddr leaves)))))))
```

Here we have an iterative process.  If `leaves` is empty, we return `null`, and if it only has one element, we return that element.  Otherwise, we take the first two elements from `leaves`, "merge" them to create a new node using `make-code-tree`, then add it to the set of remaining leaves.  Note that we `adjoin-set` here and not just a simple `cons`.  This allows us to maintain the invariant that `leaves` is always an ordered set, so it's always safe to assume that the two nodes that should be merged are the first two.

We can test this again with the sample message:

```scheme
(encode '(a d a b b c a) (generate-huffman-tree '((C 1) (D 1) (B 2) (A 4))))
=> '(0 1 1 0 0 1 0 1 0 1 1 1 0)
```

Note: the ordering of symbols `'C` and `'D` in the list matters, otherwise we won't get the same encoding as `'D` and `'C` will both be leaves on the same level of the tree.

###Ex 2.70

We need just 84 bits to encode the song.  With a fixed-length encoding, each of the symbols would be would be at least 3 bits long.  The song contains 36 symbols, so a fixed-length encoding of the song with a 3-bit alphabet would require 108 bits.

###Ex 2.71

First, we have to realize that every time a symbol undergoes a merge, its encoding length increases by one since it becomes the child of another node. Also, any symbol (or the tree it's a part of) is merged only if it's one of the two least frequent symbols in the set. For the most frequent symbol, this doesn't happen until the very end because the rest of the set has a combined frequency of $$2^n-1$$.  Therefore, the most frequent symbol will have an encoding length of 1 bit.  On the other hand, the two least frequent symbols will undergo $$n-1$$ merges, because they will always be part of the tree with the lowest frequency. In particular, for any subtree with combined frequency $$2^n - 1$$, there is a symbol with frequency $$2^n$$ that it will be merged with.  Therefore, the least frequent symbol will have an encoding length of $$n-1$$ bits.

###Ex 2.72

For the type of tree generated for Exercise 2.71, with $$n$$ symbols, the height is $$n-1$$.  And at each node, we check which part of the tree the symbol is in (either left or right), which, given our choice of to use an unordered list as our set representation, this check takes $$O(n)$$ time.  For the most frequent symbol, the search finishes at the top of the tree after a single search, so it takes $$O(n)$$ time.  For the most frequent symbol, it takes until we reach the bottom of the tree, after we've gone down $$O(n)$$ internal nodes, doing $$O(n)$$ search at each node, so it's $$O(n^2)$$.
