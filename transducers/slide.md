# Transducers

----

What are They?

- extract the essence of map, filter et al
- away from the functions that transform sequences/collections
- so they can be used elsewhere
- recasting them as process transformations

---

What Kinds of Processes?

- ones that can be defined in terms of a succession of steps
- where each step ingests an input
- building a collection is just one instance
- seeded left reduce is the generalization

----

Why 'transducer'?

- reduce
  'lead back'
- ingest
  'carry into'
- transduce
  'lead across'
- on the way back/in, will carry inputs across a series of transformations

----

Transducers in the Real World

- 'put the baggage on the plane'
- 'as you do that'
  - break apart pallets
  - remove bags that smell like food
  - label heavy bags

----

Conveyances, sources, sinks are irrelevant

- And unspecified
- Does baggage come/go on trolleys or conveyor belts?

  Rules don't care

----

Transformation in the Programming World

- Collection function composition:

(comp
  (partial map label-heavy)
  (partial filter non-food?)
  (partial mapcat unbundle-pallet))

----

Conveyances are everywhere

- map, filter, mapcat are functions of sequence -> sequence
- 'rules' only work on sequences
- creates sequences between steps

----

No reuse

- Every new collection/process defines its own versions of map, filter, mapcat et al
  - MyCollection -> MyCollection
  - Stream -> Stream
  - Channel -> Channel
  - Observable -> Observable ...
- Composed algorithms are needlessly specific and inefficient

----

Creating Transducers

(def process-bags
  (comp
    (mappcatting unbundle-pallet)
    (filtering non-food?)
    (mapping label-heavy)))
- mapcatting, filtering, mapping return transducers
- process-bags is a transducer
- transducers modify a process by transforming its reducing function

----

Using Transducers

;; build a concrete collection
(into airplane process-bags pallets)
;; build a lazy sequence
(sequence process-bags pallets)
;; like reduce, but takes transducer
(transduce
  (comp process-bags (mapping weigh-bag))
  + 0 pallets)
;; a CSP channel that processes bags
(chan 1 process-bags)
;; it's an open system
(observable process-bags pallet-source)

----

Transducible Processes

- into, sequence, transduce, chan etc accept transducers
- use the transducer to transform their (internal, encapsulated) reducing function
- do their job with the transformed reducing function

----
Deriving Transducers

- Lectures on Constructive Functional Programming
  by R.S. Bird
- A tutorial on the universality and expressiveness of fold
  by GRAHAM HUTTON

- http://www.cs.osx.ac.uk/files/3390/PRG69.pdf
- http://www.cs.nott.ac.uk/~gmh/fold.pdf

---

Many list fns can be defined in terms of foldr

- encapsulates the recursion
- easier to reason about and transform

(defn mapr [f coll]
  (foldr (fn [x r] (cons (f x) r))
         () coll))
(defn filterr [pred coll]
  (foldr (fn [x r] (if (pred x) (cons x r) r))
         () coll))

----

Similarly, via reduce (foldl)

- returning eager, appendy vectors

(defn mapl [f coll]
  (reduce (fn [r x] (conj r (f x)))
           [] coll))
(defn filterl [pred coll]
  (reduce (fn [r x] (if (pred x) (conj r x) r))
          [] coll))
(defn mapcatl [f coll]
  (reduce (fn [r x] (reduce conj r (f x)))
          [] coll))
* We want to get rid of 'conj'

---

Transducers

- modify a process by transforming its reducing function

(defn mapping [f]
  (fn [step]
    (fn [r x] (step r (f x)))))
(defn filtering [pred]
  (fn [step]
    (fn [r x] (if (pred x) (step r x) r))))
(def cat
  (fn [step]
    (fn [r x] (reduce step r x))))
(defn mapcatting [f]
  (comp (map f ) cat))

---

reduce-based map et al redux

(defn mapl [f coll]
  (reduce ((mapping f) conj)
          [] coll))
(defn filterl [pred coll]
  (reduce ((filtering pred) conj)
          [] coll))
(defn mapcatl [f coll]
  (reduce ((mapcatting f) conj)
          [] coll))
* conj is now an argument

---

Transducers are Fully Decoupled

- Know nothing of the process they modify
  - reducing function fully encapsulates
- May call step 0, 1 or more times
- Must pass previous result as next r otherwise must know nothing of r
- can transform input arg

---

Backwards comp?

(comp
  (mapcatting unbundle-pallet)
  (filtering non-food?)
  (mapping label-heavy))

  - No, composing the transformers yields input transformations that run left->right

  ![figure-001](figure-001.png)

---

Transducers are Fast

- Just a stack of function calls
  - short, inlinable
- No laziness overhead
- No interim collections
- No extra boxes

---

Transducer Types, Thus Far

![figure-002](figure-002.png)

----

Early Termination

- Reduction normally processes all input
- Sometimes a process has just 'had enough' input, or gotten external trigger to terminate
- A transducer might decide the same

(comp
  (mapcatting unbundle-pallet)
  (taking-while non-tricking?)
  (filtering non-food?)
  (mapping label-heavy))

----

Reduced

- Clojure's reduce supports early termination via (reduced result)
- A wrapper with a corresponding test: reduced?
- And unwrap/dereference

(reduced? (reduced x)) -> true
(deref (reduced x)) -> x

----

Transducers Support reduced

- step functions can return (reduced value)
- If a transducer gets a reduced value from a nested step call, it must never call that step function again with input

(defn taking-while [pred]
  (fn [step]
    (fn [r x]
      (if (pred x)
        (step r x)
        (reduced r)))))

----
Processes Must Support Reduced

- If the step function returns a reduced value, the process must not supply any more input to the step function
- the dereferenced value is the final accumulation value
- the final accumulation value is still subject to completion (more later)

----
