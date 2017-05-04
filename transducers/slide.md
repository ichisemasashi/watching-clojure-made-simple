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
