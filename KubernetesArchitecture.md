# Kubernetes Architecture Components

In any K8s cluster there are following components: 

- EtcD Cluster
- Kube ApiServer
- Kube Scheduler
- Kube Proxy
- Kubelet
- Controller Manager


Let's look at the functionality of each of the components.


# EtcD Cluster

- EtcD is a Key-Value datastore.
- In K8s it stores information about Nodes, Pods, Configs, Secrets and everything else.
- Every operation that you perform, it is updated in the EtcD cluster.

# Kube-ApiServer

- This is the primary management component in the K8s which is responsible for carrying out majority of operations. 
- To begin with, whenever you run a Kubectl command it talks to ApiServer. Your request is authenticated, validated and then you receive the response back, which is usually queried from EtcD cluster.
- ApiServer is the only component that talks to EtcD Cluster directly.
- You can talk to kube apiserver directly using curl commands as well.
- EtcD runs as a cluster and it can be deployed on same nodes as k8s or externally as well.
- It uses port 2379

## Example WF - Pod Creation

1. You request new pod to be created.
2. Your request is authenticated and validated by ApiServer.
3. ApiServer creates a `Pod object` and inserts it into EtcD cluster.
4. It reports back to you saying a Pod was created. In reality though Pod creation is far from complete at this point.
5. Kube-scheduler keeps watching the ApiServer and detects that `A pod object was created but it has no node associated with it`.
6. Kube-scheduler looks at the Pod and available worker nodes, and reports back to ApiServer about where the Pod should be scheduled.
7. ApiServer updates the EtcD cluster with the node information for the Pod and instrcuts the kubelet about new Pod.
8. Kubelet starts creating the Pod.
9. Kubelet instructs the container engine to pull necessary images and spin up the container within the pod.
10. Kubelet updates the status back to ApiServer and ApiServer updates the data in the EtcD cluster.


# Kube Contoller Manager

- Controller is analogous to an office on the master ship.
- These offices(controllers) are always on the lookout for the status of the K8s.
- And it also keeps trying to rememdiate the situation and tries to keep cluster in sync with the required state as per in the EtcD cluster.
- E.g. the Node Controller, Monitors the nodes every 5s for the heartbeats. If none received in 40s, it marks the node unreachable. If still none received by 5mins it evicts the pods and schedules them to run elsewhere if possible. The values for monitors can be modified as needed.
- There are many controllers in a k8s environment e.g. Deployment, CronJob, Namespace, ReplicaSet, StatefulSet.
- All these controllers are part of the kube-controller manager and part of the installation. By Default all controllers are enabled, but you can select which controllers should be available.

# Kube-Scheduler

- This is the component that decides where a Pod should be scheduled or if a Pod was evicted/killed etc. where else it should be restarted and so on.
- When a new Pod is to be scheduled:
    1. Scheduler checks all the available nodes.
    2. It does a first pass filtering based on Limits/Requests/taints/Tolerations/Affinity/Selectors and so on.
    3. Once the filtering is done, it looks at remaining nodes and calculates a node score between 0 ad 10.
    4. Node Score is way of Kube-scheduler deciding to place the pod on one of the nodes and score is calculated such that after placing the pod, which node will still have many resources available. Of course there could be other ways to do that too. You can implement your own scheduler if you wanted to.

# Kubelet

- Kubelet registers itself with master.
- Kubelet is the captain on the container ship. They lead all activities on the ship. 
- They are sole point of contact on Container ship, load unload ship as per request from master ship(apiServer)
- Kubelet also keeps sending status back to master periodically.
- Kubelet is the only component that is not automatically deployed, even if you use Kubeadm.


# Kube Proxy

- Every pod can reach every other pod and that is enabled by Pod Network.
- Pod Network is a cluster spanning virtual network that enables inter-pod communications.
- There are many solutions available for this and k8s does not dictate which one you should use as long as it is CNI Compliant.
- Kube-proxy is a process that runs on each node.
- Kube-proxy looks for new services and everytime a new service is created, kube-proxy creates forwarding rules to forward the traffic from Servie IP to backend Pods.
- One way to achieve this is using IPTables.
