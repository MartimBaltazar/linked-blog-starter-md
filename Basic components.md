### Pods

Smallest deployable unit in Kubernetes. It holds one or more containers operating in the same network namespace, sharing storage and networking resources such as a single IP address, allowing tightly coupled containers to run together.

The main container is where your application lives. Pods support two additional container types:

- An **init container** runs before the main container starts, handling setup tasks like checking dependencies.

- A **sidecar container** runs alongside the main container, handling supporting tasks (e.g. filebeat that collects logs from main container, like when main container writes logs to stdout/stderr)

### Nodes

A node is a machine in a Kubernetes cluster. Worker nodes are where pods actually run. Every cluster has two main parts:

- **The [[#Control Plane]]** manages the cluster's state. It decides where pods run, whether containers should restart, and how scaling should occur.

- **Worker nodes** run the actual application workloads. When you deploy an application, it is packaged into pods and scheduled onto worker nodes.

### Service

Pods are disposable. When they are killed and replaced, their internal IP addresses change. A service solves this by providing a stable entry point that maps to those shifting pods, keeping the application reachable.

Kubernetes service types:

- **ClusterIP**: Assigns a stable cluster IP address, reachable from inside the cluster. This is the default type and is used for pod-to-pod communication.

- **NodePort**: Opens a static port on every node's IP. It allows external access to the service from outside the cluster.    

- **LoadBalancer**: Exposes the service using an external load balancer from your cloud provider. Sits on top of NodePort.

- **ExternalName:** Maps a service to an external DNS name, routing traffic to a service outside the cluster.

### Namespaces

Namespaces let you create multiple virtual clusters within a single physical cluster. Different teams get their own partition without stepping on each other. Resource names only need to be unique within a namespace, so conflicts across teams are not an issue

### Control Plane
 
Brain of the cluster. It runs as a group of control plane components, including the API server, scheduler, and controller manager. In production clusters, these components run across multiple nodes to improve reliability and avoid a single point of failure.

Imagine you deploy an online store. You package the application into a container and push it to the cluster. The control plane schedules the pods onto worker nodes. If traffic increases, the deployment scales. If a node fails, it reschedules the workload to another node.

### ReplicaSet

Kubernetes controller that keeps a stable number of identical pod replicas running in the cluster.

You declare the desired count. From that point, it watches the actual state. If one pod crashes or a node fails, a new one is created on a healthy node. If there are more pods than specified, the extras are terminated.

![[Pasted image 20260611090721.png]]
### Deployments

When you create a Kubernetes deployment, you are telling Kubernetes the desired state: how many replicas to run, which container image to use, and how much resource each pod needs. Kubernetes works to make that state real across the Kubernetes nodes in your cluster.

Behind the scenes, the Deployment creates a ReplicaSet that manages the actual pods. If a pod crashes or a node fails, the ReplicaSet creates a replacement automatically.

You can also create an **HPA (horizontal pod autoscaler)** that targets your deployment.

### Rolling update

Keeps your application responsive while the backend is being updated. Instead of shutting everything down at once.

Here’s how it works:

- Kubernetes creates a new pod with the updated version of the application.

- It waits for that pod to pass its readiness check.

- Once the new pod is healthy, the old pod is terminated.

This process repeats until all pods are running the new version.

#### What happens when you run a Kubectl command?

Kubectl reads the kubeconfig file to determine which cluster to talk to. The request is then sent to the API server. The API server authenticates you, checks your permissions through RBAC, and then processes the request. If the request changes the cluster state, the API server stores that change in etcd.

#### Why does every request in Kubernetes go through the API server?

Kubernetes is structured so that all communication happens via the API, which is the central point. All requests must pass through it, whether from kubectl, the scheduler, or any other component.

Imagine if components could update cluster data directly without validation. You'd have inconsistent states and security issues. The API server checks every request, confirms permissions, and then either performs the action or returns information.

### ETCD

Etcd is the distributed key-value store that holds the cluster's data. It stores configuration details, metadata, and system state, everything the cluster needs to function.

It is built to be fault-tolerant. Even if a node fails or a network partition occurs, etcd preserves data consistency using quorum-based consensus. In a multi-node control plane setup, this keeps the cluster state reliable when things go wrong.

### DaemonSet

Ensures one copy of a pod runs on every node in the cluster. Add a new node, and the pod lands on it automatically. Remove a node, and the pod goes with it. (e.g. the filebeat we have running as a daemonset reads the disk in node for logs written there)


### HPA vs Karpenter

**HPA** and **Karpenter** are not competing technologies, but rather ==complementary tools that work together at different layers of your infrastructure stack==.

- **HPA (Horizontal Pod Autoscaler):** Scales your _applications_ (Pods). It monitors metrics like CPU or memory and adds more pod replicas when traffic spikes. 

- **Karpenter:** Scales your _infrastructure_ (Nodes/Servers). When your HPA scales up and there are not enough servers to host the new pods, Karpenter steps in to instantly provision the exact size and type of cloud instance needed.
- 
When **HPA** creates more pods than your current cluster can fit, Karpenter instantly steps in to supply the exact machine(s) needed to run them


### HTTPRoute

HTTPRoute is part of the Kubernetes Gateway API. It defines how HTTP traffic should be routed to backend services based on conditions like hostname, path, headers, or query parameters. Think of it as a more flexible, standardized replacement for Ingress. An HTTPRoute is attached to a Gateway (the entrypoint) and specifies rules like: "route/api/* to the API service, but /static/*
to the static content service." 

It supports advanced features like weight-based traffic splitting for canary deployments, retries, and request modifications. The key difference from Ingress: HTTPRoute is explicitly bound to a Gateway resource, making it clearer how traffic flows from the edge into your cluster.

### Vi-gateway

A Gateway is the entry point into your cluster. It acts as a proxy that receives external traffic and routes it to services based on HTTPRoute rules. Unlike Ingress (which is more of a configuration), a Gateway is an actual load balancer or reverse proxy that you deploy. 

You define a GatewayClass to specify which controller manages the Gateway (e.g., nginx, Envoy, cloud provider). Then you create a Gateway resource that listens on certain ports and protocols. HTTPRoutes attach to this Gateway and define where traffic should go. Gateways provide better separation of concerns: infrastructure teams manage the Gateway, application teams manage HTTPRoutes

### Pod.namespace.svc.local to connect to service (clusterip)

Pods discover services via DNS without needing to know their IP address. When a pod needs to connect to a service, it queries the cluster DNS using the service's fully qualified domain name. The pattern is:

<service-name>.<namespace>.svc.cluster.local

For example, a pod in the default namespace connecting to a service named api in the
payment namespace would use api.payment.svc.cluster.local. If both are in the same namespace, you can simply use api. 
The DNS resolver returns the stable ClusterIP of the service, and traffic is load-balanced across the backing pods. This abstracts away pod IP churn — as pods die and restart with new IPs, the DNS name stays constant.


### How are pods named


**`<deployment-name>-<pod-template-hash>-<random-suffix>`**



