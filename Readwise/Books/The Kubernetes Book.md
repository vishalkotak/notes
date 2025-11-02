# The Kubernetes Book

![rw-book-cover](https://images-na.ssl-images-amazon.com/images/I/41db9ap7UML._SL200_.jpg)

## Metadata
- Author: [[Nigel Poulton]]
- Full Title: The Kubernetes Book
- Category: #books

## Highlights
- Control plane nodes implement the Kubernetes intelligence, and every cluster needs at least one. However, you should have three or five for high availability (HA). ([Location 220](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=220))
    - Tags: [[pink]] 
- It’s based on the etcd distributed database, and most Kubernetes clusters run an etcd replica on every control plane node for HA. ([Location 253](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=253))
    - Tags: [[pink]] 
- For example, multiple writes to the same value from different sources need to be handled. etcd uses the RAFT consensus algorithm for this. ([Location 265](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=265))
    - Tags: [[pink]] 
- Every worker node runs a kube-proxy service that implements cluster networking and load balances traffic to tasks running on the node. ([Location 305](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=305))
    - Tags: [[pink]] 
- Apps need to be wrapped in Pods to run on Kubernetes ([Location 322](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=322))
    - Tags: [[pink]] 
- Pods are normally wrapped in higher-level controllers for advanced features ([Location 322](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=322))
    - Tags: [[pink]] 
- The container wraps the app and provides dependencies The Pod wraps the container so it can run on Kubernetes The Deployment wraps the Pod and adds self-healing, scaling, and more ([Location 330](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=330))
    - Tags: [[pink]] 
- Terminology: We use the terms actual state, current state, and observed state to mean the same thing — the most up-to-date view of the cluster. ([Location 337](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=337))
    - Tags: [[pink]] 
- The most common way of posting YAML files to Kubernetes is with the kubectl command-line utility. ([Location 346](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=346))
    - Tags: [[pink]] 
- The imperative model requires complex scripts of platform-specific commands to achieve an end-state ([Location 353](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=353))
- For example, if a Pod has two containers and only one is started, the Pod is not ready. ([Location 389](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=389))
    - Tags: [[pink]] 
- Worker nodes also have one or more runtimes and the kube-proxy service. ([Location 423](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=423))
    - Tags: [[pink]] 
- Topology spread constraints ([Location 629](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=629))
- To select on Pods, the scheduler takes a similar list of labels and schedules the Pod to nodes running other Pods possessing the labels. ([Location 636](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=636))
- This is very different from the design goals of Pods and is why KubeVirt wraps VMs in a modified Pod called a VirtualMachineInstance (VMI) and manages them using custom workload controllers. ([Location 670](https://readwise.io/to_kindle?action=open&asin=B072TS9ZQZ&location=670))
    - Tags: [[pink]] 
