# Types of Server (Backend) Architecture

A quick guide to the main ways backend applications are structured.

## 1. Monolithic

The whole application (UI, business logic, data access) is built and deployed as a single unit.

- **Pros:** Simple to develop, test, and deploy; easy debugging
- **Cons:** Harder to scale parts independently; large codebases get difficult to change
- **Best for:** Small teams, MVPs, early-stage products

## 2. Microservices

The application is split into small, independent services that communicate over APIs.

- **Pros:** Independent scaling and deployment; teams can work in parallel; technology flexibility
- **Cons:** Operational complexity; network latency; distributed debugging
- **Best for:** Large systems and large teams

## 3. Modular Monolith

A single deployable unit, internally organized into well-separated modules.

- **Pros:** Simplicity of a monolith with cleaner boundaries; easy path to microservices later
- **Cons:** Requires discipline to keep module boundaries clean
- **Best for:** Growing products that aren't ready for microservices

## 4. Serverless (FaaS)

Code runs as on-demand functions (AWS Lambda, Google Cloud Functions, Azure Functions). The provider manages the servers.

- **Pros:** No server management; automatic scaling; pay per execution
- **Cons:** Cold starts; vendor lock-in; execution limits
- **Best for:** Spiky workloads, event handlers, lightweight APIs

## 5. Service-Oriented Architecture (SOA)

Larger, coarser-grained services share an enterprise service bus.

- **Pros:** Reusable services across the enterprise
- **Cons:** Heavy middleware; centralized bus can become a bottleneck
- **Best for:** Legacy and enterprise integration

## 6. Event-Driven

Components communicate through events or message queues (Kafka, RabbitMQ).

- **Pros:** Loose coupling; good scalability and resilience
- **Cons:** Harder to trace flows; eventual consistency
- **Best for:** Real-time processing, async workflows

## Comparison

| Architecture | Complexity | Scalability | Deployment |
|---|---|---|---|
| Monolithic | Low | Limited | Single unit |
| Modular monolith | Low–Medium | Moderate | Single unit |
| Microservices | High | High | Per service |
| Serverless | Medium | Very high | Per function |
| SOA | High | Moderate | Per service via bus |
| Event-driven | Medium–High | High | Per component |

## Summary

The most common choice is **monolith vs. microservices**. Start with a monolith or modular monolith, and move to microservices or serverless when scale or team size demands it.