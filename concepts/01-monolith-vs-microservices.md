# Monolithic vs Microservices Architecture

## The Monolithic Architecture

### What is a Monolith?
A **monolithic application** is built as a single, unified unit. All components — UI layer, business logic, database access layer — are tightly coupled and deployed together as one artifact (one JAR, one binary, one deployment).

```
┌──────────────────────────────────────────────────────────┐
│                    Monolithic App                        │
│  ┌──────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  UI/API  │  │ Business     │  │  Data Access     │   │
│  │  Layer   │──│ Logic Layer  │──│  Layer (DB ORM)  │   │
│  └──────────┘  └──────────────┘  └──────────────────┘   │
│                                            │              │
│                                     ┌──────────┐         │
│                                     │ Database │         │
│                                     └──────────┘         │
└──────────────────────────────────────────────────────────┘
```

### Advantages of Monoliths
- **Simple to develop:** One repo, one codebase, one deployment. Great for small teams.
- **Easy testing:** All code in one place — integration tests are straightforward.
- **No network overhead:** Function calls instead of network calls between components.
- **Simple deployment:** Deploy one artifact.

### Disadvantages of Monoliths
- **Scaling is all-or-nothing:** You must scale the entire app even if only one component (e.g., search) is under load.
- **Slow deployments:** A small change requires redeploying the entire monolith.
- **Technology lock-in:** One language/framework for everything.
- **Team bottlenecks:** Large teams working in the same codebase causes merge conflicts and coordination overhead.
- **Reliability:** One bug in one module can crash the entire application.

---

## The Microservices Architecture

### What is Microservices?
A **microservices architecture** structures an application as a collection of small, independently deployable services that communicate over network APIs (HTTP/REST or messaging queues).

```
          ┌─────────────┐
          │ API Gateway │  ← Single entry point for clients
          └──────┬──────┘
     ┌───────────┼───────────┐
     ▼           ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│  User   │ │  Order  │ │ Payment │
│ Service │ │ Service │ │ Service │
│ (DB)    │ │ (DB)    │ │ (DB)    │
└─────────┘ └─────────┘ └─────────┘
```

Each service:
- Owns its own database (**Database per Service** pattern).
- Can be deployed, scaled, and updated independently.
- Communicates via REST APIs, gRPC, or message brokers (Kafka, RabbitMQ).
- Can be written in a different language/framework.

### Advantages of Microservices
- **Independent scaling:** Scale only the services under load.
- **Independent deployment:** Deploy User Service without touching Payment Service.
- **Technology flexibility:** Different services can use different languages/DBs.
- **Fault isolation:** Failure in one service doesn't necessarily crash others (with proper circuit breakers).
- **Team autonomy:** Teams own their service end-to-end.

### Disadvantages of Microservices
- **Operational complexity:** Dozens of services to deploy, monitor, and debug.
- **Network latency:** Service-to-service calls add latency compared to local function calls.
- **Distributed system problems:** Data consistency, partial failures, distributed tracing.
- **Testing is harder:** Integration testing across services is complex.
- **Overhead for small teams:** The "microservices tax" isn't worth it until scale demands it.

---

## When to Choose Which

| Signal | Go Monolith | Go Microservices |
|--------|-------------|-----------------|
| Team size | < 10 engineers | > 10, multiple teams |
| Stage | Early startup (MVP) | Scale-up, established product |
| Scale needs | Uniform | Some components need 10× scale |
| Deploy frequency | Weekly | Multiple times per day per service |

**The "Strangler Fig" Pattern:** Start with a monolith and gradually extract services as bottlenecks emerge — the pragmatic path most successful companies (Netflix, Amazon, Uber) took.

---

## Hands-on Lab: Design a Microservices Decomposition

### Scenario
You have a monolithic e-commerce app. Decompose it into microservices.

### Exercise
On paper or in a markdown file, answer these questions:
1. **Identify business domains:** List the key capabilities (User management, Product catalog, Order processing, Payment, Notifications, Shipping).
2. **Define service boundaries:** For each domain, define:
   - What data it owns (its database).
   - Its public API (what endpoints it exposes).
   - Its dependencies (which other services it calls).
3. **Identify communication patterns:**
   - Synchronous (REST/gRPC): User Service → Auth check.
   - Asynchronous (Kafka/RabbitMQ): Order placed → Payment Service, Notification Service receive events.
4. **Design the API Gateway:** What does the gateway expose to the outside world? What does it route internally?
5. **Plan for failure:** If Payment Service is down, what happens? Implement a circuit breaker concept.

### Deliverable
Create `decomposition.md` documenting your design with an ASCII architecture diagram similar to the one above.
