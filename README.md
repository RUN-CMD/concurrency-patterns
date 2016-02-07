# Notes n Biz

## Chapter 1

This space intentionally left blank.

## Chapter 2

This space unintentionally left blank.

## Chapter 3: Functional Programming

__reify__ - to make something more _real_

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

#### Do

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

### Day 2

#### Find

 * The video of Rich Hickey presenting reducers at QCon 2012 [video link](https://vimeo.com/45561411)
   * Blog post on reducers [web link](http://clojure.com/blog/2012/05/08/reducers-a-library-and-model-for-collection-processing.html)

 * The documentation for `pcalls` and `pvalues`
   * `pcalls` - Executes the no-arg fns in parallel, returning a lazy sequence of their values
     * [link](https://clojuredocs.org/clojure.core/pcalls)
     * `(pcalls function-1 function-2 ...)` => (result1 result2 ...)
   * `pvalues - Returns a lazy sequence of the values of the exprs, which are evaluated in parallel
     * [link](https://clojuredocs.org/clojure.core/pvalues)
     * `(pvalues (expensive-calc-1) (expensive-calc-2))` => (2330 122)
 * How do they differ from `pmap` ?
   * `pcalls` vs `pvalues`: "`pvalues` is a convienience macro around `pcalls`, though because macros are not first class it can't be composed or passed around in the same way a function can"
     * [from stackoverflow](http://stackoverflow.com/questions/21340186/clojure-pvalues-vs-pcalls)
   * this is all well and good, but _how do they differ from `pmap` ?_
 * Is it possible to implement `pcalls` and/or `pvalues` in terms of `pmap` ?

#### Do

 * Create `my-flatten` and `my-mapcat` along the lines of `my-map` (pg. 66)
 * Create `my-filter`

### Day 3

#### Notes

 * __referential transparency__ - _"anywhere an invocation of the function appears, we can replace it with its result without changing the behavior of the program" (pg. 72)_

 * _"Clojure is an impure functional language - it is possible to write [non-referentially transparent functions]" (pg. 73)_

 * Clojure enables concurrency via __dataflow programming__ which is realized via __futures__ and __promises__

 * __futures__ - _"takes a body of code and executes it in another thread
   * its return value is a `future` object
   * we can retrieve the value of a future by dereferencing it with either `deref` or `@`
   * dereferencing a future will block until the value is available (or _realized_)
   * we can use this to create [a dataflow graph]

 * __promises__ - similar to futures, except they do not execute immediately
   * a promise's value is set with `deliver`

Common clojure idiom:
 * create a promise
 * create a future that uses that promise
 * use `deliver` to set the value of the promise, which unblocks the future's thread and executes

A promise is _what_, a future is _how_.

#### Find

 * What is the difference between `future` and `future-call`?

> `(future-call f)`
> Takes a function of no args and yields a `future` object that will
> invoke the function in another thread, and will cache the result and
> return it on all subsequent calls to `deref`/`@`. If the computation has
> not yet finished, calls to `deref`/`@` will block, unless the variant
> of deref with timeout is used.

_"The future macro is just syntax sugar around future-call"_ - [Object Commando Blog](http://objectcommando.com/blog/2010/06/10/clojure-futures/)

 * How would you implement one in terms of the other?

 * How do you tell if a future has been realized without blocking?
 * How do you cancel a future?

From the blog post linked above, given a future `f`

```
(def f
  (future
    (println "Starting to sleep...")
    (Thread/sleep 600000)
    (println "Done sleeping.")))
```

This future sleeps for an hour (should give us enough time to poke at it).
The future object itself can be interrogated:

```
(future-done? f)
=> false

(future-cancelled? f)
=> false
You can then decide to cancel the future:

(future-cancel f)
=>true

(future-cancel f)
=>false
```

#### Do

 * Modify the transcript server so that a GET request to _/translation/:n_ doesn't block if the translation isn't yet available, but returns an HTTP 409 status code instead

 * Implement the transcript server in an imperative language. Is your solution as simple as the functional Clojure implementation? How confident are you that it contains no race conditions?

### Wrap-Up

 * Monads / Monoids

> A statically typed functional language like Haskell uses concepts like monads and monoids to allow its type system to accurately encode restrictions on where particular functions and values can be used and to keep track of side effects while remaining functional.

In Clojure, the burden is on the developer "to make sure that functions and values are used in appropriate contexts."

## Chapter 4 - The Clojure Way

### Day 1

#### Notes

__Persistent List__

A persistent data strucure is a construct that allows separation of identity and state. Here, _identity_ means "the idea we're representing", and _state_ represents "how the idea is represented at any given point in time".

Clojure represents a list as a persistent data structure by maintaining a representation of the list's state at every operation.

Given a list `listv1` as `(1 2 3)`, Clojure represents this in memory as (visually) `[listv1] -> [1] -> [2] -> [3]`

If we take that list `listv1`, prepend a value to it (say, 4), and define the result as `listv2`, we expect to have the result `(4 1 2 3)`. Clojure's representation now looks something like this:

```
[listv2] -> [4]
                \
      [listv1] -> [1] -> [2] -> [3]
```

At each definition, Clojure names and assigns a pointer to the head of the list representing the lists state at each point in time. If we traverse the result from `listv2`, we get `(4 1 2 3)`, which is the representation of our list identity with the state as it existed when we defined  `listv2`. However, we can still reference `listv1`, which still maintains a representation of the list at state `listv1`, which is still `(1 2 3)`.

This strikes me as similar to how git will represent different states ("commits") of a repository ("identity"), and may count as an implementation of the [memento design pattern](https://en.wikipedia.org/wiki/Memento_pattern).
