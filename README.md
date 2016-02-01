# Notes n Biz

## Chapter 1

This space intentionally left blank.

## Chapter 2

This space unintentionally left blank.

## Chapter 3: Functional Programming

### Day 1

#### Find

 * [The Clojure "cheat sheet"](http://clojure.org/api/cheatsheet)
 * [Documentation - `lazy-seq`](http://clojuredocs.org/clojure.core/lazy-seq)

#### Do

 * Rewrite `recursive-sum` using Clojure's `loop` and `recur`

`loop` -

Evaluates the exprs in a lexical context in which the symbols in
the binding-forms are bound to their respective init-exprs or parts
therein. Acts as a recur target.

`recur` -

Evaluates the exprs in order, then, in parallel, rebinds the bindings of
the recursion point to the values of the exprs

```
;; original form
(defn recursive-sum [numbers]
	(if (empty? numbers)
		0
		(+ (first numbers) (recursive-sum (rest numbers))))
```

using loop and recur

loop and recur example:
`(loop [sum 0 cnt 4] (if (= cnt 0) sum (recur (+ cnt sum) (dec cnt))))`

recursive-sum attempts

`(loop [numbers] (if (empty? numbers) 0 (recur (+ (first numbers) rest))))`

`(loop [numbers] (if (empty? numbers) 0 (+ (first (recur rest)))))`

`(defn loop-sum [numbers] (loop [numbers] (if (empty? numbers) 0 (+ recur rest))))`

=> IllegalArgumentException loop requires an even number of forms in binding vector in recursive-sum.core:1  clojure.core/loop (core.clj:4211)

_why does `loop` "require an even number of forms in binding vector" ?_

`(defn loop-sum [numbers] (loop [numbers numbers] (if (empty? numbers) 0 (+ (recur rest)))))`

=> CompilerException java.lang.UnsupportedOperationException: Can only recur from tail position

```
(defn loop-sum [numbers] (loop [numbers [] sum 0] (if (empty? numbers) sum (recur rest (+ first sum)))))

(loop-sum [1 2 3 4])
```

=> 0

`(loop [numbers [1 2 3 4] sum 0] (if (empty? numbers) sum (recur rest (+ first sum))))`

`(loop [sum 0 numbers [1 2 3 4]] (if (empty? numbers) sum (recur (+ first sum) rest)))`

=> first cannot be cast to java.lang.Number

 * Rewrite `reduce-sum` to use the `#()` reader macro instead of `(fn ...)`

This is a simple substitution.

From the docs:

> Anonymous function literal (#())
> `#(...) ⇒ (fn [args] (...))`
> where args are determined by the presence of argument literals
> taking the form `%`, `%n` or `%&`. `%` is a synonym for `%1`, `%n` designates
> the nth arg (1-based), and `%&` designates a rest arg. This is not
> a replacement for `fn` - idiomatic use would be for very short
> one-off mapping/filter `fn`s and the like.
> `#()` forms cannot be nested.

original:

```
(defn reduce-sum [numbers]
  (reduce (fn [acc x] (+ acc x)) 0 numbers))
```

modified:

`acc` becomes `%1` (argument 1) and `x` becomes `%2`.
The parameters vector is implied from the argument literals `%1` and `%2`, so you could think of it as `(fn [%1 %2] (+ %1 %2))`

```
(defn reduce-sum [numbers]
  (reduce #(+ %1 %2) 0 numbers))
```
