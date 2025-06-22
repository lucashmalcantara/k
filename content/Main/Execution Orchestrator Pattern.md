---
title: Execution Orchestrator Pattern
draft: false
tags:
  - software-development
  - software-architecture
  - cloud-computing
  - microservices-architecture
---
 
## What problem does this pattern solve?

This pattern is used in many scenarios, but primarily in [[Microservices Architecture]]. The Execution Orchestrator Pattern is an extension of the [[Scatter Gather Pattern]] that allows us to communicate with different services (each with its own responsibility) to control the request flow and aggregate the results into a single response. Ideally, the Execution Orchestrator Pattern doesn't do any business logic. If we add too much business logic to the orchestrator, it can become a big monolithic application that simply talks to microservices. The orchestrator also handles exception and retries and maintain the state of the flow until it gets the final result. The orchestration services isn't a dumb API Gateway, it fully understands the context of the request.

![[Pasted image 20250622171908.png]]_Execution Orchestrator pattern scheme[[#1] _
## Pros

- Allows us to scale our architecture easily by adding more services.
- The contract between the client and the orchestrator doesn't need to change if we make changes to the API or the architecture.
- It is useful for tracing and debugging issues in the flow because we can use the orchestrator logs as a reference, since it centralizes the communication flow between services.
## Cons

- Because the business logic is spread across multiple services, the orchestrator must handle different communications and aggregate the results.
- Since the orchestrator may crash during the process, we need to introduce additional complexity to ensure fault tolerance.
## References

### 1

Pogrebinsky. “Architect Large Scale Systems using Cloud Computing, Software Architecture Patterns & Modern System Design Principles.” github.com. Accessed: Jun. 22, 2025. [Online.] Available: https://www.udemy.com/course/the-complete-cloud-computing-software-architecture-patterns/learn/lecture/33528000#overview