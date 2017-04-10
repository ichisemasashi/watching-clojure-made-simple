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

----
Or:

```
(defn batch [max-size]
  (chan 1 (comp cat (partition-all max-size))))
```

----
Or:

```
(defn batch-xf [max-size]
  (comp cat (partition-all max-size)))
```

----
Example 2

```
(defn get-value [key callback] ...)

(defn value-for [key]
  (let [c (chan)]
    (get-value key #(async/put! c %))
    c))

(defn sum-values [keys]
  (go (loop [acc 0, [h & t] keys]
        (if h
          (recur (+ acc (<! (value-for h))) t)
          acc))))

(<!! (sum-values "a" "b" "c"))
```

----
Thoughts

- In most VMs, async code pollutes the return type (true in C#, Python, JS, Clojure, etc.)
- Each of these places has problems with error handling, exceptions, etc.
- Exceptions as values would require us to check the return type (or write channel ops that throw and then wrap everything in a try)
- IO...IO everywhere

----
We need better solutions and abstractions

----
Patterns and ideas for reducing the complexity of async code

---
One weird trick that will drasticly reduce the complexity of your code ...

----
Don't go async! (Unless you need it)

----
Use transducer enabled async callbacks:
```
; Simulate an async page request
(defn get-page [idx k]
  (println "Fetching Page" idx)
  ;; Simulate network callback
  (Thread/sleep 100)
  (future
    ;; Dispatch on a different Thread
    (k (vec (range (* idx 5) (* (inc idx) 5))))))
```

----
Wrap the async function:

```
(defn pages [xf rf k]
  (let [f (xf rf)
        start (fn process-page [idx acc page]
                ;; Call the function with the acc and page
                (let [result (f acc page)]
                  (if (reduced? result)
                    (k @result)
                    ;; Continue with the next page
                    (get-page (inc idx)
                              (partial process-page
                                       (inc idx)
                                       result)))))]
      (get-page 0 (partial start 0 (f)))))
```

----
Usage:

```
(def xform (comp cat
                 (filter #(> % 10))
                 (take 10)))
(pages xform
       conj
       (fn [result]
         (println "Finished with " result)))
```

-----
Thoughts on this pattern:

1. Works with any language (no core.async or other transforms required)
1. Removes the "implied buffer of 1" per process, allowing for more control over number of pages requested
1. Could be expanded to pass errors as values
1. Could be expanded to pre-fetch pages
1. Easily extended to core.async channels

----
Om.Next Model

```
(defi AutoCompleter
  static om/IQueryParams
  (params [_]
    {:query ""})

  static om/IQuery
  (query [_]
    '[(:search/results {:query ?query})])

  Object
  (render [this]
    (let [{:keys [search/results]} (om/props this)]
      (dom/div nil
        (dom/h2 nil "Autocompleter")
        (cond->
          [(search-field this
            (:query (om/get-params this)))]
          (not (empty? results))
            (conj (result-list results)))))))
```

----
Thoughts on this pattern

- No async code to be seen (handled by the framework)
- Required data is explicit (allows for bulk loading)
- Code is very testable
- Combine the params definitions with spec for even more power


---
Reified Stack

----
Example: Pedestal Interceptors

```
(defrecord Interceptor [enter exit on-error])

(def pipeline [parse-params encode-body handler])

(defn parse-params-enter
  [{:keys [params-string] :as ctx}]
  (assoc ctx :params (split-params params-string)))

(defn encode-body-exit
  [{:keys [body] :as ctx}]
  (assoc ctx :body-string (json/encode body)))

(defn handler-enter [{:keys [params db-conn] :as ctx}]
  (go
    (assoc ctx :body (<! (get-record (:id params))))))
```

----
Reified Stack

- We can modify the stack while it's being executed (if we store the stack inside the context)
- Non-async interceptors are kept Non-async
- We could execute parts of the stack multiple times (for websockets/SSE this might make sense)
- Applies to message pipeline (could integrate with (n)ack features of message queues)

----
Dataflow (and FRP)

- Nodes connected via communication channels
- Each node takes and emits from one or more inputs/outputus
- Each node consists of a function that computes outputs based on inputs

----
Dataflow (and FRP)

```
(defn emit [state port value]
  (update-in state [:outputs port] conj values))

(defn filter-fn [pred]
  (fn [state msg]
    (if (pred msg)
        (emit state :out msg)
        state)))

(def filter-node {:inputs {:in (filter-fn pos?)}
                  :outputs {:out {}}})
```

----
Dataflow (and FRP)

```
(defn split-fn [key-fn state msg]
  (emit state (key-fn msg) msg))

(defn distinct-fn [state msg]
  (if (contains? (:seen state) msg)
      state
      (-> (emit state :out msg)
          (update-in state :seen (fnil conj #{}) msg))))
```

----
Dataflow and (FRP)

```
;; Leverage Transducers
(defn transducer-fn [xducer]
  (let [rf (fn [state msg]
             (emit state :out msg))]
             (xducer rf)))

(def filter-node {:inputs {:in (transducer-fn filter pos?)}}
                  :outputs {:out []})
```

-----
Dataflow (and FRP)

```
(-> (graph)
    (add-node :split (split-node #(if (odd? %) :odd :even)))
    (add-node :square (transducer-node (map #(* % %))))
    (connect :split :even :square :in)
    (connect :split :odd :graph-exit)
    (connect :square :out :graph-exit)
    (ingest-all (range 1 4)))
```

----
Dataflow (and FRP)

Thoughts:
- Nodes are easily testable (functionally pure)
- Async code is removed from the user interface
- Connections are explicit
- Could use core.async channels as connections

-----
Rant on Actors

---
Discussion of Actors

Looks like the Dataflow example except:
- Connections are implicit
- Many (most) actor systems allow for sending messages inside user code
- Opaque state hidden in loop local
- One input mailbox requires senders to tag messages with extra info.

-----
Pattern in the Patterns?

- Keep userspace code pure
- Move the complexity of async out of the user space
- Make dependencies explict
- Leverage this for easier testing
- Use Core.Async to enable cleaner abstractions, not as an end in itself.
