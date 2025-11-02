# Understanding Distributed Systems

![rw-book-cover](https://m.media-amazon.com/images/I/61otEO9SSYL._SY160.jpg)

## Metadata
- Author: [[Roberto Vitillo]]
- Full Title: Understanding Distributed Systems
- Category: #books

## Highlights
- The responsibility of building and communicating the routing tables across routers lies with the Border Gateway Protocol (BGP1). ([Location 190](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=190))
    - Tags: [[pink]] 
- To create the illusion of a reliable channel, TCP partitions a byte stream into discrete packets called segments. ([Location 196](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=196))
    - Tags: [[pink]] 
- The operating system manages the connection state on both ends through a socket. ([Location 201](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=201))
    - Tags: [[pink]] 
- Unlike TCP, UDP does not expose the abstraction of a byte stream to its clients. ([Location 252](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=252))
    - Tags: [[orange]] 
- Request methods can be categorized based on whether they are safe and whether they are idempotent. A safe method should not have any visible side effects and can safely be cached. An idempotent method can be executed multiple times, and the end result should be the same as if it was executed just a single time. ([Location 467](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=467))
    - Tags: [[pink]] 
- The fair-loss link model assumes that messages may be lost and duplicated, but if the sender keeps retransmitting a message, eventually it will be delivered to the destination. The reliable link model assumes that a message is delivered exactly once, without loss or duplication. A reliable link can be implemented on top of a fair-loss one by de-duplicating messages at the receiving side. The authenticated reliable link model makes the same assumptions as the reliable link but additionally assumes that the receiver can authenticate the sender. ([Location 682](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=682))
    - Tags: [[pink]] 
- The arbitrary-fault model assumes that a process can deviate from its algorithm in arbitrary ways, leading to crashes or unexpected behaviors caused by bugs or malicious activity. For historical reasons, this model is also referred to as the “Byzantine” model. More interestingly, it can be theoretically proven that a system using this model can tolerate up to of faulty processes1 and still operate correctly. ([Location 689](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=689))
    - Tags: [[pink]] 
- The crash-recovery model assumes that a process doesn’t deviate from its algorithm but can crash and restart at any time, losing its in-memory state. The crash-stop model assumes that a process doesn’t deviate from its algorithm but doesn’t come back online if it crashes. Although this seems unrealistic for software crashes, it models unrecoverable hardware faults and generally makes the algorithms simpler. ([Location 693](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=693))
    - Tags: [[pink]] 
- The rate at which a clock runs faster or slower is also called clock drift. In contrast, the difference between two clocks at a specific point in time is referred to as clock skew. ([Location 749](https://readwise.io/to_kindle?action=open&asin=B09YLRB7QV&location=749))
    - Tags: [[pink]] 
