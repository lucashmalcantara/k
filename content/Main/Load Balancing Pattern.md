---
title: Load Balancing Pattern
draft: false
tags:
  - cloud-computing
  - software-development
  - software-architecture
  - scalability-patterns
---
## What problem does this pattern solve?


**This pattern is used in scenarios where a single server cannot handle a high volume of requests and helps us improve application scalability and resilience.** When incoming traffic exceeds the CPU, memory, or network capacity of a single server, it will either crash or experience significant performance degradation. Upgrading the server hardware may not solve the problem — it will only delay it. Therefore, the solution is to distribute the incoming requests among multiple application instances. These instances typically run as identical copies of the same application on separate physical or virtual machines.


![[Pasted image 20250619190603.png]]_Load balancing structure [[#1]]_

![[Pasted image 20250619190435.png]]_Load balancing in microservices architecture [[#1]]_

## Implementation techniques

### Cloud Load Balancing

A cloud load balancing is a implementation provided by cloud providers such as AWS (Amazon Web Services), Azure, GCP (Google Cloud Platform). It has a proprietary implementation, the implementation details are hidden from us.

![[Pasted image 20250619190935.png]]_Cloud Load Balancing [[#1]]_

### Load Balancing Pattern with Message Broker


Although **the main purpose of a message broker is not load balancing**, it can function as one. Publishers can publish messages to a queue that will be consumed by multiple consumers, thereby distributing the processing load among them.

![[Pasted image 20250619191231.png]]_Load Balancing Pattern with Message Broker [[#1]]_

## Routing Algorithms

### Round Robin

The Round Robin algorithm consists in redirecting each incoming request sequentially to the next server. It's usually the default load balancing algorithm implementation. However, this algorithm works well only under the assumption that our application is stateless. In other words, it’ll work only if each request can be handled by any application server.

![[Pasted image 20250619192343.png]]_Round Robin algorithm [[#1]]_

So, what's the problem with round robin algorithm in this case? It doesn't work when we need to maintain an active session between the client and the server. Let's see some examples.

In an authentication scenario, the client must communicate with the server that holds the session credentials. If the client is redirected to a different server, they must authenticate again. Similarly, when uploading a large file, all parts must be handled by the same server to properly assemble the final file.

![[Pasted image 20250619194029.png]]_Failed use of the round robin load balancing algorithm [[#1] _

### Sicky Session / Session Affinity

Using this algorithm the load balancer tries its best to send traffic from a given client to the same server as long as that server is healthy. The load balancer can use either the cookie sent with each request or the client’s IP address.

### Least Connections

Using this algorithm, the load balancer will route the new requests to servers that hold the least number of already established connections.

![[Pasted image 20250619194905.png]]_Least connections load balancing algorithm [[#1]]_


## References

### 1

Pogrebinsky. “Architect Large Scale Systems using Cloud Computing, Software Architecture Patterns & Modern System Design Principles.” github.com. Accessed: Jun. 19, 2025. [Online.] Available: https://www.udemy.com/course/the-complete-cloud-computing-software-architecture-patterns/learn/lecture/33360024?start=0#overview
