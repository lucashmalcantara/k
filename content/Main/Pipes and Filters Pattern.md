---
title: Pipes and Filters Pattern
draft: false
tags:
  - software-development
  - software-architecture
  - cloud-computing
---
## What problem does this pattern solve?
 
The Pipes and Filters pattern provides the ability to process data using multiple stages (filters) that can run sequentially or in parallel. The connection that passes data between these stages is called a _pipe_. In this pattern, the entry point is called the _data source_ — the origin of the incoming data. The final destination is called the _data sink_.


![[Pasted image 20250622134759.png]]_Pipes and Filters schema [[#1]]_

Being part of a single application, the monolithic approach is limited because we can't easily scale stages independently, nor can we implement those stages in different technologies, such as different programming languages.

![[Pasted image 20250622135309.png]]_Pipes and filters - Monolithic approach [[#1]]_

## Pros

- Freedom to choose technologies/Programming languages for each operation.
- Running each operation on optimized hardware.
- Ability to scale each component as needed.
## Cons

- High complexity and additional overhead.
- Each filters needs to be stateless, and should be provided with enough information as part of its input.
- Not a good fit for transactions (where atomicity and data consistency are important).

## References

### 1

Pogrebinsky. “Architect Large Scale Systems using Cloud Computing, Software Architecture Patterns & Modern System Design Principles.” github.com. Accessed: Jun. 19, 2025. [Online.] Available: https://www.udemy.com/course/the-complete-cloud-computing-software-architecture-patterns/learn/lecture/33360024?start=0#overview
