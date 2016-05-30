# Java Datalog Engine with Semi-Naive Evaluation and Stratified Negation

Datalog is a subset of the Prolog programming language that is used as a query language in deductive databases.

## Introduction

A Datalog program consists of facts and rules. Facts describe knowledge about the world. Rules describe the
relationships between facts from which new facts can be derived.

The following Datalog program describes that Alice is a parent of Bob and Bob is a parent of Carol, and then
provides rules for deriving an ancestor relationship from the facts:

    parent(alice, bob).
    parent(bob, carol).
    
    ancestor(X, Y) :- parent(X, Y).
    ancestor(X, Y) :- ancestor(X, Z), parent(Z, Y).

Variables in Datalog are capitalized. In the example, `X`, `Y` and `Z` are variables, whereas `alice` and `bob`
are constants. Facts cannot contain variables - they are said to be _ground_.

The collection of facts is called the _Extensional Database_ (EDB).

In the fact `parent(alice, bob)` the `parent` is called the predicate, while `alice` and `bob` are the
terms. The number of terms is called the _arity_. The arity of `parent` is 2 and some literature will write it
as `parent/2`. It is expected that all facts with the same predicate will have the same arity.

In the example, the two facts

 * `parent(alice, bob)` reads "`alice` is a parent of `bob`"
 * `parent(bob, carol)` reads "`bob` is a parent of `carol`"

The collection of rules is called the _Intensional Database_ (IDB). Rules consist of a _head_ and a _body_, separated
by a `:-` symbol. The head of the rule describes a new fact that can be derived whereas the body describes how that
fact should be derived.

In the rule `ancestor(X, Y) :- parent(X, Y)` the `ancestor(X, Y)` is the head, and `parent(X, Y)` is
the body. It specifies that the fact "`X` is an ancestor of `Y`" can be derived if the fact "`X` is a parent of `Y`"
holds true.

Using this rule, Datalog will determine that "`alice` is an ancestor of `bob`" and "`bob` is an ancestor of `carol`"
when queries are executed.

The second rule `ancestor(X, Y) :- ancestor(X, Z), parent(Z, Y)` says that the fact "`X` is an ancestor of `Y`"
can also be derived if there exists a `Z` such that "`X` is an ancestor of `Z`" _and_ "`Z` is a parent of `Y`".

Using this rule, Datalog will determine that "`alice` is an ancestor of `carol`" from all the other facts that have already
been derived.

Queries can be run against the database once the facts and the rules have been entered into the system:

 * `ancestor(X, carol)?` queries "who are `carol`'s ancestors?"
 * `ancestor(alice, Y)?` queries "of who is `alice` the ancestor?"
 * `ancestor(alice, carol)?` asks "Is `alice` an ancestor of `carol`?"

Answers come in the form of a collection of the mapping of variable names to values that satisfy the query. For example, the
query `ancestor(X, carol)?`'s results will be `{X: alice}` and `{X: bob}`

### Fluent API

In addition to a parser for the Datalog language, JDatalog also provides an API through which the database can be accessed and
queried directly in Java programs.

The following is an example of how the facts and the rules from above example can be written using the Fluent API:

    JDatalog jDatalog = new JDatalog();
    
    jDatalog.fact("parent", "alice", "bob")
        .fact("parent", "bob", "carol");
    
    jDatalog.rule(Expr.expr("ancestor", "X", "Y"), Expr.expr("parent", "X", "Z"), Expr.expr("ancestor", "Z", "Y"))
        .rule(Expr.expr("ancestor", "X", "Y"), Expr.expr("parent", "X", "Y"));

The queries can then then be executed as follows:

    Collection<Map<String, String>> answers;
    answers = jDatalog.query(Expr.expr("ancestor", "X", "carol"));

The Javadoc documentation contains more information and the unit tests contain some more examples.

### Implementation

JDatalog's evaluation engine is bottom-up, semi-naive with stratified negation.

_Bottom-up_ means that the evaluator will start with all the known facts in the EDB and use the rules to derive new facts.
It will repeat this process until no more new facts can be derived. It will then match all of the facts to the goal of the
query to determine the answer (The alternative is _top-down_ where the evaluator starts with a series of goals and use the
rules and facts in the database to prove the goal).

_Semi-naive_ is an optimization wherein the evaluator will only consider a subset of the rules that may be affected by facts
derived during the previous iteration, rather than all of the rules in the IDB.

_Stratified negation_ arranges the order in which rules are evaluated in such a way that negated goals "makes sense". Consider,
for example, the rule `p(X) :- q(X), not r(X).`: All the `r(X)` facts must be derived first before the `p(X)`
facts can be derived. If the rules are evaluated in the wrong order then the evaluator may derive a fact `p(a)` in one
iteration and then derive `r(a)` in a future iteration which will contradict each other.

Stratification also puts additional constraints on the usage of negated expressions in JDatalog, which the engine checks for.

In addition JDatalog implements some built-in predicates: equals "=", not equals "<>", greater than ">", greater or
equals ">=", less than "<" and less or equals "<=".

### Notes, Features and Properties

* The engine implements semi-naive bottom-up evaluation.
* It implements stratified negation, or _Stratified Datalog&not;_.
* It can parse and evaluate Datalog programs from files and Strings (actually anything that implements `java.io.Reader`).
* It has a fluent API through which it can be embedded in Java applications to run queries. See the unit tests for examples.
* It implements "=", "<>" (alternatively "!="), "<", "<=", ">" and ">=" as built-in predicates.
* It avoids third party dependencies because it is a proof-of-concept. It should be able to stand alone.
* Values with "quoted strings" are supported.

## Usage

### With Maven

The preferred method of building JDatalog is through [Maven](https://maven.apache.org/).

    # Compile like so:
    mvn package
    
    # Generate Javadocs
    mvn javadoc:javadoc

    # Run like so:
    java -jar target\jdatalog-0.0.1-SNAPSHOT.jar [filename]
    
### With Ant

An [Ant](http://ant.apache.org/) build.xml file is also provided:

    # Compile like so:
    ant 
    
    # Generate Javadocs
    ant docs
    
    # Run like so:
    java -jar dist\jdatalog-0.0.1.jar


## License

JDatalog is licensed under the [Apache license version 2](http://www.apache.org/licenses/LICENSE-2.0):

    Copyright 2015-2016 Werner Stoop
    
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

## References:

* [wiki]  Wikipedia topic, http://en.wikipedia.org/wiki/Datalog
* [elma]  Fundamentals of Database Systems (3rd Edition); Ramez Elmasri, Shamkant Navathe
* [ceri]  What You Always Wanted to Know About Datalog (And Never Dared to Ask); Stefano Ceri, Georg Gottlob, and Letizia Tanca
* [bra1]  Deductive Databases and Logic Programming; Stefan Brass, Univ. Halle, 2009
            http://dbs.informatik.uni-halle.de/Lehre/LP09/c6_botup.pdf
* [banc]  An Amateur’s Introduction to Recursive Query Processing Strategies; Francois Bancilhon, Raghu Ramakrishnan
* [mixu]  mixu/datalog.js; Mikito Takada, https://github.com/mixu/datalog.js
* [kett]  bottom-up-datalog-js; Frederic Kettelhoit http://fkettelhoit.github.io/bottom-up-datalog-js/docs/dl.html
* [davi]  Inference in Datalog; Ernest Davis, http://cs.nyu.edu/faculty/davise/ai/datalog.html
* [gree]  Datalog and Recursive Query Processing; Todd J. Green, Shan Shan Huang, Boon Thau Loo and Wenchao Zhou
            Foundations and Trends in Databases Vol. 5, No. 2 (2012) 105–195, 2013
            http://blogs.evergreen.edu/sosw/files/2014/04/Green-Vol5-DBS-017.pdf
* [bra2]  Bottom-Up Query Evaluation in Extended Deductive Databases, Stefan Brass, 1996
            https://www.deutsche-digitale-bibliothek.de/binary/4ENXEC32EMXHKP7IRB6OKPBWSGJV5JMJ/full/1.pdf
* [sund]  Datalog Evaluation Algorithms, Dr. Raj Sunderraman, 1998
            http://tinman.cs.gsu.edu/~raj/8710/f98/alg.html
* [ull1]  Lecture notes: Datalog Rules Programs Negation; Jeffrey D. Ullman;
            http://infolab.stanford.edu/~ullman/cs345notes/cs345-1.ppt
* [ull2]  Lecture notes: Datalog Logical Rules Recursion; Jeffrey D. Ullman;
            http://infolab.stanford.edu/~ullman/dscb/pslides/dlog.ppt
* [meye]  Prolog in Python, Chris Meyers, http://www.openbookproject.net/py4fun/prolog/intro.html
* [alec]  G53RDB Theory of Relational Databases Lecture 14; Natasha Alechina;
            http://www.cs.nott.ac.uk/~psznza/G53RDB07/rdb14.pdf
* [rack]  Datalog: Deductive Database Programming, Jay McCarthy, https://docs.racket-lang.org/datalog/
            (Datalog library for the Racket language)

## Ideas and Notes

*Just some thoughts on how the system is currently implemented and how it can be improved in the future*

TODO: I could've named the program Jatalog. _Catchy!_

----

The purpose of passing a `Map<String, String>` containing the bound variables is for having the equivalent of 
JDBC *prepared statements*, to allow statements like, for example `jDatalog.query(new Expr("foo","X", "Y"), binding)`, 
with `binding = {X : "bar"}`, in the fluent API to perform bulk inserts or queries and so on.

Because the varargs ... operator must come last in the method declaration, I only have the bindings in the method that accepts 
the `List<Expr>` as an argument.

I now need to create a method `prepareStatement()` that can return a `List<Expr>` from a parsed query.

Actually, the `List<Expr>` should be wrapped in a `JStatement` (or something) interface so that you can run insert rules, insert facts and delete facts through these *prepared statements*.

----

There are opportunities to run some of the methods in parallel using the Java 8 Streams API (I'm thinking of the calls to 
`expandStrata()` in `buildDatabase()` and the calls to `matchRule()` in `expandStrata()` in particular).

I've now gone through the effort of removing the `DatalogException`s from `expandStrata()` on down to open the road for this
implementation. `Expr#evalBuiltIn()` may throw a `RuntimeException` for one of a number of conditions which are supposed to be
caught earlier, like in `Rule#validate()`.

----

The Racket language has a Datalog module as part of its library [rack]. I've looked at its API for ideas for my own. They use the syntax `<clause>~` for a retraction, e.g `parent(bob, john)~`, which is a syntax I might want to adopt. The [rack] implementation lacks some of the features of my version, like negation and queries with multiple goals.

----

I've decided against arithmetic built-in predicates, such as `plus(X,Y,Z) => X + Y = Z`, for now:

* Arithmetic predicates aren't that simple. They should be evaluated as soon as the input variables (X and Y) in this case becomes available, so that Z can be computed and bound for the remaining goals.
* Arithmetic expressions would require a more complex parser and there would be a need for `Expr` to have child `Expr` objects to build a parse tree. The parse tree would be simpler if the `terms` of `Expr` was a `List<Object>` - see my note above.

----

There are several opportunities to optimize the EDB.

You can trim facts before you start with `expandDatabase()` so that you only evaluate facts that are relevant to your goals.
So, for example, if your goal is related to "cousins" then you can filter out facts related to "employment".
The key is in this line in `query(List<Expr> goals)`:

    IndexedSet<Expr,String> facts = new IndexedSet<>(edbProvider.allFacts());

You'll have to replace it with something that only builds `facts` from facts that are relevant to the current query in the same way 
that you filter the rules in `getRelevantRules()`. 

The EDB is now abstracted behind an `EdbProvider` interface, but it will need a method `Collection<Expr> getFacts(String predicate)` 

It is intended that users of the library will be able to use different sources for the EDB, such as a SQL database, CSV or XML files. For
example, an EDB that is backed by a database can do a `SELECT * FROM predicate` when necessary. 

The SQL idea will require statements like `query = "SELECT * FROM " + predicate;` to manage it, so you'd better first verify that 
the `predicate` is only an alpha-numeric string.

----

It is conceptually possible to make the `List<String> terms` of `Expr` a `List<Object>` instead, so that you can store complex Java objects in the database (as POJOs). 

The `isVariable()` method will just have to be modified to check whether its parameter is `instanceof` String and starts with an upper-case character, the bindings will become a `Map<String, Object>`, the result of `query()` will be a `List<Map<String, Object>>` and a couple of `toString()` methods will have to be modified. 

It won't be that useful a feature if you just use the parser, but it could be a *nice-to-have* if you use the fluent API. I don't intend to implement it at the moment, though.
