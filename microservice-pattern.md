
# Don’t Build a Distributed Monolith
https://notes.jomcgi.dev/notes-on-blogs-and-podcasts/don-t-build-a-distributed-monolith/

<img width="509" height="478" alt="image" src="https://github.com/user-attachments/assets/6344ddb4-3853-4537-abd9-631396b262bf" />


1. Ball of Mud Monolith (Bad): Physically monolithic, logically monolithic. The classic spaghetti-code monolith where everything is tightly coupled. Changes in one area create bugs in another.
2. Modular Monolith (Good): Physically monolithic, logically modular. A single, deployable application with well-defined internal boundaries. This is often the best starting point for most systems.
3. True Microservices (Good, if you need it): Physically distributed, logically modular. Independent services that can be developed, deployed, and scaled separately. This is the ideal, but it comes at a high cost.
4. Distributed Monolith (The Monster 👹): Physically distributed, logically monolithic. The architecture we must avoid. It looks like microservices on the surface, but a change in one service requires changes and redeployments across many others.

#### Binary coupling
Binary coupling is a situation where two or more microservices have a direct dependency on each other, often in the form of an API contract or shared database schema.

Binary coupling can be problematic because it can lead to a tightly coupled system where changes in one microservice can have a ripple effect on other microservices. This can make it difficult to make changes to the system or introduce new features without affecting other parts of the architecture.

To avoid binary coupling, the author suggests using **event-driven architectures**, where microservices communicate with each other through asynchronous events rather than synchronous API calls. This approach decouples the services, allowing each service to evolve independently without affecting the other services. Additionally, event-driven architectures can help improve system scalability and resilience by enabling services to handle and recover from failures gracefully.

#### Consistency (Monoliths) vs. Availability (Microservices)

---

# Microservices Patterns

<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/990d0bae-239f-4533-94e2-41b7af3e2462" />

Reference:
1. Microservices Patterns, Second Edition
2. Reddit - https://www.reddit.com/r/microservices/comments/1lvbrct/microservices_patterns_2nd_edition_reflections_on/?rdt=38516

## CHAPTER 1 Escaping monolithic hell

#### Scale cube and microservices

<img width="677" height="426" alt="image" src="https://github.com/user-attachments/assets/04467abb-9dfc-420d-a52a-86548be7cd66" />


1. X-axis scaling, a.k.a. horizontal duplication; Scale by cloning.
2. Z-axis scaling, a.k.a. data partitioning; Scale by splitting similar things, such as by customer ID.
3. Y-axis scaling, a.k.a. functional decomposition; Scale by splitting things that are different, such as by function.

A service is a mini application that implements narrowly focused functionality, such as order management, customer management, and so on. A service is scaled using the X-axis
scaling, though some services may also use Z-axis scaling. For example, the Order service consists of a set of load-balanced service instances.

The high-level definition of microservice architecture (microservices) is an architectural style that functionally decomposes an application into a set of services.

#### Move fast without breaking things

Four useful metrics for assessing software development are as follows:
1. Deployment frequency—How often software is deployed into production
2. Lead time—Time from a developer checking in a change to that change being
deployed
3. Mean time to recover—Time to recover from a production problem
4. Change failure rate—Percentage of changes that result in a production problem
   
In a traditional organization, the deployment frequency is low, and the lead time is
high. Stressed-out developers and operations people typically stay up late into the
night fixing last-minute issues during the maintenance window. 

In contrast, a DevOps organization releases software frequently, often multiple times per day, with far fewer
production issues.

## CHAPTER 2 Decomposition strategies
#### What is software architecture?
> The software architecture of a computing system is the set of structures needed to reason about
the system, which comprises software elements, relations among them, and properties of both. - Documenting Software Architectures by Bass et al.


Its essence is that an application’s architecture is its decomposition into parts (the elements) and the relationships (the relations) between those parts. Decomposition is important for a couple of reasons:

1. It facilitates the division of labor and knowledge. It enables multiple people (or multiple teams) with specialized knowledge to work productively together on an application.
2. It defines how the software elements interact.

####  4+1 view model
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/e7228315-927f-4dd8-b223-d3619804de36" />
The ‘4+1’ View Model of Software Architecture” (www.cs.ubc.ca/~gregor/teaching/
papers/4+1view-architecture.pdf)

####  A layered architecture
1. Presentation layer—Contains code that implements the user interface or external APIs
2. Business logic layer—Contains the business logic
3. Persistence layer—Implements the logic of interacting with the database

Drawbacks:
1. Single presentation layer - actually, there is more than one system that will invoke
2. Single persistence layer - actually, there is more than one DB to be interacted with
3. Defines the business logic layer as depending on the persistence layer—In theory, this dependency prevents you from testing the business logic without the database.

####  HEXAGONAL ARCHITECTURE STYLE

<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/b53bbb78-dc9d-43db-a633-f7031a64631b" />

The hexagonal architecture style organizes the logical view in a way that places the business logic at the center. 

Instead of the presentation layer, the application has one or more inbound adapters that handle requests from the outside by invoking the business logic. 

Similarly, instead of a data persistence tier, the application has one or more outbound adapters that are invoked by the business logic and invoke external applications.

A key characteristic and benefit of this architecture is that the business logic doesn’t depend on the adapters. Instead, they rely upon it.

The business logic has one or more ports. A port defines a set of operations and is how the business logic interacts with what’s outside of it.

#### WHAT IS A SERVICE?
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/dc14fc0f-3a67-4ed0-aba3-eaad17534c71" />

A service is a standalone, independently deployable software component that implements some useful functionality

A service’s API encapsulates its internal implementation. Unlike in a monolith, a developer can’t write code that bypasses its API. As a result, the microservice architecture enforces the application’s modularity.

#### WHAT IS LOOSE COUPLING?
The requirement for services to be loosely coupled and to collaborate only via APIs prohibits services from communicating via a database.

You must treat a service’s persistent data like the fields of a class and keep them private. Keeping the data private enables a developer to change their service’s database schema without having to spend time coordinating with developers working on other services. Not sharing database tables also improves runtime isolation.

#### THE SIZE OF A SERVICE IS MOSTLY UNIMPORTANT

#### Decomposing an application into services 
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/5e6a78d7-4c0a-450e-afb6-5d7930f92e5b" />

Decomposing an application into services by applying the decomposition patterns: Decompose by business capability and Decompose by subdomain



1. Define the system operations: 
   1. Define a high-level domain model, define nouns
   2. Two types of system operations: Commands (CUD) and Queries (R)

   <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/01edb25d-e690-4783-8e58-ff57753e4796" />
   
2. Determine the decomposition into services.
   1. One strategy, which has its origins in the discipline of business architecture, is to define services corresponding to business capabilities. (https://microservices.io/patterns/decomposition/decompose-by-business-capability.html)

   <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/00096315-3383-4b5e-ac08-29163594f3ce" />

   
   2. Another strategy is to organize services around domain-driven design subdomains. The result is services that are organized around business concepts rather than technical concepts (https://microservices.io/patterns/decomposition/decompose-by-subdomain.html)

   <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/105790c6-1562-447b-8380-4f37c145d949" />

   - subdomains and bounded contexts (includes the code artifacts that implement the model)

Decompose microservices: Business capability vs Domain (https://stackoverflow.com/questions/45688730/decompose-microservices-business-capability-vs-domain)

#### Obstacles to decomposition

1. Network latency (batch API for fetching multiple objects/combine services, replacing expensive IPC with
language-level method or function calls)
2. Synchronous communication between services reduces availability (use asynchronous messaging)
3. Requirement to maintain data consistency across services (rarely a problem)
4. God classes, which are used throughout an application, can use concepts from domain-driven design to eliminate god classes

#### SINGLE RESPONSIBILITY PRINCIPLE
> A class should have only one reason to change. - Robert C. Martin

#### COMMON CLOSURE PRINCIPLE
> The classes in a package should be closed together against the same kinds of changes. A change that affects a package affects all the classes in that package. －Robert C. Martin

The goal is that when that business rule changes, developers only need to change code in a small number of
packages (ideally, only one). Adhering to the CCP significantly improves the maintainability of an application.


#### Bounded context concept
Using the bounded context concept from domain-driven design (DDD) to untangle data and make
decomposition easier  

