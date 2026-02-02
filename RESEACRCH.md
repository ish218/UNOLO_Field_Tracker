# Real-Time Location Tracking – Architecture Recommendation

## Overview

For this research task, I tried to think like an engineer working at a startup rather than aiming for a perfect or over-engineered solution. The requirement is to track field employees’ locations in real time and show them on a manager’s dashboard. This means we need frequent updates, decent reliability on mobile networks, and something that won’t drain the phone battery or burn too much money.

Below is my analysis of different real-time communication approaches and the option I’d personally choose for Unolo.

---

## 1. Technology Comparison

### 1. WebSockets

**How it works:**
WebSockets open a persistent, two-way connection between the client and the server. Once connected, both sides can send data at any time without making repeated HTTP requests.

**Pros:**

* True real-time, bi-directional communication
* Very low latency
* Widely supported in browsers and backend frameworks

**Cons:**

 Maintaining thousands of open connections can be resource-heavy
 Requires careful handling of reconnections on flaky mobile networks
 Slightly more complex to scale (load balancers, sticky sessions, etc.)

**When to use:**
Best when you need instant updates in both directions (e.g., chat apps, live collaboration tools).

---

### 2. Server-Sent Events (SSE)

**How it works:**
SSE uses a single long-lived HTTP connection where the server continuously pushes updates to the client. Communication is one-way (server → client).

**Pros:**

* Simpler than WebSockets
* Built on standard HTTP
* Automatic reconnection support in browsers

**Cons:**

* One-way only (client can’t push data on the same connection)
* Not ideal for mobile apps compared to browsers
* Limited support compared to WebSockets

**When to use:**
Good for dashboards or live feeds where clients mostly listen for updates.



### 3. Long Polling

**How it works:**
The client repeatedly sends requests to the server, and the server holds the request open until new data is available or a timeout occurs.

**Pros:**

* Simple to implement
* Works everywhere
* No special protocols required

**Cons:**

* Inefficient at scale
* Higher latency compared to WebSockets
* Increased server load due to frequent requests

**When to use:**
Only suitable for small-scale systems or when real-time requirements are loose.

---

### 4. Third-Party Services (Firebase / Pusher / Ably)

**How it works:**
These services provide managed real-time messaging and syncing, handling scaling and connections for you.

**Pros:**

* Very fast to integrate
* Handles scaling and reliability automatically
* Less backend complexity

**Cons:**

* Cost increases quickly with usage
* Vendor lock-in
* Less control over internals

**When to use:**
Best when speed of development matters more than long-term cost or control.

---

## 2. My Recommendation

For Unolo’s use case, I would recommend **WebSockets**.

**Why WebSockets make sense here:**

* We need near real-time updates (every ~30 seconds) for thousands of users
* The manager dashboard benefits from live updates without polling
* Location updates are small payloads, so bandwidth usage is reasonable
* We already have a backend, so adding WebSocket support is realistic

Even though WebSockets are slightly more complex, they strike a good balance between performance, control, and cost for a startup.

---

## 3. Trade-offs

Choosing WebSockets does come with trade-offs:

* **Complexity:** We need to handle reconnections, dropped connections, and scaling carefully
* **Infrastructure:** Load balancers and horizontal scaling need extra attention
* **Battery usage:** Keeping a persistent connection open uses more battery than occasional HTTP calls

I would reconsider this approach if:

* The team is very small and needs a faster solution → a managed service might be better
* Scale grows far beyond expectations (100k+ users constantly online)

At very large scale, we might need a message broker (Kafka, Redis Streams) or move to a managed real-time platform.

---

## 4. High-Level Implementation Plan

**Backend:**

* Add a WebSocket server (e.g., using Socket.IO or native WebSockets)
* Authenticate connections using JWT
* Accept location updates from mobile clients and broadcast them to managers

**Frontend / Mobile:**

* Mobile app sends location updates every 30 seconds
* Manager dashboard subscribes to updates and renders live markers on a map

**Infrastructure:**

* Node.js server with WebSocket support
* Redis for pub/sub if multiple backend instances are used
* Load balancer configured for WebSocket connections

---

## Final Thoughts

There is no perfect solution for real-time systems. My recommendation is based on practicality, cost awareness, and development effort. WebSockets give us flexibility and performance while still being realistic for a startup like Unolo.

If requirements or scale change significantly, the architecture should evolve accordingly rather than trying to over-engineer from day one.
