#### References
- https://www.hellointerview.com/learn/system-design/deep-dives/redis
-  https://www.youtube.com/watch?v=fmT5nlEkl3U
#### Basics
- Written in C
- In-memory
- Single threaded
- Data Structures
	- Strings
	- Objects
	- Lists
	- Sets
	- Sorted Sets
	- Bloom Filters
	- Geospatial Indexes
	- Time Series
- Uses wire protocol - custom query language
- TTL for each key can be setup
#### Infrastructure Configurations
- Single Node
- Replicated
- Cluster
	- Nodes use gossip protocol to communicate. If you request a key from the wrong node, Redis will redirect to the correct node.
	- **Redis expects all data for a given request to be on a single node.** 
#### Performance
- O(100K) writes per second
#### Use Cases
- Cache
- Distributed locking
	- If you want to lock access to updating user profile `user:123`, you might use the Redis key `lock:user:123`
	- A process wanting the lock executes the `INCR lock:user:123` command on Redis.
	- `INCR` is an atomic operation in Redis. This means Redis guarantees that even if multiple processes send the `INCR` command at the exact same moment, Redis will execute them sequentially, one after the other, without interruption.
	- **Success (`INCR` returns 1):** If the `INCR` command returns `1`, it means this process was the _first_ one to increment the key (or the key didn't exist, so `INCR` created it and set it to `1`). This process has successfully acquired the lock.
    - **Failure (`INCR` returns > 1):** If the `INCR` command returns a value greater than `1` (e.g., 2, 3, etc.), it means another process had already incremented the key to `1` (or more) before this process's `INCR` command was executed. This process did _not_ acquire the lock.
- Leaderboard
- Rate Limiting
- Proximity Search
- Event Sourcing - Streams
#### Shortcomings
- Hot key problem
#### Single Threaded
![[Pasted image 20250322123229.png]]
#### Data Types and Scenarios
##### Sorted Sets
- Redis implements sorted sets using a combination of a **hash table** (mapping members to scores) and a **skip list** (a probabilistic data structure that keeps elements sorted by score and allows for efficient searching and range queries).

| Operation              | Descripton                                                                                                      | Time Complexity |
| ---------------------- | --------------------------------------------------------------------------------------------------------------- | --------------- |
| ZADD                   |                                                                                                                 | O(log N)        |
| ZINCRBY                | This involves finding the member (O(1) via hash table) and updating its position in the skip list (O(log N)).   | O(log N)        |
| `ZRANGE` / `ZREVRANGE` | It takes O(log N) to find the starting rank in the skip list and then O(M) to collect the M elements requested. | O(log N + M)    |
#### Geohash

- **The Problem:** Computers work well with numbers and strings, but latitude and longitude are pairs of floating-point numbers. Searching for "nearby" points using raw coordinates can be slow if you have many locations because you'd potentially have to calculate the distance between your target point and _every other point_.
- **The Solution (GeoHash):** GeoHashing is a clever technique that encodes a geographic location (latitude and longitude) into a short string of letters and numbers.
- **The Magic:** The key property of GeoHash is that **locations physically close to each other will often have similar GeoHash prefixes** (the beginning part of the string). It's like a hierarchical address system for the globe. The longer the shared prefix, the closer the points generally are.
##### How to use it?
###### Storing Locations (GEOADD)

```
# A key name (like a category, e.g., "restaurants" or "user_locations").
# The longitude.
# The latitude.
# A unique name for that specific location (e.g., "Central Park", "User123").

GEOADD landmarks -73.968285 40.785091 "Central Park"
```

_Behind the scenes:_ Redis calculates the GeoHash for these coordinates and stores it efficiently, linking it to the name "Central Park" under the key "landmarks".
###### Querying Nearby Locations (`GEORADIUS` or `GEORADIUSBYMEMBER`)
This is where it shines. You can ask Redis: "Find all locations within X meters/kilometers of this latitude/longitude" (`GEORADIUS`) or "Find all locations within X meters/kilometers of this _already stored location_" (`GEORADIUSBYMEMBER`).

https://redis.io/docs/latest/commands/georadius/

```
# GEORADIUS key longitude latitude radius <M | KM | FT | MI>
#  [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC | DESC]
#  [STORE key | STOREDIST key]

GEORADIUS businesses -73.9850 40.7480 5 MI WITHDIST WITHCOORD COUNT 1

>>1) 1) "Central Park"
	 2) "2.7087"
	 3) 1) "-73.96828383207321167"
		2) "40.7850917691370114"
```
#### Pub-Sub

https://redis.io/docs/latest/develop/interact/pubsub/

[`SUBSCRIBE`](https://redis.io/docs/latest/commands/subscribe/), [`UNSUBSCRIBE`](https://redis.io/docs/latest/commands/unsubscribe/) and [`PUBLISH`](https://redis.io/docs/latest/commands/publish/) implement the [Publish/Subscribe messaging paradigm](http://en.wikipedia.org/wiki/Publish/subscribe) where (citing Wikipedia) senders (publishers) are not programmed to send their messages to specific receivers (subscribers). Rather, published messages are characterized into channels, without knowledge of what (if any) subscribers there may be. Subscribers express interest in one or more channels and only receive messages that are of interest, without knowledge of what (if any) publishers there are. This decoupling of publishers and subscribers allows for greater scalability and a more dynamic network topology.
##### Notes
- Messages are received in the order they are published
- Clients subscribing shouldn't be publishing messages
- At-most once delivery
- Supports pattern matching using PSUBSCRIBE. For example clients can subscribe to news-* and it will receive messages from all channels with the pattern news-*
- No persistence
##### How it Works
- **`SUBSCRIBE channel_name`**: This command tells the Redis server, "I am interested in messages sent specifically to `channel_name`. Please send them to me as soon as they arrive." The connection stays open, waiting.
- **`PUBLISH channel_name "message content"`**: This command tells the Redis server, "Take this `message content` and send it immediately to _all clients that are currently subscribed_ to `channel_name`."
- **Decoupling:** The publisher doesn't know or care _who_ is subscribed. It just sends the message to the channel. The subscriber doesn't know who sent the message, only that it arrived on the channel it's listening to.
- **Fire-and-Forget:** Pub/Sub doesn't store messages; they are only delivered to clients connected _at the time of publishing_.

```
# Terminal 1

SUBSCRIBE news_channel

>>> 1) "subscribe" 2)
>>> 2) "news_channel" 3)
>>> 3) (integer) 1

# Terminal 2

PUBLISH news_channel "Hello everyone! Big news today."

>>> (integer) 1

# Terminal 1 Update
>>> 1) "message"
>>> 2) "news_channel" 
>>> 3) "Hello everyone! Big news today."
```
####  Streams