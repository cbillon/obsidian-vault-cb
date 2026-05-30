---
link: https://www.dbos.dev/blog/new-three-tier-application
excerpt: How to manage the complexity of distributed backend architectures - exploring different ways of orchestrating services and ensuring durable execution
slurped: 2026-05-29T19:40
title: The New Three-Tier Application | DBOS
tags:
  - architecture
---

In the beginning (that is, the 90’s), developers created the three-tier application. Per [Martin Fowler](https://martinfowler.com/books/eaa.html), these tiers were the _data source tier_, managing persistent data, the _domain tier_, implementing the application’s primary business logic, and the _presentation tier_, handling the interaction between the user and the software. The motivation for this separation is as relevant today as it was then: to improve modularity and allow different components of the system to be developed relatively independently.

![original 3-tier application architecture diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e465625c01d90c61930f_AD_4nXdyLMaHtGd9sld39sm9zaoLWEE_0j1qTdPNM6qKwSkRzRUMv09TWxZvE5HdxEoN0np5RMoyGpdsyeCcyJxZNb1H2WQN0Rso8xlRUZsjOnyZxAzXqwtBGuvLPKJBOrjywsdJMbPkNA.png)

Of course, application architecture has evolved greatly since the 90's. The first big change was in the presentation tier. While most applications once used native clients or the terminal as their interface, they’ve now mostly moved to a web interface. Thus, the presentation tier became the frontend and the domain tier became the backend:

![monolithic cloud backend architecture diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e465dcbabf9fbbe439cb_AD_4nXcxCeTRfNiS3VhsPHvU-1THiGurej0xwUJPKROtIN2rMnhckUX8c-gwVGkOrfu0nbrtdeGB2de7annWVgKtoeFUcLGY5y6raMs-Sd1usNuhh1Z8Hqwt_wy_-YCE51qTowss5fpL.png)

Over the last ~15 years, an even larger shift has occurred in the domain tier/backend. These used to be largely monolithic, implemented in a single software artifact on a single server. However, as both the computational complexity (increasing data volumes and processing demands) and organizational complexity (larger engineering organizations, specialized domain knowledge, need for parallel development) of applications increased, developers began distributing them into many loosely-coupled microservices and even serverless functions. Nowadays, a single application’s backend can consist of many interoperating services:

![Microservices backend architecture diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e465a7add21a6b10653d_AD_4nXfGIfOT5_NtHuvIgBFqWMZ5HU_ftoR0d8c-f8ip2-pgyHH7HT11fgH5C0fAa24_P8_BrPIfyZQC2IaHZWsPPLBN4tu4ZolpqrWHKti_QbAmSicigDu38nITn8w2CNbaoYDjrIS2_g.png)

This complexity has created a new problem for application developers: how to coordinate operations in a distributed backend? For example:

- How to **atomically** perform a set of operations in multiple services, so that all happen or none do?
- How to request a remote service execute a task **exactly once**?
- How to execute a task **asynchronously**?

These are difficult challenges to solve in any setting, but are especially hard for a distributed backend because of the possibility of transient failures in any service at any time. Even monolithic backends now face similar challenges, as they increasingly depend on numerous third-party services (e.g. OpenAI for AI capabilities, Stripe for billing, Twilio for messaging, Auth0 for authentication) and must carefully coordinate interactions with them.

To solve these problems, developers have introduced a new application tier: an _orchestration tier_ that coordinates operations across distributed microservices and presents a simple API to the frontend.

![microservices backend architecture with orchestration layer](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e46577639a037303d8ed_AD_4nXcKm7oASH78oiwWjNheb_stQnrWCPGshMJSt5C2HIHbsUrft3ARSYXga0zRJMrK0quJtWPXxpBJg9ceI44LqzNtQ-K6ZhmsK_GsZcNEU9JtlEnEUFCVnXsOZq8CU0F0tW6YLPhp.png)

This orchestration tier is primarily responsible for **guaranteeing code executes correctly despite failures**. For example, an orchestration tier might:

- Guarantee a set of operations are executed atomically by following a saga pattern, retrying transient failures and “backing out” by undoing earlier operations if later operations fail unrecoverably.
- Execute a task exactly-once by submitting it with an idempotency key and retrying in case of transient failure.
- Safely execute an asynchronous task by monitoring its execution and restarting it if it is interrupted.

## **How to Build a Workflow Orchestration Tier**

At this point, developers have been building orchestration tiers for more than a decade. Broadly, there are two classes of orchestration tier. Each has advantages and disadvantages, and most large enterprises use both for different applications.

### **Option 1: Do-It-Yourself**

The first class is “Do-It-Yourself” orchestration. Here, developers implement orchestration themselves, often leveraging an event processing system or message broker like Apache Kafka, AWS SQS, or RabbitMQ. For example, for service A to schedule a task in service B, service A would write the task to Kafka, then service B would read the message from Kafka and execute the task. Doing this correctly is hard and requires deep knowledge of the semantics of the underlying system. In this example, service B would have to correctly handle duplicate messages (since Kafka delivers at-least-once) and would have to manage timeouts while processing its task.

![DIY microservices orchestration architecture diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e465ca5be9b253b9e76f_AD_4nXeXh1YO25oG-qGLFmg00nLVR54kN5GYMpKDsnL32H5yRleNFsiPu_hYAt-TkTWp4dVWwkKfDnZyKziewMF3lqD92bBA_5ieGR45ANPyBNVN838EB5-cMA4DFTou5XZN4K2Q_b-D.png)

### **Option 2: Dedicated External Orchestrator (e.g., Temporal)**

The second class of orchestration tier are dedicated orchestration systems, which started to emerge in the last few years in response to the complexity of DIY solutions. Most of these use a _workflow_ abstraction, where developers write programs as workflows of tasks. The system **durably executes** the workflow, retrying individual steps until they succeed and keeping track of the workflow’s progress in a persistent store. Some popular orchestration systems include [AWS Step Functions](https://aws.amazon.com/step-functions/), for AWS operations (especially AWS Lambda functions), [Apache Airflow](https://airflow.apache.org/), for data engineering pipelines, and [Temporal](https://temporal.io/), for asynchronous backends.

![heavyweight microservices  architecture orchestration diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e465a7add21a6b10654a_AD_4nXcAJIxYKrXrwTLTFV_WXY_em4RNp0J1859hNtfpipMMhj_iB9riWdAbZY3nws1kdQzaj2BzvTc92JOqKP1-UQx0emsWi198Vcjv1VMjDLWckNxdOFDrCJUOcCo0nRIGhEiJUioYPw.png)

Right now, an orchestration tier seems necessary to manage the complexity of distributed systems. However, neither class is completely satisfactory. DIY solutions are complex and hard to maintain. Orchestration systems are easier to use, but require outsourcing your application’s control flow to an external system, with all the architectural complexity that entails. Additionally, both DIY solutions and orchestration systems typically come with significant performance overhead because of the need for many rounds of communication between the orchestration and backend tiers and because most orchestration solutions are highly asynchronous.

## **What Comes Next: Embedded Workflow Orchestration**

You can’t put the genie back in the bottle. At the technical and organizational scale of modern enterprises, the complexity of orchestrating distributed systems is unavoidable. However, we need **better ways of managing that complexity**.

The biggest source of complexity comes from the **separation** of the orchestration and application tiers. Running an application’s control flow on a separate system from its business logic adds friction to every step of developing, testing, and debugging an application. It essentially turns individual applications into distributed microservices, with all the complexity that implies.

To manage this complexity, we believe that any good solution to the orchestration problem should **combine** the orchestration and application tiers. At DBOS, we’re throwing our hat in the ring by building DBOS Transact (Python, TypeScript, Go, Java, Kotlin): a **lightweight orchestration library** you can add to any program. Like in existing orchestration systems, you write programs as workflows of steps. For example, here’s a simplified program for performing checkout in an e-commerce service:

![DBOS durable execution library python code example](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e465c553481511e22b32_AD_4nXfIRPALnzliu5_w33eI9bnCw1uyir0CdkeQo_QL3eGXwmMHummMxssh7018EU7YcgYrIsRJa9AamZgJOJYynhADe3CulIhtGPWB7CfQGR--wITuws3iLP0NotGlqpLAKinbzgsaUw.png)

The library (specifically, the `DBOS.workflow()` and `DBOS.step()` decorators) wraps your functions with code that orchestrates them. It persists your program’s execution state–which workflows are currently executing and which steps they’ve completed–in a Postgres database.

![DBOS durable execution state machine diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e465a15e080c1762eefa_AD_4nXdekIIfwINA1AK2jjEfRhagFhh4HGcWTn07KNRsR3on2oO8WWhaSqyblCPYhr8N1YToAHUg5sP7G_1hhJVbR6myePM2a9N8UUBP4kxz5DzipJv55B7bFf5qFFJHIr83uOL51BJFEw.png)

By persisting execution state to a database, a lightweight library can fulfill the primary goal of an orchestration system: **guaranteeing code executes correctly despite failures.** If a program fails, the library can look up its state in Postgres to figure out what step to take next, retrying transient issues and recovering interrupted executions from their last completed step.

To make this more concrete: imagine the order shipping service experiences an outage. The workflow doesn't cancel paid orders (which would frustrate customers). Instead, it retries with exponential backoff—potentially for hours—until the shipping service recovers. If the checkout service itself crashes or restarts while workflows are waiting, orders aren't lost. The service simply looks up each workflow's state in Postgres and resumes each from where it left off, ensuring customer orders are processed correctly even through multiple system failures.

Implementing orchestration in a library connected to a database means you can eliminate the orchestration tier, pushing its functionality into the application tier (the library instruments your program) and the database tier (your workflow state is persisted to Postgres). This manages the complexity of a distributed world, bringing the complexity of a microservice RPC call or third-party API call closer to that of a regular function call. So once again, applications will have three tiers:

![Efficient microservices backend orchestration architecture diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/67c7e4650728c36da934fbd2_AD_4nXdGqo0XY30-gEv6ufqGtuEZ0Hg9WweEQIWvgPhnwTjEAOdcPOTeteKUmwjPzaUk8dazce9QRRH6hfE1xwWF2xIyD7c-Stm71GWELWIOsU2rNCmfoBbCZ9_aa4YD2-JStMGBCz2H3g.png)

‍