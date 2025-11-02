##### Requirements
- Schedule a task to be executed
- Schedule a task to be executed on a schedule
- 30 Seconds SLA
- Multi-Tenant = 2B Tasks / Day
- Each team = Tenant in the org
##### High Level Diagram

![[Pasted image 20250330174456.png]]
##### Deep Dive

Reference: https://drive.google.com/file/d/1Pesb6goi9aA5GkS9XTiEZ9On8n1VOype/view (Arpit)

![[Pasted image 20250330180819.png]]
- Poller service will also look at cron schedule and add it to jobs for next 10 run
#### Additional Notes
- **Worker failure**
	- Arpit had mentioned that the workers will be pinged using health check endpoints using central orchestrator to determine if a job needs to be rerun on a different worker if the worker fails
	- Hello Interview: Adding this orchestrator is overhead because health checks in a distributed system can get complicated. What if there is a network failure between the orchestrator and the worker. 
		- Solution 1: Use redis as distributed locking service. When a worker receives a job from the queue, it can acquire a lock on the job with its worker id. If the worker needs more time than the lease allocated time, it needs to refresh the lease with a new timestamp. What if the worker is not able to connect to Redis, another worker might acquire the lease as the first worker's lease would have expired
		- **Solution 2:** Amazon SQS uses **visibility timeouts** to manage worker failures. When a worker picks up a message, it becomes invisible to others for a set time. If the worker fails or crashes before processing it, the message becomes visible again for others to retry. To support **long-running jobs** while enabling **quick recovery from failures**, workers can set a **short visibility timeout** (e.g., 30 seconds) and **periodically extend it** using the `ChangeMessageVisibility` API (heartbeating). For instance, a 5-minute job can extend the timeout every 15 seconds.
			- No need for extra infrastructure or coordination.
			- Automatically retries failed messages and supports **dead-letter queues**.
- **Idempotency**
	- Solution 1: Check if the job has been executed. If it is then skip the execution
		- Adds database operation on worker node 
		- Also, possible race condition. A worker has sent the database request and same request comes on another worker node
	- **Solution 2: Let the job take care of idempotency**

### Security

```
# --- Template Secure Docker Run Command ---
docker run \
    --rm \
    --read-only \
    --network none \
    --memory 128m \
    --memory-swap 128m \
    --cpus 0.5 \
    --pids-limit 100 \
    --ulimit nofile=1024:2048 \
    --user 1000:1000 \
    --cap-drop ALL \
    --security-opt no-new-privileges \
    --security-opt seccomp=/path/to/your/seccomp_profile.json \
    --security-opt apparmor=your_docker_profile \
    --tmpfs /run:size=16m,noexec,nosuid,nodev \
    --tmpfs /tmp:size=64m,noexec,nosuid,nodev \
    --volume /path/on/host/to/user_code:/app/code:ro \
    --volume /path/on/host/for/output:/app/output:rw \
    --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 \
    your-secure-python-image python /app/code/user_script.py # Or your Java command

# Note: Replace placeholders like paths, limits, user ID, profile names, image name, etc.
```

- --rm: Automatically removes the container's filesystem when the container exits.
- --read-only: Mounts the container's root filesystem as read-only. It forces all writes to explicitly defined volumes or temporary filesystems
- --mount type=bind,source=/host/path,target=/container/path: Mounts a directory or file from the host into the container. Only mount the _specific_ directories needed
- --memory (limit): Restricts the maximum amount of RAM the container can use (e.g., `128m`, `1g`).
- --cpus (limit): Restricts the amount of CPU time the container can use. `0.5` means 50% of one CPU core, `2.0` means 2 full CPU cores.
- --pids-limit (limit): Limits the number of concurrent processes (PIDs) that can be created inside the container. The default is typically unlimited (or very high).
- --network (type): Connects the container to a network.
	- `none`: **Most Secure.** The container has no network stack (no loopback interface, no external access). Use this if the user code doesn't need _any_ network communication.
- --cap-drop ALL: Drops _all_ Linux capabilities for the container. Capabilities are fine-grained permissions that break down the traditional power of the root user.
- --security-opt no-new-privileges: Prevents any process inside the container from gaining _additional_ privileges via mechanisms like `setuid` or `setgid` binaries or `file capabilities`
- 
