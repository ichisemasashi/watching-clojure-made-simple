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
