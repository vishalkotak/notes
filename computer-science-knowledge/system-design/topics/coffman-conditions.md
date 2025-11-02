**Coffman conditions** are necessary for a **deadlock** to occur in an operating system or a multi-threaded system.

### 1. Bounded Resources
- This means that **a limited number of resources** are available in the system.
- If multiple threads/processes request these resources and they are all allocated, the system **runs out of free resources**.

Example:
- Suppose there are **two printers** in a system, and three threads need to print.
- If two threads have acquired the printers, the third thread has to wait.
- If resources were infinite, deadlock would never occur.
### 2. No Preemption
- Once a resource (like a lock or a printer) is **allocated to a thread**, it **cannot be forcibly taken away**.
- The thread holding the resource must **release it voluntarily**.

Example:
- Thread A acquires **Printer 1**.
- Thread B acquires **Printer 2**.
- If Thread A now requests **Printer 2**, it **must wait** until Thread B releases it.
- If we allowed preemption (forcing Thread B to release the printer), the deadlock could be avoided.
### 3. Waiting While Holding Locks
- A thread/process may **hold some resources while waiting for additional ones**.
- This increases the chance of a **deadlock** if multiple threads are waiting on each other.

Example:
- Thread A acquires **Printer 1** and now waits for **Printer 2**.
- Thread B acquires **Printer 2** and now waits for **Printer 1**.
- Both are waiting **while holding a lock**, which can lead to deadlock.
### 4. Circular Waiting
- There exists a **circular chain of waiting processes**, each waiting for a resource held by the next process in the chain.
- This creates an **infinite wait** cycle.

Example:
- Thread A holds **Resource 1** and waits for **Resource 2**.
- Thread B holds **Resource 2** and waits for **Resource 3**.
- Thread C holds **Resource 3** and waits for **Resource 1**.
- This forms a **cycle**, meaning **no thread can proceed**.