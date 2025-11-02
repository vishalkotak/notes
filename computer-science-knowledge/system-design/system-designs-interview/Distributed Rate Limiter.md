- Jordan has no life
	- https://www.youtube.com/watch?v=MDH7xybVr8o
	- https://www.youtube.com/watch?v=VzW41m4USGs&t=1s

![[Pasted image 20250411120943.png]]

- Exponent - https://www.youtube.com/watch?v=SgWb6tWx3S8&t=1014s 

![[Pasted image 20250411121504.png]]

#### Sample Rule Config

```
{
  "rule_id": "api_user_get_orders",
  "description": "Limit GET requests to /orders/* per authenticated user",
  "identifier_type": "user_id", // What identifier is USED for the Redis key
  "algorithm": "sliding_window",
  "limit": 50,
  "window_size_seconds": 60,

  "match": { // Criteria to SELECT this rule
    "path_pattern": "/orders/*",
    "methods": ["GET"],
    "requires_authentication": true // Implies an identifier like user_id must be available
    // Optionally add: "required_headers": { "X-Client-Type": "mobile" }
    // Optionally add: "ip_subnet": "192.168.1.0/24" 
  },
  "priority": 10 // Optional: Lower number means higher priority if multiple rules match
}

// Another example: IP-based limit for login attempts
{
  "rule_id": "login_attempt_ip",
  "description": "Limit POST requests to /auth/login per IP",
  "identifier_type": "ip_address",
  "algorithm": "fixed_window",
  "limit": 5,
  "window_size_seconds": 300,

  "match": {
    "path_pattern": "/auth/login",
    "methods": ["POST"]
    // requires_authentication would likely be false here
  },
  "priority": 5 // Higher priority than the general user rule
}
```

#### Rate Limit Execution
- Parse the incoming request to get the below
	- Path
	- Method
	- Authentication
	- IP Address
- Iterate over the rules
	- Rule 1
		- Does not match
	- Rule 2
		- Does match
	- If multiple rules match, we will have to apply all the rules and execute AND operation
- Collect the list of matching rules
- Execute the rate limit check for the rule using the mentioned algorithm
#### Fixed Window Check
##### Key
ratelimit:<rule_id>:<identifier_value>:<window_start_ts>
- window_start_ts:  At `t=1713650375`,  the corresponding value will be 1713650340
- Example: ratelimit:api_reads_ip_fixed:1.2.3.4:1713650340
##### LUA Script

```
-- KEYS[1]: The Redis key for the specific entity and window (e.g., "ratelimit:api_reads_ip_fixed:1.2.3.4:1713650340")
-- ARGV[1]: limit (max requests allowed)
-- ARGV[2]: window_size_seconds (TTL for the key)

local key = KEYS[1]
local limit = tonumber(ARGV[1])
local expire_seconds = tonumber(ARGV[2])

-- Increment the counter for the current window
-- INCR is atomic and returns the new value
local current_count = redis.call('INCR', key)

local allowed = false
if current_count <= limit then
  allowed = true
end

-- If this is the first increment (count is 1), set the expiry
-- Ensures the key cleans up after the window passes
if current_count == 1 then
  redis.call('EXPIRE', key, expire_seconds)
end

-- Return { Allowed (0=false, 1=true), Current Count }
if allowed then
  return {1, current_count}
else
  -- Even if denied, return the count that caused the denial
  return {0, current_count} 
end
```
##### Sliding Window Check
- Given `window_size = 60` seconds and the current time 7:20:46 PM, the _current_ fixed window is **7:20:00 PM - 7:20:59 PM**.
- Therefore, the _previous_ fixed window was **7:19:00 PM - 7:19:59 PM**.
- Since the current minute started at 7:20:00 PM and the time is 7:20:46 PM, `time_in_current_window` is **46 seconds**.
- `window_size - time_in_current_window`: This is `60 - 46 = 14` seconds. This represents the duration of the _previous_ fixed window (7:19:00-7:19:59) that still overlaps with our target sliding window (7:19:47-7:20:46). Specifically, the overlapping period is 7:19:47 PM to 7:19:59 PM, which is indeed 14 seconds long.
- Calculate the _proportion_ of the previous window's duration that overlaps. `14 / 60 ≈ 0.233`.
- The **fractional weight (≈ 23.3%)** given to the _previous minute's total count_ (`prev_count`) to estimate how many requests likely occurred within the **last 14 seconds** of that previous minute (from 7:19:47 to 7:19:59). This assumes requests were evenly distributed throughout the 7:19 PM minute
- approximates the total number of requests in the sliding window ending _now_ (7:19:47 PM to 7:20:46 PM) by combining:
	- An _estimated_ count for the portion of the window in the previous minute (7:19:47 - 7:19:59).
	- The _actual_ count for the portion of the window in the current minute (7:20:00 - 7:20:46).
###### Example LUA Script

```
-- KEYS[1]: Current window key (e.g., "ratelimit:login_attempts_user_sliding:user456:1713650340")
-- KEYS[2]: Previous window key (e.g., "ratelimit:login_attempts_user_sliding:user456:1713650280")
-- ARGV[1]: limit (max requests allowed in sliding window)
-- ARGV[2]: window_size_seconds
-- ARGV[3]: current_time_seconds (epoch seconds)

local current_key = KEYS[1]
local previous_key = KEYS[2]
local limit = tonumber(ARGV[1])
local window_size = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

-- Get counts from previous and current windows
local prev_count = tonumber(redis.call('GET', previous_key)) or 0
local curr_count = tonumber(redis.call('GET', current_key)) or 0

-- Calculate the weight for the previous window's contribution
local time_in_current_window = now % window_size
local prev_window_weight = (window_size - time_in_current_window) / window_size

-- Calculate the approximate count in the sliding window *before* incrementing
local estimated_count = (prev_count * prev_window_weight) + curr_count

local allowed = false
-- Check if adding one more request would exceed the limit
if (estimated_count + 1) <= limit then
  allowed = true
end

local final_current_count = curr_count -- Keep track of count for return value

-- If allowed, increment the current window count and set expiry
if allowed then
  -- Increment the current window count
  final_current_count = redis.call('INCR', current_key)

  -- Set TTL if it's the first hit in this specific window bucket
  -- Use PTTL to check remaining TTL; if -2 (no key) or -1 (no TTL), set it.
  -- Alternatively, simpler: set EXPIRE every time, Redis handles it efficiently.
  if final_current_count == 1 then
      -- Add a little buffer to ensure it covers the full window duration
      redis.call('EXPIRE', current_key, window_size + 5) 
  end

  -- Ensure previous key also has a reasonable TTL if it exists (optional, depends on strategy)
  -- If prev_count > 0 then
  --  redis.call('EXPIRE', previous_key, window_size + 5) 
  -- end

end

-- Return { Allowed (0=false, 1=true), Estimated Count *Before* Increment (for info) }
-- Note: Returning estimated_count helps clients know the state before their attempt.
--       Returning final_current_count might also be useful. Choose based on needs.
if allowed then
  return {1, estimated_count} 
else
  return {0, estimated_count}
end
```

#### Scaling
##### Regional Redis Clusters
- Deploy independent redis clusters in key geographic regions close to your users
- Our rate limiting service will connect to the Redis cluster in the same region to reduce cross continent latency
- Disadvantage:
	- Regional limits only. A user can go to a different region and get access as it doesn't enforce a global limit. 
	- Possible Solution
		- Divide the total allocation by the number of regions and allow only those many requests. Can lead to inaccuracies
		- Regional redis instances share state with each other using gossip protocols but these limits might be slightly delayed
##### Active-Active Geo-Replication (Advanced)
- Allows multiple redis clusters in different regions to accept writes and automatically replicate data to other participating clusters using Conflict-Free Replicated Data Types (CRDT's) to handle concurrent writes
- Pros
	- Global consistency than basic regional clusters
- Cons
	- Consistency is **strong eventual** than linearizability 
#### Redis Master Failure
- Have replicas for master node within a region
- Redis Sentinel can automatically failover to the replica to maintain availability
#### Redis Horizontal Scaling
- If a single master node cannot handle the request volume or store the entire dataset in memory, use Redis cluster
- Redis cluster automatically shards your data across multiple master nodes (splits the keyspace using 16384 hash slots). Each master can have replicas. Client library direct requests for a specific key to the correct shard/node
#### Zookeeper

##### Structuring Rules
- **Parent Znode:** Create a persistent parent znode to hold all rate limiting rules. For example: `/config/ratelimit/rules`
- **Individual Rule Znodes:** Under the parent znode, create a persistent znode for each rate limit rule. The name of the znode could be the `rule_id`. For example:
	- /config/ratelimit/rules/api_user_get_orders
	- /config/ratelimit/rules/login_attempt_ip
- **Rule Data:** Store the actual rule configuration (as defined previously: `identifier_type`, `algorithm`, `limit`, `window_size_seconds`, `match` criteria, etc.) as the data within each rule's specific znode. This data is typically stored as a byte array, often serialized in a format like JSON or Protocol Buffers.

![[Pasted image 20250420195707.png]]
##### Loading Initial Configuration on Startup
- **Connect:** It establishes a connection to the ZooKeeper ensemble (the group of ZooKeeper servers) using a ZooKeeper client library (Kazoo)
- **List Rules:** It calls `getChildren` on the parent znode (`/config/ratelimit/rules`). This returns a list of the names of the child znodes (the `rule_id`s: `["api_user_get_orders", "login_attempt_ip"]`).
- **Fetch Rule Data:** For each `rule_id` obtained, it calls `getData` on the corresponding child znode (e.g., `/config/ratelimit/rules/api_user_get_orders`). This retrieves the byte array containing the rule's JSON (or other format) data.
- **Deserialize & Cache:** The application deserializes the data into its internal rule objects and stores them in an **in-memory cache**.
- **Set Watches:** Crucially, while performing steps 2 and 3, the application sets _watches_ on the znodes.
##### Handling Updates via Zookeeper Watches
- **Watch on Parent (`getChildren` watch):** The application sets a watch when calling `getChildren` on `/config/ratelimit/rules`.
	- **Trigger:** This watch fires when a child znode is _created_ or _deleted_ under the parent (i.e., a new rule is added, or an existing rule is removed).
	- **Action:** When the watch fires, the application re-runs the `getChildren` operation (step 2 above) to get the updated list of rule IDs. It compares the new list with its cached list to identify added/removed rules. For added rules, it fetches their data (`getData`) and adds them to the cache, setting a data watch on the new rule znode. For removed rules, it removes them from its in-memory cache. It then _re-registers_ the `getChildren` watch for future changes.
- **Watch on Rule Znode (`getData` watch):** The application sets a watch when calling `getData` on each individual rule znode (e.g., `/config/ratelimit/rules/api_user_get_orders`).
	- **Trigger:** This watch fires when the _data_ within that specific znode is _modified_ (i.e., an existing rule's limit, window size, or match criteria are changed). It also fires if the znode is deleted (handled also by the parent watch).
	- **Action:** When the watch fires due to a data change, the application re-runs the `getData` operation for that specific rule znode. It deserializes the new data and updates the corresponding rule object in its in-memory cache. It then _re-registers_ the `getData` watch on that znode for future changes.
#### Workflow for Updating a Rule
- **Write to ZooKeeper:** To modify the limit for `api_user_get_orders`, the tool updates the data associated with the znode `/config/ratelimit/rules/api_user_get_orders` using a `setData` operation. To add a new rule, it uses `create` to add a new child znode under `/config/ratelimit/rules`. To delete, it uses `delete`.
- **Notify Watchers:** ZooKeeper notifies all connected application instances that have registered a relevant watch (either a `getData` watch on the modified znode or a `getChildren` watch on the parent znode if a rule was added/deleted).
#### Zookeeper Scalability
- Zookeeper follows an ensemble process where you have multiple nodes containing one leader and multiple followers
- Generally you will have odd number of servers
- All requests are followed to the leader
- Leader shares the update with the followers using Zookeeper Atomic Broadcast protocol
- Change is only committed after persisted by a quorum of followers
- Reading can happen on any follower. Clients are configured with all the servers from the ensemble and it picks one at random to connect with
- More nodes does not increase write throughput
- More nodes will increase read throughput
- If the leader fails, a new leader will be elected within seconds
- If a follower fails, client should realize it and connect to a different zookepeer instance
- Watches can be executed by any node
