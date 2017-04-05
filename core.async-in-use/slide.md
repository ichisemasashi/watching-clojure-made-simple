Core.Async in Use - Timothy Baldridge
https://www.youtube.com/watch?v=096pIlA3GDo


----
(aka) Core.Async
from the Trenches

----
(aka) Core.Async
Where Do We Go from Here

----
Why this talk? Why Now?

- It's been about 4 years since we started building core.Async
- As a consultant I've seen things (good and bad)
- Lots of talks on the "how" of core.async. Not many on "when to use" core.async.
- Not a judgement call on tech. Libraries are collections of value propositions, we should recognize and evaluate the tradeoffs.

---
Agenda

- An analysis of styles of core.async programming seen in "the real world"
- Discussion of the pain points people see when using core.async
- "New" models and patterns that can help ease the pain

----
Batching operation

```
(defn batch [in max-size]
  (let [out (chan 1)]
    (go
      (loop [acc nil]
        (when-some [v (<! in)]
          (let [new-acc (concat acc v)]
            (if (>= (count new-acc) max-size)
              (when (>! out new-acc)
                (recur nil))
              (recur new-acc))))))))
```

----
Batching operation

Problem: IO...IO...everywhere
Batching operation

```
(defn batch [in max-size]
  (let [out (chan 1)]
    (go
      (loop [acc nil]
        (when-some [v (<! in)] ;; <-- IO
          (let [new-acc (concat acc v)]
            (if (>= (count new-acc) max-size)
              (when (>! out new-acc) ;; <-- IO
                (recur nil))
              (recur new-acc))))))))
```

----
Batching operation

Problem: Concat degrades over time (I've tested this)
Batching operation

```
(defn batch [in max-size]
  (let [out (chan 1)]
    (go
      (loop [acc nil]
        (when-some [v (<! in)]
          (let [new-acc (concat acc v)] ;; <-- BAD
            (if (>= (count new-acc) max-size)
              (when (>! out new-acc)
                (recur nil))
              (recur new-acc))))))))
```

---
Batching operation

Problem: count is O(n) when used on a lazy seq
(I came to Clojure to avoid thinking about this)

```
(defn batch [in max-size]
  (let [out (chan 1)]
    (go
      (loop [acc nil]
        (when-some [v (<! in)]
          (let [new-acc (concat acc v)]
            (if (>= (count new-acc) max-size) ;; O(n) op
              (when (>! out new-acc)
                (recur nil))
              (recur new-acc))))))))
```

----
Batching operation

Problem: How do I supply options for this output channel?

```
(defn batch [in max-size]
  (let [out (chan 1)] ;; <-- No way for user to change
    (go
      (loop [acc nil]
        (when-some [v (<! in)]
          (let [new-acc (concat acc v)]
            (if (>= (count new-acc) max-size)
              (when (>! out new-acc)
                (recur nil))
              (recur new-acc))))))))
```

----
Batching operation

Problem: No error handling!
```
(defn batch [in max-size]
  (let [out (chan 1)]
    (go
      (loop [acc nil]
        (when-some [v (<! in)] ;; what if we get an int?
          (let [new-acc (concat acc v)]
            (if (>= (count new-acc) max-size)
              (when (>! out new-acc)
                (recur nil))
              (recur new-acc))))))))
```

----
Batching operation

Who caught the last flaw?

```
(defn batch [in max-size]
  (let [out (chan 1)]
    (go
      (loop [acc nil]
        (when-some [v (<! in)]
          (let [new-acc (concat acc v)]
            (if (>= (count new-acc) max-size)
              (when (>! out new-acc)
                (recur nil))
              (recur new-acc))))))))
```

----
Batching operation

Problem: I don't return the right channel!

```
(defn batch [in max-size]
  (let [out (chan 1)] ;; <-- never returned
    (go
      (loop [acc nil]
        (when-some [v (<! in)]
          (let [new-acc (concat acc v)]
            (if (>= (count new-acc) max-size)
              (when (>! out new-acc)
                (recur nil))
              (recur new-acc))))))))
```

----
Thoughts:

1. What we have is imparative side-effecti(y) code
2. This doesn't seem "functional" at all
3. Why do I have to think at this low a level?
4. Why not work with something more declarative?

----
Why not Transducers?

```
(defn batch [in max-size]
  (let [out (chan 1)
        xf (comp cat (partition-all max-size))]
    (pipeline 1 out xf in)
    out))
```
