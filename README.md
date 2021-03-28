# (Highly) Opinionated (Microservice) Design Guide

Following are the ideas I have gathered while working with microservices at Wise (former Transferwise).

## Ideas on the macro design

### Microservice size
There are 2 factors to consider when deciding how big/small should a service be: technical scalability and organisational scalability. 
If neither of these is pushing for a split it is better to keep going in a monolith. 
There is no reason to assume that if we are unable to design things well in a single codebase we will be able to design things well in a distributed codebase. 
[Read more](https://medium.com/transferwise-engineering/what-is-a-good-size-for-a-microservice-d8d369d827ae)

### Cross-service communication
If you indeed need to split things into microservices be careful [not to replicate the same antipatterns you might have in the current (legacy) codebase](https://medium.com/@urgo/how-to-avoid-entity-services-58bacbe3ee0b). 
Otherwise, you may just replace the [monolith with a distributed monolith](https://medium.com/transferwise-engineering/from-monolith-to-distributed-monolith-fd53d8dbbeba).

Having as few synchronous calls between services as possible is always something to strive for. 
Using Kafka to build local replicas of data is one solution. 
Often Kafka is an obvious solution where we need complex data transformation or aggregation of multiple sources.
However, it is also very handy for avoiding single source [sync data fetching as well](https://medium.com/@urgo/how-to-avoid-service-to-service-calls-for-plain-data-8ebe5fcd7879).  

## Ideas on the micro design

### How to name things
Avoid technical suffixes like Service, Gateway, Repository and instead search for the domain specific names. 
[Names are very powerful](https://medium.com/@urgo/the-effect-of-naming-on-domain-model-design-4654a759067) - just like they can work as central cores for bringing more order into things they can also make things messier and harder to understand.

### Fool proof design
Try to design your domain model so that it is [impossible to use it incorrectly](https://medium.com/transferwise-engineering/poka-yoke-in-software-design-e6a0d955a4d8).

One such example is [using separate classes for different entity states](https://medium.com/transferwise-engineering/implementing-entity-states-as-separate-classes-abc3c745fa2)

### Plain JDBC as opposed to ORMs
There are many situations [when using Hibernate is not a good idea](https://medium.com/transferwise-engineering/when-not-to-use-hibernate-84fec5091fd1). 
For very simple domain models that clearly map to DB tables ORM may be OK. 
However, when domain model grows in complexity there will be
[more and more constraints from using ORM](https://medium.com/transferwise-engineering/hibernate-and-domain-model-design-602739ab1b15).

There are some more lightweight tools for accessing relational DBs like [jOOQ](https://www.jooq.org/) or [myBatis](https://mybatis.org/mybatis-3/).
Don't have any first-hand experience in using these, but I recommend thinking carefully if the benefits of these libraries are worth the potential constraints they may be imposing for more complex use cases. 

An idea how to manage complex data mapping to domain model using [specialised factories](https://medium.com/transferwise-engineering/java-complex-object-mapping-with-jdbc-a19169ba617)

### Avoid unnecessary horizontal layers
Often there is separate Service class for each Entity which then encapsulates all access to that Entity. 
If there is no additional logic in that Service then it should not exist. 
Relaxed layering FTW!

### Javaconfig based DI as opposed to annotations
The benefits are written [here](http://tech.transferwise.com/building-modular-apps-using-spring/).

### Hardcoded config as opposed to externalized config
Whenever possible prefer using config as is inside code as opposed to external. Read [this post for more details](https://tech.transferwise.com/application-configuration/)

### Null object
Always avoid any nulls.
Instead [use NullObject and Optional](https://medium.com/transferwise-engineering/null-object-and-optional-48fc4e74c04c).
Avoid NotNull annotations and instead [use explicit Option types](https://medium.com/the-innovation/implementing-nullability-in-object-state-1c501231810b).


## Testing

### Categorisation

Try to come up with clear categories how your team classifies tests. 
Otherwise, you may end up with all sorts of weird creations that do some "unit test" things and some "integration test" things all at once.
This means more work when deciding what tests are needed for any new feature but also hard time understanding existing tests.

### Unit tests
TDD is good, but it is also very hard. 
Even when failing to do TDD by the book I have found it valuable to [listen to what the tests are trying to tell me](https://medium.com/transferwise-engineering/5-tips-for-getting-more-out-of-our-tests-5b432ee2ea47).
If you do that then unit tests can be [great source for pushing our design](https://medium.com/transferwise-engineering/poka-yoke-in-software-design-e6a0d955a4d8).

If better design by tests is not something you want then it might be better to focus on writing more course grained tests. 
In case of Hexagonal architecture [port level tests](https://medium.com/@urgo/more-effective-testing-of-spring-microservices-1ed87d456ae7) are good for driving high level architecture. 

In any case I recommend [not writing tests against configuration](https://medium.com/transferwise-engineering/dont-write-tests-against-configuration-574d2121469c).

### Acceptance tests
In [Growing OO Software Guided by Tests](http://www.growing-object-oriented-software.com/) Steve and Nat suggest the idea of starting with a high level acceptance test before getting into new feature implementation. 
One easy way of knowing when you have enough acceptance tests is when you don't feel like you should do some manual testing before releasing.

