1. If this app had 10,000 employees checking in simultaneously, what would break first? How would you fix it?

The first thing that would likely break is the backend API and database write throughput.
Right now, every check-in is a synchronous request that performs multiple database queries (assignment check, active check-in check, insert). With 10,000 concurrent users, this could easily overwhelm the SQLite database and cause slow responses or failures.

To fix this, I would:

-->Move from SQLite to a production-grade database like PostgreSQL or MySQL

-->Introduce rate limiting and possibly a queue-based approach for check-ins

-->Horizontally scale the backend using a load balancer

-->Cache read-heavy endpoints (like dashboard stats) using Redis

The current architecture is fine for small teams, but scaling would require changes at both the database and infrastructure level.

2. The current JWT implementation has a security issue. What is it and how would you improve it?

The main security issue is that sensitive information can accidentally be placed inside the JWT payload. Since JWTs are encoded but not encrypted, anything inside them can be decoded by the client.

I noticed this early and removed sensitive fields (like passwords) from the token payload.

To improve the implementation further, I would:

-->Keep JWT payload minimal (user id, role only)
-->Reduce token expiration time and use refresh tokens
-->Store tokens securely (HTTP-only cookies instead of localStorage)

These changes would significantly improve authentication security.

3. How would you implement offline check-in support?

For offline support, I would implement a local-first approach.
On the frontend:

i.  Store check-in data in IndexedDB or localStorage when the user is offline
ii. Mark those entries as “pending sync”

On the backend:

Accept batched check-ins when the device reconnects

i.  Validate timestamps and prevent duplicates.
ii. Resolve conflicts if the same user already has an active check-in

This way, employees can continue working without internet, and data consistency is maintained once connectivity is restored.

4. Explain the difference between SQL and NoSQL databases. Which would you recommend here and why?

SQL databases use structured schemas, strong relationships, and support complex queries with joins while on the other hand we can say that NoSQL databases are more flexible and scale easily but sacrifice relational guarantees.

For this Field Force Tracker application, I would recommend a SQL database because:

-->The data is highly relational (users, managers, clients, check-ins)
-->Queries involve joins and aggregations
-->Data consistency is critical (attendance, working hours)

NoSQL would only make sense if the data became extremely unstructured.

5. What is the difference between authentication and authorization? Where are they implemented here?

Authentication is about who the user is which means in authentication we actually verify who the user is, while authorization is about what the user is allowed to do which means that after we have verified the user then what all access do we give to the user.

In this codebase:

Authentication is handled using JWT tokens

Authorization is enforced through middleware, such as role checks (e.g., manager-only routes)

For example, managers can access dashboard reports, while employees cannot. Both are implemented cleanly and separately.

6. What is a race condition? Can you identify one in this codebase?

A race condition occurs when the outcome depends on the timing of concurrent operations.

A potential race condition exists during check-in creation, where two requests could theoretically pass the “no active check-in” check at the same time. This could result in duplicate active check-ins.

To prevent this, I would:

i.  Use database-level constraints or transactions.

ii. Lock rows during check-in creation.