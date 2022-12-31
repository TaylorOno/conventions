# Developer Concepts

* [Acronyms](#acronyms)
  * [ACID](#acid)
  * [ATDD](#atdd)
  * [BDD](#bdd)
  * [CAP](#cap)
  * [CRUD](#crud)
  * [DSL](#dsl)
  * [DRY](#dry)
  * [FIRST](#first)
  * [KISS](#kiss)
  * [PEBKAC](#pebkac)
  * [RED](#red)
  * [SOLID](#solid)
  * [TDD](#tdd)
  * [YAGNI](#yagni)
  * [ZOMBIE](#zombie)
* [Concepts](#concepts)
  * [CAP Theorem](#cap-theorem)
  * [DORA metrics](#dora-metrics)
  * [Golden Signals](#golden-signals)


## Acronyms
### ACID
- **A**tomic
- **C**onsistent
- **I**solated
- **D**urable

**ACID** is an important concept in database systems, as it helps ensure the reliability and integrity of data by providing a set of rules for transactions to follow. 
It is often used as a standard for evaluating the performance and reliability of database systems.

### ATDD
- **A**cceptance
- **T**est
- **D**riven
- **D**evelopment

**ATDD** is a software development practice in which acceptance tests, which define the desired behavior and functionality of a system from the end user's perspective are written before the implementation of the corresponding code. 
These tests can be developed with the assistance of stakeholders to ensure that the final product meets their needs and expectations.

### BDD
- **B**ehavior
- **D**riven
- **D**evelopment

**BDD** similar to [ATDD](#atdd) emphasizes collaboration between developers, testers, and non-technical stakeholders. 
The behavior of a system is described through examples or scenarios. 
These examples are used to drive the development of the system and to validate that the system behaves as expected from the perspective of the end user

### CAP
- **C**onsistency
- **A**vailability
- **P**artition-Tolerance

**CAP** describes a concept in distributed systems that is often used to highlight the trade-offs involved in distributed systems.
Distributed system can only adhere to 2 of the 3 properties described above.

See also:
- [CAP Theorem](#cap-theorem)

### CRUD
- **C**reate
- **R**ead
- **U**pdate
- **D**elete

**CRUD** operations are the basic building blocks of many database-driven applications.
These set of operations are generally the minimum requirements of any persistent storage system.

### DSL
- **D**omain
- **S**pecific
- **L**anguage

**DSL** refers to a specialized programming language that is tailored to a particular domain or task. 
DSLs are designed to be more expressive and easier to use than general-purpose programming languages, and they often have a syntax and structure that is more closely related to the domain they are intended to support.

### DRY
- **D**ont
- **R**epeat
- **Y**ourself

**DRY** is a software development principle that suggests that developers should avoid duplicate code as a means of lowering maintenance costs and readability.
Developers should aim to reuse code as much as possible through the use of functions, classes, and other modular code constructs.

### FIRST
- **F**ast
- **I**ndependent
- **R**epeatable
- **S**elfValidating
- **T**horough

**FIRST** is a set of principles that are often used to guide the design of effective unit test cases.

### KISS
- **K**eep
- **I**t
- **S**imple
- **S**tupid

### PEBKAC
- **P**roblem
- **E**xists
- **B**etween
- **K**eyboard &
- **C**hair

PEBKAC is used to describe a user error or a problem caused by the user rather than a technical issue with the system or
software being used.

### RED
- **R**ate
- **E**rrors
- **D**uration

See also:
- [Golden Signals](#golden-signals)

### SOLID
- **S**ingle Responsibility Principle
- **O**pen/Closed Principle
- **L**iskov Substitution Principle
- **I**nterface Segregation Principle
- **D**ependency Inversion Principle

### TDD
- **T**est
- **D**riven
- **D**evelopment

### YAGNI
- **Y**ou
- **A**int
- **G**onna
- **N**eed
- **I**t

### ZOMBIE
- **Z**ero
- **O**ne
- **M**any
- **B**oundaries
- **I**nterface
- **E**xceptions

**ZOMBIE** is a set of principles that are often used to guide the design of effective tests cases.
Tests should capture the behavior of the inputs described by ZOMBIE in an effort to capture all possible behaviors.

## Concepts
### CAP Theorem
The CAP Theorem (defined by Eric Brewer) states that for a distributed data store only two out of the following three guarantees (at most) can be made:

- Consistency: when reading data, every request receives the _most recent_ data or an error is returned
- Availability: when reading data, every request receives _a non error response_, without the guarantee that it is the _most recent_ data
- Partition Tolerance: when an arbitrary number of network requests between nodes fail, the system continues to operate as expected

The core of the reasoning is as follows. It is impossible to guarantee that a network partition will not occur, Therefore in the case of a partition we can either cancel the operation (increasing consistency and decreasing availability) or proceed (increasing availability but decreasing consistency).

The name comes from the first letters of the guarantees (Consistency, Availability, Partition Tolerance). Note that it is very important to be aware that this does _not_ relate to [_ACID_](#TODO), which has a different definition of consistency. More recently, [PACELC](#TODO) theorem has been developed which adds constraints for latency and consistency when the network is _not_ partitioned (i.e. when the system is operating as expected).

Most modern database platforms acknowledge this theorem implicitly by offering the user of the database the option to choose between whether they want a highly available operation (which might include a 'dirty read') or a highly consistent operation (for example a 'quorum acknowledged write').

Real world examples:

- [Inside Google Cloud Spanner and the CAP Theorem](https://cloud.google.com/blog/products/gcp/inside-cloud-spanner-and-the-cap-theorem) - Goes into the details of how Cloud Spanner works, which appears at first to seem like a platform which has _all_ of the guarantees of CAP, but under the hood is essentially a CP system.

### DORA Metrics
DORA (**D**ev**O**ps **R**esearch and **A**ssessment) metrics are a set of metrics that are used to measure the performance of an organization's software development and delivery processes. 
These metrics were developed by Dr. Nicole Forsgren, Jez Humble, and Gene Kim as part of their research on the relationships between software development practices and organizational performance and are featured heavily in the State of Devops Report as well as the book Accelerate.

The DORA metrics are divided into four categories:
- Deployment frequency: This metric measures how often the organization deploys new code to production.
- Lead time for changes: This metric measures the elapsed time between code commit and code deployment. 
- Time to restore service: This metric measures the elapsed time between an incident occurring and service being restored. 
- Change failure rate: This metric measures the percentage of deployments that result in a failure that requires immediate remediation.

It was found that these metrics were strongly correlated to organizational performance (earnings).
The metrics are intended to provide a comprehensive view of an organization's software development and delivery process, and can be used to identify areas for improvement and to benchmark performance against industry standards.

### Golden Signals
- Latency - how long do requests take
- Traffic - how many requests are there
- Error Rate - the rate requests fail
- Resource Saturation - the amount of resources being used