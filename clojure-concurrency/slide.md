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

- Dynamic development  ダイナミックな開発
  - REPL, reader, on-the-fly compilation to JVM bytecode
- Primitives - numbers, including arbitary-precision integers & ratios, characters, strings, symbols, keywords, regexes  プリミティブ - 任意精度の整数と比率、文字、文字列、シンボル、キーワード、正規表現などの数値
- Aggregates - lists, maps, sets, vectors  集計 - リスト、マップ、セット、ベクトル
  - read-able, persistent, immutable, extensible  読み取り可能、永続的、不変、拡張可能
- Abstract sequences + library  抽象シーケンス+ライブラリ
- Metadata   メタデータ
- First-class functions (fn), closures  ファーストクラス関数（fn）、クロージャ
- Recursive functional looping  再帰的な機能のループ
- Destructuring binding in let/fn/loop  let / fn / loopでの結合の破壊
- List comprehensions (for)  リスト内包（for）
- Macros  マクロ
- Multimethods  マルチメソッド
- Concurrency support  並行処理のサポート
- Java interop  Java相互運用性
  - Call methods, access fields, arrays  メソッドの呼び出し、フィールドへのアクセス、配列の呼び出し
  - Proxy interfaces/classes  プロキシインタフェース/クラス
  - Sequence functions extended to Java strings, arrays, Collections  シーケンス関数をJava文字列、配列、コレクションに拡張
  - Clojure data structures implement Collection/Callable/Iterable/Comparable etc where appropriate  Clojureデータ構造は、必要に応じてCollection / Callable / Iterable / Comparableなどを実装します
- Namespaces, zippers, XML and more!  名前空間、ジッパー、XMLなど

----
State - You're doing it wrong   - それは間違っている

- Mutable objects are the new spaghetti code  可変オブジェクトは新しいスパゲッティコードです
  - Hard to understand, test, reason about  理解するのが難しい、テストする、理由がある
  - Concurrency disaster  並行性障害
  - Terrible default architecture (Java/C#/Python/Ruby/Groovy/CLOS...)  ひどいデフォルトアーキテクチャ（Java / C＃/ Python / Ruby / Groovy / CLOS ...）
- Doing the right thing is very difficult  正しいことをすることは非常に難しい
  - Languages matter!  言語は重要です！

----
Concurrency
同時実行性

- Interleaved/simultaneous execution インターリーブ/同時実行
- Must avoid seeing/yielding inconsistent data 一貫性のないデータの表示/拒否を避けなければならない
- The more components there are to the data, the more difficult to keep consistent  データに対するコンポーネントが多くなればなるほど、一貫性を維持することが難しくなります
- The more steps in a logical change, the more difficult to keep consistent  論理的な変更のステップが増えるほど、一貫性を維持することが難しくなります
- Opportunities for automatic parallelism  自動並列処理の機会
  - Emphasis here on coordination  ここでのコーディネーションの強調

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
Coarse Granularity Locking

- Create external Lock representing a set of data structures
- Clients must obtail a lock to maiipulate any of the structures
- Each multi-part logical operation requires only one lock

----
Coarse Granularity Locking

x Safest
- Can be confusing as to what constitutes the set(s), what needs to be locked
  - X needs a/b/c, Y needs b/c/d
- Least throughput
  - Possible needless blocking
- Should read lock?

----
Fine Granularity Locking

- Use locks on data structures themselves
- Clients must obtain a lock on each of the structures
- A multi-part logical operation may require several locks

----
Fine Granularity Locking

- Dangerous
- Locking order is critical
  - X locks a/b, Y locks b/a - deadlock possible
  - Very difficult to enforce locking order
- Best throughput
  - Minimal blocking
- Should reads lock?
---
Concurrency Methods

- Conventional way:
  - Direct references to mutable objects
  - Lock and pray (manual/convention)
- Clojure way:
  - Indirect references to immutable persistent data structures
  - Concurrency semantics for references
    - Automatic/enforced
    - No locks!

----
Clojure References

- The only things that mutate are references themselves, in a controlled way
- 3 types of mutable references
  - Vars - Isolate changes within threads
  - Refs - Share synchronous coordinated changes between threads
  - Agents - Share asynchronous independent changes between threads

----
Vars

- Like Common Lisp's special vars
  - dynamic scope
  - stack discipline
- Shared root binding established by def
  - root can be unbound
- Can be changed (via set!) but only if first thread-locally bound using binding
- Functions stored in vars, so they too can be dynamically rebound
  - context/aspect-like idioms

----
Refs and Transactions

- Software transactional memory system (STM)
- Refs can only be changed within a transaction
- All changes are Atomic and Isolated
  - Every change to Refs made within a transaction occurs or none do
  - No transaction sees the effects of any other transaction while it is running
- Transactions are speculative
  - Will be retried automatically if conflict
  - Must avoid side-effects!

----
The Clojure STM

- Surround code with (dosync ...)
- Uses Multiversion Concurrency Control (MVCC)
- All reads of Refs will see a consistent snapshot of the 'Ref world' as of the starting point of the transaction, + any changes it has made.
- All changes made to Refs during a transaction will appear to occur at a single point in the timeline.
- Readers never block writers/readers, writers never block readers, supports commute

----
Agents

- Manage independent state
- State changes through actions, which are ordinary functions (state => new-state)
- Actions are dispatched using send or send-off, which return immediately
- Actions occur asynchronously on thread-pool threads
- Only one action per agent happens at a time

----
Agents

- Agent state always accessible, via deref/@, but may not reflect all actions
- Can coordinate with actions using await
- Any dispatches made during an action are held until after the state of the agent has changed
- Agents coordinate with transactions - any dispatches made during a transaction are held until it commits
- Agents are not Actors (Erlang/Scala)

----
Walkthrough

- Ant colony simulation
- World populated with food and ants
- Ants find food, bring home, drop pheromones
- Sense pheromones, food, home
- Ants act independently, on multiple real threads
- Model pheromone evaporation
- Animated GUI
- < 250 lines of Clojure

----
