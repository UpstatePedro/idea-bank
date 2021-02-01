# (Py)Spark

Spark is a programming model for distributed & parallelised in-memory data-processing.

Spark supports both batch & stream processing paradigms.

Spark is written in Scala, but provides Java & Python (PySpark) wrappers.

Components of a Spark deployment:

- Cluster: distributed worker nodes to carry out the processing
- Driver programme: schedules & issues instructions to the worker
- SparkContext: the object through which the driver programme access the spark environment (represents a connection to the cluster)
- "Resilient Distributed Dataset" (RDD): the main programming abstraction which represents the dataset on which the cluster is operating

RDD

- Read-only / immutable
- Distributed across the cluster in **partitions** such that it permits parallel operations
- Primarily in-memory, but different levels of storage are available (fallback to disk is possible) and intermediate RDDs can be cached if we expect to reuse them
- Preserves the data lineage to ensure partitions can be re-created in the event of a node failure (each new RDD contains a pointer to its parent RDD)
- All work in the spark cluster is carried out by operating on an RDD

## Operating on an RDD

Once created, an RDD enables two types of operation:
1. Transformation
- lazy operations to build new RDDs from other RDDs
- examples include `map`, `filter`, `join`
2. Action
- return a result or write it to storage
- examples include `count`, `collect`, `save`

| **Transformations** | **Actions** |
|:---------------:|:-------:|
|map(func)        | reduce(func) |
|flatMap(func)    | collect() |
|filter(func)     | count() |
|groupByKey()     | first() |
|reduceByKey(func)| take(n) |
|mapValues(func)  | saveAsTextFile(path) |
|sample()         | countByKey() |
|union(rdd)       | foreach(func) |
|distinct()       ||
|sortByKey()      ||

## Transformations

Transformations can be described as *narrow* or *wide*. These labels are used to describe the potential cardinality of the inputs that each RDD's output value might involve for a given transformation.
Narrow transformations might be a one-to-one mapping from input to output, whereas a wide transformation might draw on a large number of inputs to compute each output.
Due to the distributed nature of the partitions in the RDD, wide transformations can be slow / expensive due to the potentially large amount of data shuffling that could be required to get all the data in the right place to compute each output value.

## Execution of Spark programmes

When a job is submitted to a Spark cluster, Spark works through several steps during the course of the job's lifecycle:
1. Build a DAG of operations from the transformations & actions defined in the driver programme
2. Split the DAG into a series of staged tasks
3. Submit each stage of tasks for execution in the cluster as capacity allows
4. Manage, track & recover the tasks/RDDs to mitigate node failures

Due to this evident separation of (1) specifying the desired outcomes, from (2) decisions about *how* to achieve them, we can see that Spark follows a declarative programming paradigm.

## Resources

[Spark programming guide](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html)

[Talk by Amy Krause & Andreas Vroutses; EPCC](https://events.prace-ri.eu/event/896/sessions/2721/attachments/997/1669/Spark_Introduction.pdf)