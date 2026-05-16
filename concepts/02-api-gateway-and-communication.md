# API Gateway and Service Communication Patterns

## What is an API Gateway?
An **API Gateway** is the single entry point for all client requests to your microservices backend. Instead of clients knowing about and calling individual services directly, they call the gateway, which routes requests to the appropriate service.

```
Client (Browser, Mobile, 3rd Party)
        │
        ▼
┌───────────────────────┐
│     API Gateway       │  Auth, Rate Limiting, Routing, SSL termination
└───────────────────────┘
   │         │        │
   ▼         ▼        ▼
[Users]   [Orders]  [Products]   ← Internal microservices
```

### Responsibilities of an API Gateway
- **Routing:** Forward requests to the right service based on path (`/users` → User Service).
- **Authentication:** Validate JWT tokens or API keys before routing.
- **Rate Limiting:** Prevent abuse (e.g., max 100 requests/min per IP).
- **SSL Termination:** Handle HTTPS at the gateway; services communicate over plain HTTP internally.
- **Request/Response Transformation:** Modify headers, transform JSON shapes.
- **Load Balancing:** Distribute traffic across multiple instances of a service.

Popular API Gateways: **AWS API Gateway**, **Kong**, **NGINX**, **Traefik**, **Envoy**.

---

## Service Communication: Sync vs Async

### Synchronous (Request/Response)
Services call each other via **HTTP/REST** or **gRPC** and wait for a response.
- **Pros:** Simple, immediate feedback.
- **Cons:** Tight temporal coupling — if the downstream service is slow or down, the caller blocks or fails.

```
Order Service ──HTTP POST /payments──► Payment Service
Order Service ◄──200 OK, paymentId──── Payment Service
```

### Asynchronous (Event-Driven)
Services communicate via a **message broker** (Kafka, RabbitMQ, AWS SQS). The producer publishes an event and moves on. Consumers process it independently.

```
Order Service ──publishes "order.placed" event──► Kafka Topic
                                                       │
                      ┌────────────────────────────────┤
                      ▼                                ▼
              Payment Service               Notification Service
              (processes payment)           (sends email)
```
- **Pros:** Loose coupling, high resilience, natural backpressure.
- **Cons:** Eventually consistent, harder to trace and debug.

---

## Circuit Breaker Pattern
Without circuit breakers, a slow downstream service causes cascading failures — all callers block waiting, exhausting threads. The **Circuit Breaker** (Hystrix, Resilience4j) monitors failed calls and "opens the circuit" after a threshold, returning a fallback immediately instead of waiting.

```
States: CLOSED → OPEN → HALF-OPEN
CLOSED: Normal operation, calls pass through.
OPEN: Failure threshold hit, calls fail immediately (no waiting).
HALF-OPEN: After timeout, let one call through to test recovery.
```

---

## Hands-on Lab: Model a Microservices Communication Flow

### Scenario
Design the communication flow for a food delivery app: **Customer places an order**.

### Exercise
Create a file `communication-flow.md` and document:

1. **Synchronous calls:**
   - Customer → API Gateway → Order Service (create order)
   - Order Service → Inventory Service (check stock - sync, must confirm before proceeding)

2. **Async events after order is confirmed:**
   - Order Service → publishes `order.confirmed` event to Kafka
   - Payment Service subscribes → processes payment → publishes `payment.completed`
   - Notification Service subscribes to `order.confirmed` → sends "Order received" email
   - Delivery Service subscribes to `payment.completed` → assigns a driver

3. **Answer these design questions:**
   - Why is Inventory check synchronous but notification async?
   - What happens if Payment Service is down when an event arrives?
   - How does the customer know the order status? (hint: polling, WebSockets, or SSE)

### Deliverable
An ASCII diagram of the full flow with labels for sync (→) and async (⇢) calls.
