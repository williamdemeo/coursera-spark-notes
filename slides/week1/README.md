 
[Big data analysis with Scala and Spark](https://www.coursera.org/learn/scala-spark-big-data)

Original slides by **Heather Miller**

Transcribed in Markdown by **William DeMeo**

---

# Week 1

## 1.1 Introduction, Logistics, What You'll Learn

### So far in the Scala courses...

**Focused on:**

+ Basics of Functional Programming. Slowly building up on fundamentals.

+ Parallelism. Experience with underlying execution in shared memory parallelism.

---

### So far in the Scala courses...

**Focused on:**

+ Basics of Functional Programming. Slowly building up on fundamentals.
+ Parallelism. Experience with underlying execution in shared memory parallelism.

**This course:**
Not a machine learning or data science course!  
+ This is a course about distributed data parallelism in Spark.  
+ Extending familiar functional abstractions like functional lists over large clusters.  
+ Context: analyzing large data sets.  

---

### Why Scala? Why Spark?

**Normally:**  
Data science and analytics is done "in the small," in R/Python/MATLAB, etc

**If your dataset ever gets too large to fit into memory,**  
these languages/frameworks won't allow you to scale. You’ve to reimplement
everything in some other language or system.

**Oh yeah, there’s also the massive shift in industry to data-oriented decision
making too!**  
...and many applications are "data science in the large"

---

### Why Scala? Why Spark?

**By using a language like Scala, it's easier to scale your small problem to
the large with Spark, whose API is almost 1-to-1 with Scala's collections.**

That is, by working in Scala, in a functional style, you can quickly scale your
problem from one node to tens, hundreds, or even thousands by leveraging
Spark, successful and performant large-scale data processing framework which
looks a and feels a lot like Scala Collections!

---

### Why Spark?

**Spark is...**

+ **More expressive.** APIs modeled after Scala collections. Look like functional lists! Richer, 
  more composable operations possible than in MapReduce.

+ **Performant.** Not only performant in terms of running time... 
  But also in terms of developer productivity! Interactive!

+ **Good for data science.** Not just because of performance, but because it enables 
  iteration, which is required by most algorithms in a data scientist's toolbox.

+ **Good to know**  
  Spark and Scala skills are in extremely high demand!

---

### In this course you'll learn...

+ Extending data parallel paradigm to the distributed case, using Spark.

+ Spark's programming model

+ Distributing computation, and cluster topology in Spark 

+ How to improve performance; data locality, how to avoid recomputation and shuffles in Spark.

+ Relational operations with DataFrames and Datasets

---

### Prerequisites

**Builds on the material taught in the previous Scala courses.**

+ [Principles of Functional Programming in Scala](https://www.coursera.org/learn/progfun1)
+ [Functional Program Design in Scala](https://www.coursera.org/learn/progfun2)
+ [Parallel Programming (in Scala)](https://www.coursera.org/learn/parprog1)

Or at minimum, some familiarity with Scala.

---

### Books, Resources

Many excellent books released in the past year or two!


+ **Spark in Action** (2017), by Zecevic, Bonaci

+ **Learning Spark** (2015) by Karau, Konwinski, Wendell, Zaharia

+ **High Performance Spark** (in progress), by Karau, Warren

+ **Advanced Analytics with Spark** (2015), by Ryza, Laserson, Owen, Wills

+ [Mastering Apache Spark 2](https://www.gitbook.com/book/jaceklaskowski/mastering-apache-spark/details) (in progress), by Laskowski

---

### Tools

As in all other Scala courses...

+ IDE of your choice
+ sbt
+ [Databricks](https://databricks.com/) Community Edition (optional)  
  Free hosted in-browser Spark notebook. Spark "cluster" managed by
  Databricks so you don’t have to worry about it. 6GB of memory for
  you to experiment with.
  
---

### Assignments

**Like all other Scala courses, this course comes with autograders!**

**Course features 3 auto-graded assignments that require
you to do analyses on real-life datasets.**

--

## 1.2 Data-Parallel to Distributed Data-Parallel

### Visualizing **Shared Memory** Data-Parallelism

What does data-parallel look like?

```scala
val res = 
  jar.map(jellyBean => doSomething(jellyBean))
```

**Shared memory data parallelism:**
+ Split the data
+ Workers/threads independently operate on the data shards in parallel
+ Combine when done (if necessary)

**Scala's Parallel Collections is a collections abstraction over shared memory data-parallel execution.**

---

### Visualizing **Distributed** Data-Parallelism

What does **distributed** data-parallel look like?

```scala
val res = 
  jar.map(jellyBean => doSomething(jellyBean))
```

**Distributed data parallelism:**
+ Split the data *over several nodes*
+ *Nodes* independently operate on the data shards in parallel
+ Combine when done (if necessary)

**New Concern:** Now we have to worry about network latency!

**However, like parallel collections, we can keep collections abstractions over distributed data-parallel execution.**

---

### Data-parallel to **Distributed** Data-Parallel

**Shared memory case:** Data-parallel programming model.  Data is partitioned in (shared) memory and operated upon in parallel.

**Distributed case:** Data-parallel programming model.  Data is partitioned and distributed 
across machines, network in between, operated upon in parallel.

Overall, most all properties and aspects of
shared memory data-parallel collections that we learned about 
previously can also be applied to their distributed counterparts. 
*E.g., watch out for non-associative reduction operations!*

**However, we must now consider _latency_ when using a distributed model**

---

### Apache Spark

Throughout this part of the course we will use the Apache Spark framework for distributed data-parallel programming.

**Spark implements a distributed data parallel model called**

Resilient Distributed Datasets (RDDs)

---

### Distributed Data-Parallel: High Level Illustration

Given some large dataset that can't fit into memory on a single node...

...chunk up the data and distribute it over your cluster of machines.

From there, think of your distributed data like a single collection...

**Example**  
Transform the text (not titles) of all wiki articles to lowercase.

```scala
val wiki: RDD[WikiArticle] = ...

wiki.map {
  article => article.text.toLowerCase
}
```

---

## 1.3 Latency

### Data-Parallel Programming

In the Parallel Programming course, we learned:

+ Data parallelism on a single multicore/multi-processor machine
+ Parallel collections as an implementation of this paradigm

Today:

+ Data parallelism in a *distributed* setting
+ Distributed collections abstraction from Apache Spark as an implementation of this paradigm.

---

### Distribution

Distribution introduces important concerns beyond what we had to worry about when dealing with paralleism in the shared-memory case:

+ **Partial failure**: crash failures of a subset of the machines involved in a distributed computation.

+ **Latency**: certain operations have a much higher latency than other operations due to network communication.

**Latency cannot be masked complete; it will be an important aspect that also impacts the programming model**

---

### Important Latency Numbers

**Main memory reference**: 100ns

**Read 1 Mb sequentially from memory**  250,000ns 

**Read 1 Mb sequentially from SSD**  1,000,000ns

**Read 1 Mb sequentially from (non SSD) disk**  20,000,000ns

**Send packet US -> Europe -> US**: 150ms

NB sending packets between continents is *1 million times* slower than main memory references.

---

### Latency Number Intuitively

(omitting slides about "humanized" latency numbers)

---

### Big Data Processing and Latency?

With some intuition now about how expensive network communication and 
disk ops can be, one may ask:

*How do these latency numbers relate to big data processing?*

To answer this question, let's first start with Spark's prececessor, Hadoop.

---

### Hadoop/MapReduce

Hadoop is a widely-used large-scale batch data processing framework.
It's an open source implementation of Google's MapReduce.

**MapReduce was ground-breaking because it provided:**

+ a simple API (simple `map` and `reduce` steps)
+ **fault tolerance**

**Fault tolerance** is what made it possible for Hadoop/MapReduce to 
scale to 100s or 1000s of nodes at all.

---

### Hadoop/MapReduce + Fault Tolerance

**Why is this important?**

For 100s or 1000s of old commodity machines, likelihood of at least one node failing 
midway through the job is **very high**.

Thus, Hadoop/MapReduce's ability to recover from node failure enabled
computations on previously unthinkably large data sets to proceed to completion.

**Fault tolerance + simple API** made it possible for average Google software 
engineer to craft a complex pipeline of map/reduce stages on extremely large data sets.

---

### Why Spark?

**Fault tolerance in Hadoop/MapReduce comes at a cost**

Between each map and reduce step, in order to recover from potential failures, Hadoop/MapReduce shuffles its data and writes intermediate data to disk.

**Reading/writing to disk is 100 x slower than in-memory**

**Network communication is 1 mil x slower than in-memory**

---

### Why Spark?

**Spark...**

+ Retains fault-tolerance
+ Different strategy for handling latency (latency significantly reduced!)

**Achieves this using ideas from functional programming!**

**Idea:** Keep all data **immutable and in-memory**. All operations on data are just 
functional transformations, like we do to regular Scala collections. Fault tolerance 
is achieved by replaying functional transformations over original dataset.

**Result:** Spark has been shown to 100 x more performant than Hadoop, while 
adding even more expressive APIs.

---

### Spark versus Hadoop

(omitting slides with graphs comparing Spark and Hadoop)

---

## 1.4 Resilient Distributed Datasets (RDDs), Spark's Distributed Collections

### Resilient Distributed Datasets (RDDs)

RDDs seem a lot like **immutable** sequential or parallel Scala collections.

```scala
abstract class RDD[T] {
  def map[U](f: T => U): RDD[U] = ...
  def flatMap[U](f: T => TraversableOnce[U]): RDD[U] = ...
  def filter(f: T => Boolean): RDD[T] = ...
  def reduce(f: (T,T) => T): T = ...
}
```

Most operations on RDDs, like Scala's immutable `List` and parallel collections, 
are higher-order functions.

*That is, they are methods that work on RDDs and take a function as an argument 
and typically return RDDs*

---
### Resilient Distributed Datasets (RDDs)

RDDs seem a lot like **immutable** sequential or parallel Scala collections.


|**Combinators on Scala parallel/sequential collections:** | **Combinators on RDDs:** |
| --- | --- |
|`map` | `map`|
|`flatMap` | `flatMap`|
|`reduce` | `reduce` |
|`fold` | `fold` |
|`aggregate` | `aggregate` |

---
### Resilient Distributed Datasets (RDDs)

While their signatures differ a bit, their semantics (macroscopicaly) are the same:

```scala
map[B](f: A => B): List[B] // Scala List
map[B](f: A => B): RDD[B] // Spark RDD

flatMap[B](f: A => TraversableOnce[B]): List[B] // Scala List
flatMap[B](f: A => TraversableOnce[B]): RDD[B] // Spark RDD

filter(pred: A => Boolean): List[A] // Scala List
filter(pred: A => Boolean): RDD[A] // Spark RDD
```

---
### Resilient Distributed Datasets (RDDs)

While their signatures differ a bit, their semantics (macroscopicaly) are the same:

```scala
reduce(op: (A, A) => A): A // Scala List
reduce(op: (A, A) => A): A // Spark RDD

fold(z: A)(op: (A, A) => A): A // Scala List
fold(z: A)(op: (A, A) => A): A // Spark RDD

aggregate[B](z: => B)(seqop: (B, A) => B, combop: (B, B) => B): B // Scala List
aggregate[B](z: B)(seqop: (B, A) => B, combop: (B, B) => B): B // Spark RDD
```

(Notice that `z : B` is not call-by-name in Spark's `aggregate`)

---

### Resilient Distributed Datasets (RDDs)

Using RDDs in Spark feels alot like Scala sequential/parallel collections, 
with the added knowledge that your data is distributed across several machines.

**Example:**  
Given, `val encyclopedia: RDD[String]`, say we want to search all of `encyclopedia` 
for mentions of EPFL, and count the number of pages that mention EPFL.

```scala
val result = encyclopedia.filter(page => page.contains("EPFL")).count()
```

---

### Example: Word Count

The "Hello, World!" of programming with large-scale data.

```scala
// Create an RDD
val rdd = spark.textFile("hdfs://...")

val count = rdd.flatMap(line => line.split(" ") // separate lines into words
        .map(word => (word, 1))          // include something to count
        .reduceByKey(_ + _)              // sum up the 1s in the pairs
```

That's it!

---

### How to create an RDD?

RDDs can be created in two ways:

1. Transforming an existing RDD
2. Form a `SparkContext` (or `SparkSession`) object

**Transforming an existing RDD**  
Just like a call to `map` on a `List` returns a new `List`, many higher-order
functions defined on an `RDD` return a new `RDD`.

**Form a SparkContext (or SparkSession) object**  
The `SparkContext` object (renamed `SparkSession`) can be thought of as your 
handle on the Spark cluster.  It represents the connection between the 
Spark cluster and your running application.  It defines a handful of 
methods that can be used to create and populate a new RDD:

+ `parallelize` -- convert a local Scala collection to an RDD
+ `textfile` -- read a text file from a **Hadoop distributed file system (HDFS)**
  or a local file system and return an RDD or `String`

---

## 1.5 Transformations and Actions

### Transformations and Actions

Recall **transformers** and **accessors** from Scala sequential and parallel collections.

**Transformers** return new collections as results (not single values)  
**Examples:** `map`, `filter`, `flatMap`, `groupBy`

```scala
map(f: A => B): Traversable[B]
```

**Accessors** return single values as results (not collections)  
**Examples:** `reduce`, `fold`, `aggregate`

```scala
reduce(op: (A,A) => A): A
```

---

### Transformations and Actions

Similarly, Spark defines **transformations** and **actions** on RDDs

These are similar to **transformers** and **accessors** but there are some important differences.

**Transformations** return new RDDs as results  
**They are lazy---their result RDD is not computed immediately.**

**Actions** compute a result based on an RDD, and either return this result or 
save it to external storage (eg, an HDFS).  
**They are eager---their result is computed immediately.**

**Laziness/eagerness** is how we can limit network communication using the programming model!!!

---

### Example

Consider the following simple example:

```scala
val largeList: List[String] = ...
val wordsRdd = sc.parallelize(largeList)
val lengthsRdd = wordsRdd.map(_.length)
```

What has happened on the cluster at this point?

**Nothing!** Execution of `map` (a transformation) is deffered.

To kick off the computation and wait for its result we add an **action** 
to the end of the code listing above:

```scala
val totalChars = lengthsRdd.reduce(_ + _)
```

---

### Common Transformations in the Wild

These are **LAZY!**

| Name | signature | description |
| --- | --- | --- |
| `map` | `map[B](f: A => B): RDD[B]` | apply function to each element in RDD and return the result in an RDD |
| `flatMap` | `flatMap[B](f: A => TraversableOnce[B]): RDD[B]` | apply function to each element in RDD and return the result in an RDD |
| `filter` | `filter(pred: A => Boolean): RDD[A]` | apply predicate to each element in RDD and return an RDD of elements that have passed the predicate condition |
| `distinct` | `distinct(): RDD[B]` | return RDD with duplicates removed |

---

### Common Actions in the Wild

These are **EAGER!**

| Name | signature | description |
| --- | --- | --- |
| `collect` | `collect(): Array[T]` | return all elements of the RDD |
| `count` | `count(): Long` | return the number of elements of the RDD |
| `take` | `take(num: Int)` | return the first `num` elements of the RDD |
| `reduce` | `reduce(op: (A, A) => A): A` | combine the elements in the RDD 
                                            using `op` and return result |
| `foreach` | `foreach(f: T => Unit): Unit` | apply `f` to each element in the RDD |

---

### Another Example

Let's assume that we have an `RDD[String]` which contains gigabytes of logs collected over the previous year. Each element of this RDD represents one line of logging.

Assuming that dates come in the form, `YYYY-MM-DD:HH:MM:SS`, and errors are logged with 
a prefix that includes the word "error" ...

**How would you determine the number of errors that were logged in Dec 2016?**

```scala
val lastYearsLogs: RDD[String] = ...
val numDecErrorLogs = lastYearsLogs.filter(lg => lg.contains("2016-12") && lg.contains("error"))
                                   .count()
```

---

### Benefits of Laziness for Large-Scale Data

Spark computes RDDs the first time they are used in an action.

This helps when processing large amounts of data.

**Example**

```scala
val lastYearsLogs: RDD[String] = ...
val firstLogsWithErrors = lastYearsLogs.filter(_.contains("Error")).take(10)
```

The execution of `filter` is deferred until the `take` action is applied.

Spark leverages this by analyzing and optimizing the **chain of operations** before executing it.

Spark will not compute intermediate RDDs. Instead, as soon as 10 elements of the 
filtered RDD have been computed, `firstLogsWithErrors` is done.  At this point Spark stops 
working, conserving the time and space that would have been required if we 
computed elements of the unused result of filter.

--- 

### Transformations on Two RDDs (LAZY!)

RDDs also support set-like operations, like `union` and `intersection`

The **Two-RDD transformations** combine two RDDs into one.

| Name | signature | description |
| --- | --- | --- |
| `union` | `union(other: RDD[T]): RDD[T]` | return an RDD containing elements from both RDDs |
| `intersection` | `intersection(other: RDD[T]): RDD[T]` | return an RDD containing the elements that appear in both RDDs |
| `subtract` | `subtract(other: RDD[T]): RDD[T]` | return an RDD with the contents of the other RDD removed |
| `cartesian` | `cartesian[U](other: RDD[U]): RDD[(T, U)]` | Cartesian product with the other RDD |

---

### Other useful RDD Actions (EAGER!)

RDDs also contain other important actions unrelated to regular Scala collections but which are useful when dealing with distributed data.

| Name | signature | description |
| --- | --- | --- |
| `takeSample` | `takeSample(withRepl: Boolean, num: Int): Array[T]` |  |
| `takeOrdered` | `takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T]` | |
| `saveAsTextFile` | `saveAsTextFile(path: String): Unit` | |
| `saveAsSequenceFile` | `saveAsSequenceFile(path: String): Unit` | |

---

## 1.6 Evaluation in Spark: Unlike Scala Collections!

### Why is Spark Good for Data Science?













1pHmiBQM BM aT`F, lMHBF2 a+H
*QHH2+iBQMb5
"B; .i MHvbBb rBi? a+H M/ aT`F
