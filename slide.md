Clojure, Made Simple

Rich Hickey

---
Me

- Creator of Clojure
- Designer of Datomic
- CTO/Co-Founder Cognitect.Inc
- Two decades delivering commercial production apps in C++/Java/C#

---
You

- (dis)contented Java dev?
- Clojure tinkerer?
  'wish I could use it at work'
- Clojure user
- Couldn't get into Brian Goetz' talk

---
Clojure

- Programming language for JVM, JavaScript et al
- Released in 2007
- Lisp, functional, data-oriented etc(more later)

---
"A lot of the best programmers and the most productive programmers I know are writing everything in ___ and swearing by it, and then just producting ridiculously sophisticated things in a very short time. And that programmer productivity matters."
Adrian Cockcroft - Battery Ventures, formerly Cloud Architect, Netflix

---
"A lot of the best programmers and the most productive programmers I know are writing everything in **Clojure** and swearing by it, and then just producting ridiculously sophisticated things in a very short time. And that programmer productivity matters."
Adrian Cockcroft
http://thenewstack.io/the-new-stack-makers-adrian-cockcroft-on-sun-netflix-clojure-go-docker-and-more/


---
Programming is an Economic Activity

- Cost/benefit
- ROI
- Time to market
- Profit

---
Stakeholders Want

- Something good
- Soon

---
Something Good

- ~~Passes all tests~~
- ~~Type checks~~
 **Programmer-centric means, not ends**
- Does what it is supposed to do
- Meets operational requirements
- Is flexible enough to accommodate change

---
Does what it is Supposed To Do?

- From the perspective of the stakeholders
- Very difficult to determine if a large, elaborate stateful program does what it is supposed to do
- Very difficult to determine if large or elaborate or stateful programs do what they are supposed to do

---
Meets Operational Requirements

- Deployment/environment  <- Shared with Java/
- Security                <- JavaScript host 
- Performance
- Etc.

---
![figure-001](figure-001.png)
http://benchmarksgame.alioth.debian.org/u64q/benchmark.php?test=all&lang=all&data=u64q

---
![figure-002](figure-002.png)
Om(ClojureScript lib) on browser ToDoMVC Benchmark
http://vuejs.org/perf/

---
Flexibility

- Ability to accommodate inevitable change
- Loose coupling is key

---
"With Clojure we get to market faster and with better quality. We avoid unintended interruptions in Java apps when code in one area impacts the application in another."

"Clojure shrinks our code base to about one-fifth the size it would be if we had written in Java."

Anthony Marcar - Senior Architect, WalmartLabs

---
"With Clojure we (get to market faster) and with (better quality). We (avoid) unintended interruptions in Java apps when code in (one area impacts) the application in (another)."

"Clojure (shrinks our code base) to about one-fifth the size it would be if we had written in Java."

Anthony Marcar - Senior Architect, WalmartLabs

---
How?

- Data orientetion
- Simplicity

What matters is not just what a programming language makes possible, but what it makes practical and idiomatic.

---
Data Processing

- Not a dirty word (nor two)
- Most programs acquire, transform, store, search, manage, transmit data
- Data is raw, immutable information
- Many langs turn into something much more elaborate - with types 'methods' etc

esp. OO conflates process constructs and information constructs

---
Data

- Clojure embraces data
- Data literals
- Code is data
- Majority of functions take/return data
- Information is represented in Clojure systems as plain data

---
Atomic Data Types

- Integers - 12345678987654
- Doubles 1.234, BigDecimals 1.234M
- Ratios - 22/7
- Strings - "fred", Characters - \a \b \c
- Symbols - fred ethel, Keywords - :fred :ethel
- Booleans - true, false, Null - nil
- Regex patterns #"a*b"

---
