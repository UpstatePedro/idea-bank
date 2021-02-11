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

1. Scalability
2. Fault tolerance/High availability
3. Latency