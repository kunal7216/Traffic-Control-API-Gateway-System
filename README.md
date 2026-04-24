# Traffic-Control-API-Gateway-System

## Overview

This project implements a high-throughput API Gateway integrated with a distributed rate limiting system designed to handle large-scale concurrent traffic.

The system simulates production-grade backend infrastructure responsible for:
- Controlling API traffic
- Preventing abuse (rate limiting)
- Ensuring high availability under load
- Maintaining consistency across distributed services

It is built using Spring Boot, Redis, Docker, and AWS, following microservices and distributed system design principles.

---

## Problem Statement

Modern backend systems face challenges such as:

- Uncontrolled API traffic leading to system overload
- Race conditions in distributed rate limiting
- Handling burst traffic without degrading performance
- Preventing cascading failures across services

This system solves these using:
- Distributed rate limiting
- Atomic operations using Redis Lua scripting
- Circuit breaker patterns
- Observability and monitoring

---

## High-Level Architecture

```

Client → Load Balancer → API Gateway → Rate Limiter (Redis) → Backend Services
↓
Monitoring

```

### Components:

1. **API Gateway**
   - Entry point for all requests
   - Handles routing, authentication, throttling

2. **Rate Limiter Service**
   - Implements Token Bucket / Sliding Window
   - Uses Redis for distributed state

3. **Redis**
   - Stores request counters
   - Executes atomic Lua scripts

4. **Backend Services**
   - Business logic services (mocked)

5. **Monitoring Layer**
   - Logs metrics
   - Tracks request rate, failures

---

## System Design Patterns Used

### 1. Token Bucket Algorithm
- Allows burst traffic up to bucket capacity
- Refills tokens at fixed rate

### 2. Sliding Window Algorithm
- Tracks requests within time window
- More accurate than fixed window

---

### 3. Circuit Breaker Pattern
- Prevents repeated calls to failing service
- States:
  - CLOSED → normal
  - OPEN → block requests
  - HALF-OPEN → test recovery

---

### 4. Retry with Exponential Backoff
- Retry failed requests with increasing delay

---

### 5. Distributed Locking
- Ensures consistency across nodes
- Implemented using Redis

---

## Detailed Workflow

### Request Lifecycle

1. Client sends HTTP request
2. Request hits Load Balancer
3. Routed to API Gateway instance
4. Gateway extracts:
   - API key
   - User ID
5. Rate Limiter invoked

---

### Rate Limiting Flow

1. Generate Redis key:
```

rate_limit:{user_id}:{api}

```

2. Execute Lua script:
- Increment counter
- Check limit
- Return allow/reject

3. Decision:
- Allow → forward request
- Reject → return HTTP 429

---

### Circuit Breaker Flow

1. Monitor failures
2. If threshold exceeded:
- Open circuit
3. Block further requests
4. After timeout:
- Half-open state
5. If success → close circuit

---

## API Endpoints

### 1. Health Check
```

GET /health

````

Response:
```json
{
  "status": "UP"
}
````

---

### 2. Test API (Rate Limited)

```
GET /api/resource
```

Headers:

```
X-User-Id: 123
X-API-Key: abc123
```

Response (Success):

```json
{
  "message": "Request successful"
}
```

Response (Rate Limited):

```json
{
  "error": "Too Many Requests"
}
```

---

### 3. Admin Endpoint (Update Limits)

```
POST /admin/rate-limit
```

Body:

```json
{
  "userId": "123",
  "limit": 100,
  "window": 60
}
```

---

## Redis Data Model

Keys:

```
rate_limit:{user_id}:{api}
```

Values:

* Request count
* Timestamp window

---

## Lua Script (Atomic Operation)

```lua
local current = redis.call("INCR", KEYS[1])
if current == 1 then
  redis.call("EXPIRE", KEYS[1], ARGV[1])
end
if current > tonumber(ARGV[2]) then
  return 0
else
  return 1
end
```

---

## Deployment

### Docker

```bash
docker build -t api-gateway .
docker run -p 8080:8080 api-gateway
```

---

### AWS EC2

1. Launch EC2 instance
2. Install Docker
3. Pull image
4. Run container

---

## Observability

* Logs:

  * Request count
  * Rejections
* Metrics:

  * Throughput
  * Error rate
* Monitoring tools:

  * (Optional) Prometheus / Grafana

---

## Failure Scenarios

| Scenario           | Handling          |
| ------------------ | ----------------- |
| Redis down         | fallback strategy |
| Traffic spike      | throttling        |
| Backend failure    | circuit breaker   |
| Duplicate requests | atomic Lua script |

---

## Scalability Strategy

* Stateless API → horizontal scaling
* Redis cluster for high throughput
* Load balancing across instances

---

## Future Enhancements

* Multi-region deployment
* Kubernetes orchestration
* Adaptive AI-based rate limiting
* API analytics dashboard

---

## How to Run Locally

```bash
# Start Redis
docker run -p 6379:6379 redis

# Run Spring Boot
mvn spring-boot:run
```

---

## Key Learnings

* Distributed system design
* Handling concurrency safely
* Designing resilient APIs
* Trade-offs in rate limiting strategies

```


