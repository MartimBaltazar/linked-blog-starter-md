#### What happens when you run a Kubectl command?

Kubectl reads the kubeconfig file to determine which cluster to talk to. The request is then sent to the API server. The API server authenticates you, checks your permissions through RBAC, and then processes the request. If the request changes the cluster state, the API server stores that change in etcd.

#### Why does every request in Kubernetes go through the API server?

Kubernetes is structured so that all communication happens via the API, which is the central point. All requests must pass through it, whether from kubectl, the scheduler, or any other component.

Imagine if components could update cluster data directly without validation. You'd have inconsistent states and security issues. The API server checks every request, confirms permissions, and then either performs the action or returns information.

#### Why can’t Kubectl work without a kubeconfig file?

A kubeconfig file resides on the machine where kubectl is installed. It holds the cluster endpoint and the credentials needed to authenticate. When you run a kubectl command, that is the first thing it reads. Without a valid endpoint and credentials, kubectl has no reference point. It wouldn't know which cluster to target or how to verify your identity.

#### How does Kubernetes scheduling work?

The scheduler decides which node a pod should run on. When you create a pod, it has no node assigned yet. The scheduler watches for pods in this unscheduled state and checks available CPU, memory, and constraints across all nodes before making a decision.

The scheduler logic is a mixture of three phases: filtering, scoring, and binding.

- **Filtering**: removes any node that cannot satisfy the pod's requirements, such as not enough CPU, not in a ready state, or wrong taint.

- **Scoring** ranks the remaining nodes based on available resources, affinity rules, and other preferences. The pod lands on the highest-scoring node.

- **Binding:** assigns the pod to the winning node by updating the API server. The kubelet on that node takes over and starts the pod.

#### What is the difference between Deployment and StatefulSet?

Deployments and StatefulSets both manage pod replicas but treat identity and storage differently.

A Kubernetes deployment works best for stateless applications like website frontends and [REST APIs](https://roadmap.sh/questions/rest-api). Pods can be created or destroyed because nothing depends on local storage. If one crashes, another takes its place, and nobody notices.

A StatefulSet is built for stateful applications like databases. Each pod has a unique identity and requires persistent storage that survives restarts. If a database pod is recreated, it comes back with the same name and the same persistent volumes. Without that stability, the database loses track of its whole data.

#### How does Kubernetes handle scaling?

Scaling means adjusting the number of running pods based on demand. **HPA** and **Karpenter** are not competing technologies, but rather ==complementary tools that work together at different layers of your infrastructure stack==.

- **HPA (Horizontal Pod Autoscaler):** Scales your _applications_ (Pods). It monitors metrics like CPU or memory and adds more pod replicas when traffic spikes. 

- **Karpenter:** Scales your _infrastructure_ (Nodes/Servers). When your HPA scales up and there are not enough servers to host the new pods, Karpenter steps in to instantly provision the exact size and type of cloud instance needed.
- 
When **HPA** creates more pods than your current cluster can fit, Karpenter instantly steps in to supply the exact machine(s) needed to run them

### Changing/creating Namespace (new service account)

They don't move—they **must be recreated**. K8s resources are namespace-bound (except cluster-scoped like ClusterRoles, PV, StorageClasses).

A **Service Account** is a K8s identity for pods (like a "user" for containers). They are tied to a namespace so, when you create a new namespace, you must create ServiceAccounts for that namespace. Existing SAs from other namespaces won't be available.

### Helm charts and their relationships with configmaps vs dynamic values

A templated, reusable package of K8s manifests (**reusable K8s manifests with placeholders (.Values.image.tag, .Values.replicas, etc.**).

**Inject config via values.yaml**. Deploy with **helm install.** Instead of creating manifests for every element in pyroscope, you simply use the helm chart.

**Dynamic Values** = Helm template parameters overridden at **deploy time**. Used when deploying the chart itself (chart setup, image tags, replicas, environment). Changes require redeployment.

**ConfigMaps** = K8s objects created by Helm to inject config into **pods at runtime**. When pod apps need to read config after chart is deployed. ConfigMap changes don't require Helm redeployment—app picks up changes (if watching).


### Pod 0/1 Ready (Not CrashLoop): What It Means

The container is running, but the readiness probe is failing.

App isn't responding, dependencies missing, wrong endpoint or it might be simply initializing


#### How does networking work in Kubernetes?

By default, pods in a Kubernetes cluster can communicate with each other directly. Each pod receives its own IP address, and Kubernetes networking allows pods to communicate across nodes as if they were on the same network. 

Pod IPs are temporary and change when a pod is recreated. To solve this, Kubernetes uses Services, which provide a stable IP address and load balance traffic across pods.

Ingress sits on top of Services. It manages incoming traffic from external clients and routes requests to the correct service based on domains or URL paths, using an ingress controller to enforce these rules.

#### What are ConfigMaps and Secrets?

ConfigMaps and Secrets keep configuration and sensitive information out of your container images.

Hardcoding values into application code creates security risks and makes the codebase harder to maintain. Kubernetes lets you inject configuration at runtime instead.

A ConfigMap holds non-sensitive information like URLs, port configurations, and environment variables. A Secret is for database passwords, API keys, and tokens.

#### How do you mount a secret in a pod?

A Secret can be mounted as a volume or injected as an environment variable. When mounted as a volume, the application reads sensitive data from the file system.

When used as an environment variable using secretKeyRef, a particular key from the secret maps to a variable in the container. This method is often used for database passwords or API tokens.