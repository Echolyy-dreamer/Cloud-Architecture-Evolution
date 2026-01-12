🔄 Cache Consistency Strategy (CloudFront + Redis + Aurora)

This architecture adopts a path-scoped consistency model rather than attempting global strong consistency across all cache layers.

The system contains three potential caching layers:

CloudFront (Edge Cache) – Static assets and selectively cached dynamic responses

Redis (Application Cache) – Cache-Aside pattern

Aurora MySQL – Source of Truth (transactional consistency)

Consistency guarantees are enforced per request path, based on business criticality.

🧱 Read & Write Model
Read Path (Dynamic Content)
Client
 → CloudFront (MISS or bypass)
   → ALB
     → PHP Application
       → Redis (Cache-Aside)
         → Aurora MySQL


Redis is accessed using a cache-aside strategy

Aurora remains the authoritative data source

CloudFront acts as a routing and acceleration layer, not a consistency layer

Write Path
PHP Application
 → Aurora MySQL (COMMIT)
   → Redis Invalidation
   → Optional CloudFront Invalidation


Writes are always persisted to Aurora first

Cache invalidation occurs after successful database commit

No write-through or write-behind caching is used

🔐 Strong Consistency Paths (Critical Business Flows)

For strong consistency requirements (e.g. checkout, payment, user profile updates), the architecture explicitly bypasses all cache layers.

CloudFront Behavior Configuration
Setting	Value
Cache Policy	CachingDisabled
TTL	0
Forward Headers	All
Forward Cookies	All
Authorization	Enabled
Application Behavior

Redis is not consulted

MySQL transactions provide consistency guarantees

Consistency scope is limited to the request lifecycle

Strong consistency is achieved by path-level cache bypass, not cache synchronization.

🧠 Redis Cache Strategy (Cache-Aside with Version Guard)

Redis follows a cache-aside pattern with versioned keys to prevent stale overwrites.

Key Design
user:{user_id}:v{version}

Write Logic
UPDATE users
SET name = ?, version = version + 1
WHERE id = ?


Each update increments a version field

Cache keys are naturally invalidated by version mismatch

Prevents race conditions where stale data overwrites newer values

This approach avoids the complexity of distributed locking while ensuring logical consistency.

🌍 CloudFront Caching Strategy for Dynamic Content

Dynamic paths are handled using minimal TTL or explicit invalidation, rather than long-lived edge caching.

Content Type	Strategy
Static Assets	Immutable filenames + long TTL
Semi-dynamic	Short TTL (0–5s)
Strong consistency paths	Cache disabled

This ensures perceived consistency without introducing cache coordination complexity.

🧩 Consistency Design Principle

Consistency is enforced per request path, not globally.

This design intentionally avoids multi-layer strong consistency, which is costly and fragile at scale.
Instead, it combines explicit cache bypass, versioned cache keys, and transactional data ownership.

🎯 Interview-Grade Summary

Redis uses cache-aside, not write-through

Aurora MySQL is the single source of truth

CloudFront caching is selective and policy-driven

Strong consistency is guaranteed by path-based cache exclusion
