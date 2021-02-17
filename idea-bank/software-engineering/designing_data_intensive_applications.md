# Book summary: Designing Data-Intensive Applications (WIP)

## Preface

Trends affecting the way we build (data-intensive web) applications:

- Massive increase in the volume of data
- Parallel execution becoming more commonplace networks are getting faster & CPU clock speeds are stabilising
- Serverless paradigm
- Applications are expected to be highly available

We refer to an application as "data-intensive" if data is the primary challenge; quantity / complexity / speed.

Applications where CPU cycles are the bottleneck are "compute-intensive". 

New tools to help data-intensive applications:

- NoSQL databases
- Message queues
- Caches
- Search indices
- Batch & stream processing frameworks

The purpose of this book is to ***find useful ways of thinking about data systems - not just how they work.***

### Structure of the book:

1. Fundamental ideas of designing data-intensive apps
    1. What are we trying to achieve?
        1. Reliability
        2. Scalability
        3. Maintainability
    2. Different data models & query languages
    3. Storage engines
    4. Data encoding
2. Distributed systems
    1. Replication
    2. Partitioning / sharding
    3. Transactions
    4. Problems with distributed systems
    5. Achieving consistency / consensus
3. Derived data
    1. Batch processing
    2. Stream processing

## Part one: Foundations of Data Systems

### Chapter 1: Reliable, Scalable, and Maintainable Applications

DDIAs are typically built from common building-blocks:

1. Databases: store date to be retrieved later
2. Caches: remember the results of expensive operations, speed up reads
3. Search indices: enable the data to be searched / filtered in useful ways
4. Stream-processing: send messages to other processes, to be handled asynchronously
5. Batch-processing: periodically process accumulated data

These are the general components from which we can fashion our own special-purpose 'data systems'.

There are three characteristics that we must consider when designing data/software systems:

1. **Reliability**: the system should continue to work ***correctly*** even in the face of ***adversity***
2. **Scalability**: the system should be capable of ***growing*** gracefully
3. **Maintainability**: it should be possible to work ***productively*** on the system - both maintaining its current behaviours, and modifying them

#### Reliability

A reliable system:

- behaves consistently & as the user expects
- is tolerant of user-errors of unexpected usage
- performs at an acceptable level for the required use-case, under the expected load
- prevents unauthorized access & abuse

We can consider these items together to mean "working correctly"

Risks that threaten the system's ability to continue working correctly are called **faults**. A system that is robust to faults is known as **fault-tolerant** or **resilient**.

Robust systems continue to work correctly even when faults occur (ie. a system component's behaviour deviates from its spec). If a system stops working correctly, we refer to this as **a failure**.

Many critical bugs are actually a result of poor error handling. Counter-intuitively, we can increase the fault-tolerance of a system by triggering faults deliberately to ensure that systems are designed in a fault-tolerant way, and are continuously tested as such - better to do this on your own terms that find out it doesn't work when it really matters.

We therefore generally prefer to engineer to ***tolerate faults*** instead of trying to prevent them.

This isn't true in areas such as security.

**System outages are predominantly caused by human error**: One study of large internet services found that
configuration errors by operators were the leading cause of outages, whereas hard‐
ware faults (servers or network) played a role in only 10–25% of outages

There is often a trade-off between reliability, development cost & operational cost. 

#### Scalability

***How does the reliability of the system change as the load changes?***

The load of a system is measured using ***load parameters*** - parameters that help to describe how intensively the system is being used. These are specific to the architecture of the system.

eg.s:

- Requests per second
- Ratio of reads to writes on the DB
- Number of concurrent users
- Hit rate on a cache

**Describing performance**

There are a couple of ways that we can approach the question of how increased load affects the system performance:

1. When a load parameter increases and system resources are held constant, how does system performance change?
2. When a load parameter increases, how much do system resources need to increase in order to keep system performance constant?

> I prefer the latter, because this is generally what we want to achieve (keeping system performance constant, rather than system resources)

But in either example, we need to be able to describe performance of the system...

- Response time
- Throughput

Look at performance metrics as a distribution and report in percentiles, rather than just looking at the mean.

eg. p50, p95, p99

NB. Backend systems often require that the user make multiple calls in order to complete a single task. In this case, the request time can never be faster than the slowest call. As the number of calls required to complete the task increases, the probability that one of the calls will be 'slow' increases, and so a higher proportion of users end up with slow response times. This is known as **tail latency amplification**.

**Coping with growth in load**

Expect to change your architecture with every order of magnitude increase in load, if not more frequently.

Approaches:

- Vertical scaling: moving onto higher spec machines
- Horizontal scaling: spreading the load across a larger number of machines

Distributing load across multiple machines is called ***shared-nothing*** architecture.

Scaling services across many machines is much easier if the service is stateless.

An architecture that scales well will need to be designed based on sound assumptions about which operations will be common or rare, and how they will grow.

#### Maintainability

The majority of the cost of building software is not in its initial development, but in its ongoing maintenance (both fixes & modifications).

We should therefore design software so as to minimize pain during maintenance. Three design principles can help us to this end:

1. **Operability**: Make it easy for the ops team to keep the system running smoothly 
2. **Simplicity**: Make it easy for new engineers & a future you to understand the system
3. **Evolvability**: Make it easy to make changes to the system

**Operability**

> "good ops can often work around the limitations of bad software, but good software cannot run reliably with bad ops"

Ops teams are generally responsible for:

- Monitoring system health & responding to issues
- Debugging system failures & performance issues
- Security patching & keeping dependency versions up to date
- Keeping on top of interactions between systems
- Capacity planning
- CI / CD practices + tools
- Complex maintenance
- Defining processes to keep ops predictable & the production env stable

**Simplicity**

As software projects grow, they often become complex; symptoms of which include:

- explosion of the state space
- tight coupling of modules
- tangled dependencies
- inconsistent naming & terminology
- performance hacks
- special cases

Complexity makes maintainability difficult, and that makes development slower, more expensive, and riskier.

**Evolvability**

System requirements will change over time.

The ease with which changes to system requirements can be accommodated is closely linked to simplicity and abstractions.

#### Summary

In order to be useful / valuable, applications need to meet a range of requirements. There are two categories that we can split requirements across:

- Functional: the behaviours that the application enables
- Non-functional: properties of the application
    - Reliability
    - Scalability
    - Maintainability
    - Compliance
    - Compatibility
    - Security

### Chapter 2: Data Models and Query Languages

Most applications are comprised of graduated layers of abstraction.

These layers are vital to the productivity of everyone who works across them; facilitating developer efficiency and enabling the ongoing evolution of the system. Each layer provides a contract to its users via its AP which is underpinned by a clean data model - enabling it to work in higher-level concepts and thereby hide some of the complexity of the lower levels.

Each layer of abstraction therefore comes with its own data models, and the importance of these data models cannot be understated: not only are they a necessary tool for the abstraction to work, but they also define how we think about our solutions to new problems; what is considered possible and what is not.

There are multiple kinds of data model; each with its own trade-offs.

Choosing a data model is often a decision that sits at the very foundation of our application and can therefore be expensive / difficult to change later. Given how profoundly it affects how our application behaves and performs, we at least need to understand the trade-offs inherent to several common data model paradigms:

1. Relational
2. Document
3. Graph

#### Relational Vs Document models

Edgar Codd proposed the relational data model in 1970.

A relation is what we refer to as a `table` in SQL.

Relational databases grew quickly as a result of strong demand for transactional & batch data processing through the 60s & 70s.

Alternatives did arise - network model, hierarchical model - but the relational model dominated.

**NoSQL**

NoSQL arose in the 2010s. The name is a bucket for anything that isn't relational and covers a multitude of sins (as well as some good ideas!).

There are a number of use-cases that different NoSQL databases have sought to address:

- Scalability to very large datasets
- Very high write throughput
- Preference for open-source software, over commercial database products
- Specialized query operations that don't play to the relational model's strengths
- A desire for less rigid schemas

Objects Vs Relations

With the rise of popularity in OOP langauges, the object data structures that the languages work with don't always fit neatly into the relational model - requiring 'awkward' translations between application data and the database (ie. another layer of software): ORMs.

The mismatch between the two is sometimes called an `impedence mismatch`.

In scenarios where each object that we're working with is:

- a largely self-contained blob
- heavily user-defined structure / content

Then a JSON-based document structure *might* be more suitable.

> The lack of a schema in the document model is often cited as an advantage, but this flexibility is easily abused and can end up causing untold chaos

The JSON-style representation in the document model can provide better locality: the data lives together rather than being scattered across multiple tables that need to be joined.

- This makes queries on individual documents more efficient
- BUT when storing data that is repeated across many documents, this data can easily fall out of alignment

How much of the data is shared across many instances and needs to be kept in sync across them all, and how much is relevent to only one entity?

How do we typically access the data? Can we access what we typically need without much pain, or does it grind to a halt because we're joining many tables together?

Do we know the possible values in advance, or do we rely on users to provide them? Are they static or constantly changing? 

Benefits of relations:

- remove ambiguity
- consistency
- easy to update
- localisation (translation to local language)
- search is easier

Normalisation enables us to manage a single copy of each piece of information

many-to-one relations work well for relational data, but not well for documents

document DBs often have poor support for joins. This often requires making multiple queries to the DB and then do the joins in the application code instead, which is rarely performant (we try to do as much in the DB as possible!)

It's also worth bearing in mind that applications have a tendency to drift over time. We might be able to avoid needing joins in the inital version, but over time new features tend to make the application more interconnected, and we may therefore find ourselves making more and more joins.

The difficulty of representing many-to-many relations in NoSQL is older than the current wave of document-oriented DBs.

An alternative to the relational model - the network model - was prevalent in the 70s. It was the major contender in proposing a solution to the many-to-many problem.

The network model was a generalisation of the hierarchical model (which adopted a tree-structure with single lineage) that enabled record to have multiple parents. This enabled M2M relations to be modelled.

The links between records in the network were not foreign keys, but more like pointers in a programming language (on-disk). Records were accessed by following a path from a root record, along these links; called the `access path`, this was a bit like traversing a linked list

### Chapter 3:

### Chapter 4:

## Part two: Distributed data systems

Part 1 focused on data systems that reside on a single node.

Part 2 asks how our thinking should change when we deal with data within the context of distributed systems.

Reasons to distribute data:

1. Scalability: large loads can exceed the capacity of a single node
2. Fault tolerance/High availability: redundancy protects us in the event that 1+ nodes fail
3. Latency: distributed consumption of the data can't always be satisfied (performantly) from a single node / location

### Scaling to larger loads

If our *only* consideration is the ability to handle larger loads, then there are two broad categories of approach we can consider:

1. Vertical scaling: use larger machines
1. Horizontal scaling: use more machines

#### Vertical scaling

##### Shared-memory architecture

A single instance of an operating system can run with many CPUs, RAM chips & disks and be considered a single machine. With very high-speed networks in modern clouds, these components of the 'machine' don't necessarily have to be co-located.
Vertical scaling in a *shared-memory architecture* involves adding more CPU, more RAM or more disk to a single node.

This increases the machine's capacity for work and can therefore address **scalability** issues.
Individual components (disks, memory modules, CPUs) can be hot-swapped, providing some additional fault-tolerance, but a single node configuration is by-definition in a single location (zone, region etc.) AND if the node as a whole fails, there is no backup.

*Shared-memory architecture* is expensive to scale; sourcing a machine with double the CPU, RAM & disk capacity will often cost more than double, and likely won't be able to do double the work. Node productivity (capacity / cost) therefore grows less than linearly.
[why is this, where are the economies of scale?]

##### Shared-disk architecture

The *shared-disk architecture* shares a single instance of disk storage across multiple machines (multiple independent CPUs + RAMs).
It is often used for data warehousing workloads, but scalability is limited by:
- contention\*
- locking limit overheads\*

#### Horizontal scaling

##### Shared-nothing architecture

The *shared-nothing arhitecture* is composed of arbitrarily-many nodes (machine / VM) that each have their own independent CPU, RAM & storage.
The nodes are coordinated across a network only using software.

This architecture mitigates the cost issues mentioned earlier because there is greater flexibility to configure individual nodes according to economics.
Because all coordination is done via software, over the network, it is easier to geographically distribute resources, and thereby address availability & latency concerns.

Shared-nothing architectures can be extremely powerful, but they also impose new constraints & trade-offs on the system that the engineer needs to be aware of.

### Distributing data across multiple nodes

There are two common approaches to distributing data across a multi-node deployment:

1. Replication: data is duplicated across multiple nodes, providing redundancy
1. Partitioning: data is split into smaller components and assigned to different nodes (also called ***sharding***)

These techniques are not mutually exclusive.

### The structure of Part II

Chapter 5 discusses replication
Chapter 6 discusses partitioning
Chapter 7 discusses transactions
Chapters 8 & 9 discuss the fundamental limitations of distributed systems

```{important}
**R**eplicate for **r**obustness / **r**eliability

**P**artition for **p**erformance at scale
```

### Chapter 5: Replication

```{important}
**Replication** involves storing multiple copies of the same data across multiple machines that are connected over a network.

Each node that stores a copy of the database is called a **replica**
```

What are the benefits of replication:
1. Latency reduction by storing data closer to your (distributed) users
1. Availability increases as the redundancy in the system ensures that the service can continue to run if/when components fail
1. Scalability improvements as additional machines are able to respond to requests

```{note}
Presumably, the redundancy also mitigates the risk of catastrophic data loss (similar to high availability, but different)
```

Replication is difficult to the extent that the data changes over time.
The key issue with replicated databases is that every change needs to be propagated to every replica

Considerations for replication:
- Synchronous Vs asynchronous
- How to handle failed replicas

There are 3 popular algorithms for replicating changes across nodes:
1. Single-leader
1. Multi-leader
1. Leaderless

> Almost all distributed databases uses one of these approaches.

#### Leader-based replication

Also known as **active/passive** or **master/slave** replication.

- One replica is assigned the role of **leader** / **master** / **primary**
- All write requests must go through the leader, which writes to its storage first
- All other replicas are assigned the role of **follower** / **read replica** / **slave** / **secondary**
- Each write on the **primary** is propagated to all **secondaries** via the **replication log** / **change stream**
- Each **secondary** is responsible for taking this log and applying all the changes to its own copy of the data (in the same order)

Leader-based replication is supported out-of-the-box by most RDBMS and a number of NoSQL DBs (MongoDB +).
It is also used by a number of applications that are not databases, but which rely on them. Eg. distributed message brokers such as Kafka & RabbitMQ.

##### Synchronous Vs asynchronous replication

Synchronous replication: the leader waits for followers to confirm they have applied the write before reporting a success on the original write request.

Pros:
- Guarantees consistency across replicas
- Guarantees redundancy (ie. if master fails)

Cons:
- Synchronous replication is *normally* fast (sub-second), but having to wait for slow replication can degrade system performance due to:
    - Recovering from a failure
    - Network issues
    - Operating at full capacity
- In the event of network or follower-failure, the database can be completely blocked from writing until the follower is back online

Asynchronous replication: the leader propagates the change to followers but does not wait for them to confirm that they have been successfully applied before reporting a success on the the original write request.

With synchronous replication, a single node outage can cause the whole system to fail. As a result, synchronous replication is normally too risky to use for all replicas.
When RDBMS offer synchronous replication, this normally means that a single node will replicated synchronously and any others will be asynchronous. (also called **semi-synchronous**)

##### Node failure

When a follower fails and subsequently recovers:
- Complete processing the writes already received from the leader prior to failure
- Request & process all data changes that have occured whilst the follower was offline from the leader
- Resume normal follower behaviour

When a leader fails:
- Reassign leadership to another node (AKA **failover**)

NB. failover raises the question of what to do about changes that were accepted by the leader before failure, but which were not replicated across the followers.
When the follower comes back online, there may be inconsistencies between the old & new leaders' data.
It's common for the failed leader's unreplicated changes to be discarded, but this may violate users' **durability** expectations. 

##### Replication logs

1. Statement-based: log each SQL statement executed
1. Write-ahead log (WAL) shipping: sequence of bytes containing all DB writes



**Single-leader replication**: a single node - the leader - receives **all** write requests and handles them independently. These changes are then propagated to all other nodes - the followers.
Read requests can be sent to one of multiple nodes, but without a guarantee that all nodes will always be up-to-date (consistent) with the leader.

**Multi-leader replication**: write requests can be handled by any one of several leaders. Changes are propagated between leaders and to followers.
\* how to prevent inconsistent changes across diff leaders?

**Leaderless replication**: write requests go to multiple nodes, and read requests are served by multiple nodes in parallel - responses are compared to detect & correct inconsistencies.

 ```{list-table}
:header-rows: 1
:name: replication

* - Single-leader
  - Multi-leader
  - Leaderless
* - (+) Easy to understand
  - (/) More difficult to reason about
  - (-) Most difficult to reason about
* - (+) No conflict resolution to handle, stronger consistency guarantees
  - (/) Some conflict resolution complexity, weaker consistency guarantees
  - (-) Most complex conflict resolution, weakest consistency guarantees
* - (-) Less robust to node failures & network issues
  - (/) More robust to node failures & network issues
  - (+) Most robust to node failutres & network issues
```

### Chapter 6: Partitioning

```{important}

**Partitioning** involves breaking a dataset into chunks that live in different locations in the distributed environment.

```

It is required when datasets are very large (high volume) or have very high read throughput (high velocity).

Partitioning is known under a range of other pseudonyms:

| Database      | Partitioning pseudonym    |
| :------:      | :--------------------:    |
| MongoDB       | Sharding                  |
| Elasticsearch | Sharding                  |
| SolrCloud     | Sharding                  |
| HBase         | region                    |
| Cassandra     | vnode                     |

- Each record typically belongs to exactly one partition
- The ability to place different partitions on separate nodes enables greater scalability
    - Greater scalability in storage by distributing across many disks
    - Greater scalability in query load by distributing across many processors
- Partitioning of data is relevant to both transactional and analytics processing



Replication and partitioning can be combined to leverage the advantages of both.

Choosing a style of replication is largely independent of the style of partitioning.

In order for partitioning to improve performance scalability, it's important that each node in the cluster do its fair 
share of the work: storing a fair share of the data & services a fair share of the queries. As a result, we need to think 
carefully about how we allocate data across partitions.

> **skew**: a situation where there is an uneven distribution of data or queries across nodes

> **hot-spot**: a partition with a disproportionately high query load

Random allocation of records to nodes/partitions would achieve even loads, but makes it difficult to track down each record

There are two common approaches to partitioning:
1. Key-range partitioning
1. Hash partitioning

#### Key-range partitioning

- Keys are sorted
- A range of keys are assigned to each partition

Advantages:
- Maintains key ordering and thereby enables efficient range queries

Disadvantages:
- More prone to **hot-spots** if key ranges are not queried uniformly

#### Hash partitioning

- Assign a range of hashes to a partition
- Apply a hash function to each key to determine its partition membership

Advantages:
- Distributes load evenly

Disadvantages:
- Destroys ordering of keys

#### Secondary indexes

```{important}
A secondary index does not uniquely identify a record, but offers a way to search for ocurrences of a specific value.
```

1. Local indexes
   - Writes update only one partition
    - Reads require a scatter/gather across all partitions
1. Global indexes
    - Writes may require several partitions to be updated
    - Reads can be served from a single partition

#### Routing

Changes in partitions over time introduces the problem of knowing which node to route writes / queries to over time.

Many distributed data systems use a coordination service (eg. ZooKeeper) to provide a centralised reference of which partitions are held on which nodes.

### Chapter 7: Transactions

```{important}
A transaction is a mechanism to allow an application to group a series of reads / writes together into a logical unit.
```

Either the whole transaction succeeds, or the whole transaction fails (and EVERYHING in the transaction rolls back to its original state)

```{important}
The **ACID** acronym summarises the safety guarantees provided by the use of transactions:

- **A**tomicity
- **C**onsistency
- **I**solation
- **D**urability
```

#### Atomicity

If a series of changes are included in an *atomic* transaction, then either all of them will be applied successfully or 
they will all fail to be applied - there is no possibility of partial completion.
If the transaction "fails" during the execution of the transaction, then any changes that have been carried out so far will be reversed. 

The word "atom" derives from Greek "atomos", which means irreducible / uncuttable. In the same way, an atomic transaction cannot be broken down into smaller parts that can succeed / fail independently of one another.

#### Consistency

The word "consistency" wears a lot of hats...

In the context of ACID, ***consistency*** refers to the transaction's ability to maintain the integrity of the database.

Importantly, the property of consistency is something that transactions support, rather than guarantee: it's up to the application developers to use transaction in a way that maintains consistency. 

#### Isolation



#### Durability



### Chapter 8: The trouble with distributed systems



### Chapter 9: Consistency & consensus




## Part three: Derived data


### Chapter 10: Batch processing



### Chapter 11: Stream processing



### Chapter 12: The future of data systems 