Simplicity Matters

シンプリシティ事項

Rich Hickey

----
> Simplicity is prerequisite for reliability

> シンプリシティは信頼性の前提条件です
Edsger W. Dijkstra

----
Word Origins
ワード起源

- Simpleシンプル
  sim- plex 単純な
  one fold/braid 1つ折り/編組
  vs complex 複雑な

- Easy 簡単
  ease < aise < adjacens  安楽<aise <adjacens
  lie near 近くに横たわる
  vs hard ハードと対戦
    
----
Simple シンプル

- One fold/braid 1つ折り/編組
  - One role 1つの役割
  - One task 1つのタスク
  - One concept 一つのコンセプト
  - One dimension 一次元

- But not だがしかし
  - One instance 1つのインスタンス
  - One operation 1回の操作
- About lack of interleaving, not cardinality カーディナリティではなくインターリーブの欠如について
- Objective 目的

----
Easy 簡単

- Near, at hand 近くに、手元に
  - on our hand drive, in our tool set, IDE, apt get, gem install 私たちの手のドライブ、ツールセット、IDE、apt get、gem install
- Near to our understanding/skill set 私たちの理解の近く/スキルセット
  - familiar おなじみの
- Near our capabilities 私たちの能力の近く
- Easy is relative 簡単です相対
- familiar おなじみの

----
Limits 限界

- We can only hope to make reliable those things we can understand われわれは理解できるものを信頼できるものにしたい
- We can only consider a few things at a time 我々は一度にいくつかのことしか考えない
- Interwined things must be considered together つながれたものは一緒に考えなければならない
- Complexity undermines understanding 複雑さが理解を損なう

----
Change 変化する

- Do more, Do it differently, Do it better もっとやりなさい、それを違うやり方で、より良くする
- Changes to software require analysis and decisions ソフトウェアの変更は分析と決定を必要とする
- Your ability to reason about your program is critical あなたのプログラムについてのあなたの能力は重要です
  - More than tests, types, tools, process テスト、タイプ、ツール、プロセス以上のもの

----
Simplicity = Opportunity シンプルさ=機会

- Architectural Agility wins Architectural Agilityが勝利
  - else - push the elephant それ以外は - ゾウを押す
- Design is about pulling things apart デザインは物事を引き離すことです
- Repurpose, substitute, move, combine, extend 複製、置換、移動、結合、拡張

----
> LISP programmers know the value of everything and the cost of nothing.

> LISPプログラマーは、すべての価値と何の費用も知っています。

Alan Perlis

----
Programmers know the benefits of everything and the tradeoffs of nothing.

プログラマは、すべてのメリットと何のトレードオフも知っています。

----
Programmers vs Programs

プログラマー対プログラム

- We focus on ourselves 我々は自分自身に焦点を当てる
  - programmer convenience プログラマの利便性
  - programmer replaceability プログラマの交換可能性
- Rather than the programs むしろプログラム
  - software quality, correctness ソフトウェアの品質、正当性
  - maintenance, change メンテナンス、変更
- gem install hairball

----
Complect

- To interleave, entwine, braid インターリーブするには、絡み合わせる
- Don’t do it! しないでください！
  - Complecting things is the source of complexity 複雑さの原因となるのは複雑なものです
- Best to avoid in the first place 最初に避けるのがベスト

----
Making Things Easy 簡単にする

- Bring to hand by installing インストールして手に入れる
  -  getting approved for use 使用が承認される
- Become familiar by learning, trying 学習、試してみてください。
- But mental capability? しかし、精神的能力？
  - not going to move very far 遠くに移動するつもりはない
- Make challenges easy by simplifying them それらを単純化することによって挑戦を容易にする

----
We can be creating the exact same programs out of significantly simpler components

大幅に単純なコンポーネントから正確に同じプログラムを作成することができます

----
What’s in your Toolkit? あなたのツールキットには何がありますか？

|Complexity|Simplicity|
|State, Objects|Values|
|Methods|Functions,Namespaces|
|Inheritance, switch, matching|Polymorphism ala carte|
|Syntax|Data|
|Imperative loops, fold|Set functions|
|Actors|Queues|
|ORM|Declarative data manipulation|
|Conditionals|Rules|
|Inconsistency|Consistency|

----
~~Simplicity--the art of maximizing the amount of work not done--is essential.~~
~~http://agilemanifesto.org/principles.html~~

----
> Simplicity is not an objective in art, but one achieves simplicity despite one's self by entering into the real sense of things

Constantin Brancusi

----
Lists and Order

- A sequence of things
- Does order matter?
  - [first-thing second-thing third-thing ...]
  - [depth width height]
- set[x y z]
  - order clearly doesn’t matter

----
Why Care about Order?

- Complects each thing with the next
- Infects usage points
- Inhibits change
  -  [name email] -> [name phone email]
- “We don’t do that”

----
Order in the Wild

|Complex |Simple|
|Positional arguments |Named arguments or map|
|Syntax |Data|
|Product types |Associative records|
|Imperative programs |Declarative programs|
|Prolog |Datalog|
|Call chains |Queues|
|XML |SON, Clojure literals|
|...||

----
Maps (aka hashes), Dammit!

- First class associative data structures
- Idiomatic support
  - literals, accessors, symbolic keys...
- Generic manipulation
- Use ‘em

----
Information is Simple

- Don’t ruin it
- By hiding it behind a micro-language
  - i.e. a class with information-specific methods
  - thwarts generic data composition
  - ties logic to representation du jour
- Represent data as data

----
Encapsulation

- Is for implementation details
- Information doesn’t have implementation
  - Unless you added it - why?
- Information will have representation
  - have to pick one

----
Wrapping Information

- The information class:
   - IPersonInfo{
       getName();
       ... verbs and other awfulness ...}
- A service based upon it:
  - IService{
      doSomethingUseful(IPersonInfo); ...}

----
Can You Move It?

- Litmus test - can you move your subsystems?
  - out of proc, different language, different thread?
- Without changing much
  - Not seeking transparency here

----
Subsystems Must Have

- Well-defined boundaries
- Abstracted operational interface (verbs)
- General error handling
- Take/return data
  - IPersonInfo - oops!
- Again, maps (hashes)

----
Simplicity is a Choice

- Requires vigilance, sensibilities and care
- Equating simplicity with ease and familiarity is wrong
  - Develop sensibilities around entanglement
- Your 'reliability' tools (testing, refactoring, type systems)
  - don't care if program is simple or not
- Choose simple constructs

----
Simplicity Matters

- Complexity inhibits understanding
  - and therefor robustness
- Simplicity enables change
  - It is the primary source of true agility
- Simplicity = Opportunity
- Go make (simple) things

----
> Simplicity is the ultimate sophistication.

Leonardo da Vinci
