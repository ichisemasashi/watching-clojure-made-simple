Clojure

A Dynamic Programming Language for the JVM

Concurrency Support

Rich Hickey

---
Agenda

- Introduction
- Feature Tour
- Shared state, multithreading and locks
- Refs, Transactions, and Agents
- Walkthrough - Multithreaded ant colony simulation
- Q & A

----
Introduction

- Who are you?
  - Know/use Lisp?
  - Java/C#/Scala?
  - ML/Haskell?
  - Python, Ruby, Groovy?
  - Clojure?
- Any multithreaded programming?

----
Clojure Fundamentals

- Functional
  - Immutable, persistent data structures
  - No mutable local variables
- Lisp
  - Not CL or Scheme
- Hosted on, and embracing, the JVM
- Supporting Concurrency
- Open Source

----
Clojure Features

- Dynamic development
  - REPL, reader, on-the-fly compilation to JVM bytecode
- Primitives - numbers, including arbitary-precision integers & ratios, characters, strings, symbols, keywords, regexes
- Aggregates - lists, maps, sets, vectors
  - read-able, persistent, immutable, extensible
- Abstract sequences + library
- Metadata
- First-class functions (fn), closures
- Recursive functional looping
- Destructuring binding in let/fn/loop
- List comprehensions (for)
- Macros
- Multimethods
- Concurrency support
- Java interop
  - Call methods, access fields, arrays
  - Proxy interfaces/classes
  - Sequence functions extended to Java strings, arrays, Collections
  - Clojure data structures implement Collection/Callable/Iterable/Comparable etc where appropriate
- Namespaces, zippers, XML and more!

----
State - You're doing it wrong

- Mutable objects are the new spaghetti code
  - Hard to understand, test, reason about
  - Concurrency disaster
  - Terrible default architecture (Java/C#/Python/Ruby/Groovy/CLOS...)
- Doing the right thing is very difficult
  - Languages matter!

----
Concurrency

- Interleaved/simultaneous execution
- Must avoid seeing/yielding inconsistent data
- The more components there are to the data, the more difficult to keep consistent
- The more steps in a logical change, the more difficult to keep consistent
- Opportunities for automatic parallelism
  - Emphasis here on coordination

----
Explicit Locks

- lock/synchronized (coll){...}
- Only one thread can have the lock, others  block
- Requires coordination
  - All code that performs non-atomic access to coll must put that in a lock block
  - Synchronized handles single-method jobs only

----
Single Lock Problems

- Can't enforce coordination via language/code
  - This is not a small problem
- Even when correct, can cause throughput bottleneck on multi-CPU machines
  - Your app is running on a multi-CPU machine
  - Readers block readers

----
Enhancing Read Parallelism

- Multi-reader/single-writer locks
  - Readers don't block each other
  - One writer at a time
  - Writers wait for reader(s)

----
Copy On Write Collections

- Reads get a snapshot
- Lock-free reading
- Atomic writes
- Internally, copy the representation and swap it
  - Writes can be expensive (copying)
- Multi-step writes still require locks

----
Persistent Data Structures

- Immutable, + old version of the collection is still available after 'changes'
- Collection maintains its performance guarantees for most operations
  - Therefore new versions are not full copies
- All Clojure data structures persistent
  - Hash map and vector both based upon array mapped hash tries (Bagwell)
  - Sorted map is red-black tree

---
![Bit-partitioned hash tries](figure001.png)

----
![Path Copying](figure002.png)

----
Structural Sharing

- Key to efficient 'copies' and therefore persistence
- Everything is final so no chance of interference
- Thread safe
- Iteration safe

----
Multi-component change

- Perceding was the easy part
- Many logical activities involve multiple data structures/multiple steps
- Two locking options
  - Coarse granularity locks
  - Fine granularity locks

----
