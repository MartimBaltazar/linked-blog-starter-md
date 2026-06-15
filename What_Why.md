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
Why what is service account

