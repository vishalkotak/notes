### Functional Requirements
- **Identify & Block:** The system must identify requests originating from specific IP addresses or authenticated user IDs.
- **Rate Limiting Trigger:** Blocking should be triggered when an identifier (IP/User ID) exceeds a predefined request rate limit.
- **Deny Action:** Once an identifier is on the deny list, subsequent requests from it should be blocked (e.g., return HTTP 403 Forbidden or 429 Too Many Requests).
- **Synchronization:** The deny list status (which identifiers are blocked) must be propagated and enforced consistently across all application servers, potentially spanning multiple data centers or cloud regions.
- **Temporary Blocking:** Blocks should typically be temporary, with a configurable Time-To-Live
- **Manual Override:** Provide a mechanism to manually add or remove identifiers from the deny list.
### Non Functional Requirements
- **High Availability:** The deny list checking mechanism must be highly available. Failure should ideally "fail open" (allow traffic) rather than blocking legitimate users, unless strict security requires "fail closed" (which is riskier). The update/synchronization mechanism should also be highly available.
- **Low Latency:** Checking if a request should be blocked must add minimal latency to the request path (ideally single-digit milliseconds).
- **Consistency:** The deny list needs to be _eventually consistent_ across all servers and regions. While strong consistency is ideal, the latency penalty might be too high for this use case. A short delay (seconds) in propagation is usually acceptable for DDoS mitigation.
- **Durability:** The list of blocked identifiers (especially manual additions) should be durable, surviving server restarts or transient failures. The state derived purely from rate limiting might be less critical to persist long-term.
### Core Entities

- DenyListEntry
- RateLimiterCounter
### API Endpoints

- POST /v1/denylist/entries
```
{ "identifier_type": "IP", "identifier_value": "...", "ttl_seconds": 3600, "reason": "..." }
```

- DELETE /v1/denylist/entries
```
{ "identifier_type": "IP", "identifier_value": "..." }
```

- GET /v1/denylist/check?type=IP&value=...

### HLD

![[Pasted image 20250420211516.png]]
### Schemas
#### DenyEntryList
- IdentifierType (IP, USER_ID)
- IdentiferValue ('127.0.0.1'. 'user123')
- ExpirationTimestamp
- Reason
- CreationTimestamp
### Questions
#### How will deny entry lists be added
- Admin can add a new entry using the **DenyEntryService**
- Once the rate limiter decides that the limit has been breached, it can create a new entry through the DenyEntryService
#### What happens when a call is made to the DenyEntryService
- The record first gets persisted to a database
- The record also gets pushed to a pub/sub system such as Kafka
#### How will our apps get updates regarding the DenyEntryList
- The app services will be polling the Kafka topics to get updates regarding the DenyEntryList
- Once it gets new records it can add this to the local cache
- If the App Service instance dies, we can load the deny list from the database or Kafka
#### What happens if we want to remove someone from the deny list
- We can mark the record in the database with tombstone to record a soft delete
- We can also add a new record in the Kafka topic with a passed expiration time so our local caches can be updated on app service instances
#### How will the checks be fast on the App Services
- The App Services are using local cache to check for deny list
- The rate limit checks don't block the request currently. This can happen for every request in the background
#### What if the deny list cannot fit in memory
- Option 1: Distributed Database such as DynamoDB. Increased Latency
- Option 2: Distributed Cache: Less durable
- Option 3: Regional DB + Async Replication: Complex to manage if pub/sub fails
#### Rate Limiting Implementation
- Sliding Window Technique
- Redis instance per region
- Rate Limiting Service can be an independent service to avoid tight coupling with the deny list blocking component
#### Move deny list to CDN
- Most CDN's provide a Deny List component with a compatible API
#### TTL
- Every record will have a TTL
- We can use DynamoDB which offers a TTL functionality for storage
- Our local caches need to consider TTL and can remove entries from the local cache if they are expired (Multithreading might be needed)