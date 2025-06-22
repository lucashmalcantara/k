---
title: Scatter Gather Pattern
draft: false
tags:
  - software-development
  - software-architecture
  - cloud-computing
---
## What problem does this pattern solve?

In the Scatter Gather pattern, the client sends a request to the dispatcher, which routes the request to all workers. Later, the dispatcher gathers the responses from all the workers, aggregates them into a single response, and sends it back to the client. Each request to a worker is independent from each other. In other words, this patterns is a great choice for high scalability because requests run parallel. The dispatcher should be decoupled from the workers, so it can't be affected when a worker is unavailable.

![[Pasted image 20250622164740.png]]_Scatter Gather pattern scheme [[#1]]_

![[Pasted image 20250622165114.png]]_Scatter Gather - Search services response [[#1]]_

![[Pasted image 20250622165350.png]]_Scatter Gather - Hospitality services [[#1]]_

![[Pasted image 20250622170426.png]]_Scatter Gather with separate dispatcher and aggregator [[#1]]_
## Pros

- Perform a large number of requests in parallel.
- Provide a large amount of information in a constant time.
- It's a great choice for high scalability.
## Cons

- Workers can become unreachable/unavailable at any moment. 
## References

### 1

Pogrebinsky. “Architect Large Scale Systems using Cloud Computing, Software Architecture Patterns & Modern System Design Principles.” github.com. Accessed: Jun. 22, 2025. [Online.] Available: https://www.udemy.com/course/the-complete-cloud-computing-software-architecture-patterns/learn/lecture/33360300#overview