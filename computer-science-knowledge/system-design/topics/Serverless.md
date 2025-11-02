Reference:
- https://www.datadoghq.com/knowledge-center/serverless-architecture/

Serverless architecture is an approach to software design that allows developers to build and run services without having to manage the underlying infrastructure. One of the most popular serverless architectures is Function as a Service (FaaS), where developers write their application code as a set of discrete functions. Each function will perform a specific task when triggered by an event, such as an incoming email or an HTTP request.

![[Pasted image 20250412202430.png]]
#### Fundamental Concepts
- **Invocation**: A single function execution.
- **Duration**: The time it takes for a serverless function to execute.
- **Cold Start**: The latency that occurs when a function is triggered for the first time or after a period of inactivity.
- **Concurrency Limit**: The number of function instances that can run simultaneously in one region, as determined by the cloud provider. A function will be throttled if it exceeds this limit.
- **Timeout**: The amount of time that a cloud provider allows a function to run before terminating it. Most providers set a default timeout and a maximum timeout.
#### Pros
- Cost
- Scalability
- Productivity of developers
#### Cons
- Loss of control
- Security
- Performance impact - cold start can add latency
- Testing - Hard to test locally
- Vendor Lock In

