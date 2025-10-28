
# Don’t Build a Distributed Monolith
https://notes.jomcgi.dev/notes-on-blogs-and-podcasts/don-t-build-a-distributed-monolith/

#### Distributed Monolith ！！BAD！！
An architecture that combines all the network latency, failure modes, and operational complexity of a distributed system with the tight coupling and deployment bottlenecks of a monolith. It is the worst of both worlds.

#### Binary coupling
Binary coupling is a situation where two or more microservices have a direct dependency on each other, often in the form of an API contract or shared database schema.

Binary coupling can be problematic because it can lead to a tightly coupled system where changes in one microservice can have a ripple effect on other microservices. This can make it difficult to make changes to the system or introduce new features without affecting other parts of the architecture.

To avoid binary coupling, the author suggests using **event-driven architectures**, where microservices communicate with each other through asynchronous events rather than synchronous API calls. This approach decouples the services, allowing each service to evolve independently without affecting the other services. Additionally, event-driven architectures can help improve system scalability and resilience by enabling services to handle and recover from failures gracefully.

# Microservices Patterns

<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/990d0bae-239f-4533-94e2-41b7af3e2462" />

Reference:
1. Microservices Patterns, Second Edition
2. Reddit - https://www.reddit.com/r/microservices/comments/1lvbrct/microservices_patterns_2nd_edition_reflections_on/?rdt=38516

#### Scale cube and microservices

<img width="677" height="426" alt="image" src="https://github.com/user-attachments/assets/04467abb-9dfc-420d-a52a-86548be7cd66" />


1. X-axis scaling, a.k.a. horizontal duplication; Scale by cloning.
2. Z-axis scaling, a.k.a. data partitioning; Scale by splitting similar things, such as by customer ID.
3. Y-axis scaling, a.k.a. functional decomposition; Scale by splitting things that are different, such as by function.

A service is a mini application that implements narrowly focused functionality, such as order management, customer management, and so on. A service is scaled using the X-axis
scaling, though some services may also use Z-axis scaling. For example, the Order service consists of a set of load-balanced service instances.

The high-level definition of microservice architecture (microservices) is an architectural style that functionally decomposes an application into a set of services.


