# CKA Notes from Udemy Course

- Kubernetes comes from Greek for Helmsman, pilot of a Ship
- Kubernetes agents convert the YAML to JSON prior to persistence to the database. 

## ETCD - Commands

Additional information about ETCDCTL Utility

ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3.  By default its set to use Version 2. Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:

    etcdctl backup
    etcdctl cluster-health
    etcdctl mk
    etcdctl mkdir
    etcdctl set


Whereas the commands are different in version 3

    etcdctl snapshot save 
    etcdctl endpoint health
    etcdctl get
    etcdctl put


To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3


When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.


Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

    --cacert /etc/kubernetes/pki/etcd/ca.crt     
    --cert /etc/kubernetes/pki/etcd/server.crt     
    --key /etc/kubernetes/pki/etcd/server.key


So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:


    kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key" 

### Backup ETCD

```sh

ETCDCTL_API=3 etcdctl snapshot save  --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/backup/

```


### Restore ETCD
```sh
ETCDCTL_API=3 etcdctl snapshot restore --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 --data-dir="/var/lib/etcd-from-backup" --initial-cluster="master=https://127.0.0.1:2380" --name="master" --initial-advertise-peer-urls="https://127.0.0.1:2380" --initial-cluster-token="etcd-cluster-1" /opt/backup
```
Then make update to static ETCD Pod:

- Update the `--initial-cluster-token`, `--data-dir` in the command section of the container for etcd pod definition file
- Update the `mountPath` for `volumeMounts` to point to `--data-dir`
- Update the `volumes` to mount correct `--data-dir`

## Create a CSR

```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```
## Approve CSR

```kubectl certificate approve <csr-name>```

## Creating role and rolebinding

```kubectl create role <name> --resource=<pods|svc|...> --verb=create,list,get,update,delete --namespace=<name>```

When binding the role, `--user` gets created implicitly.
```kubectl create rolebinding <name> --role=<role_name> --user=<username> --namespace=<name>```



## Verify access as per role/rolebinding

```kubectl auth can-i <verb> <resource> [--as=user] [--namespace=name]```

e.g. 
```
kubectl auth can-i update pods --as=john --namespace=default
kubectl auth can-i delete pod/blue-pod -n blue --as dev-user # this can be used when the `resourceNames` field is set in the role to allow access to specific resources.
```


## Running command from a pod and then removing it e.g. Nslookup

```kubectl run test-nslookup --image=busybox:1.28 --rm -it -- nslookup nginx-resolver-service```

```kubectl run test-nslookup --image=busybox:1.28 --rm -it -- nslookup 10-x-x-x.default.pod```


## Get list of all available api-versions

```kubectl api-versions```
This comes handy when writing `apiVersion` fields for your yaml objects.


## HostPath Persistent Volume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: /pv/data-analytics
```

Creating volume at pod level directly to use hostPath:
```yaml

volumes:
  - name: my-vol
    hostPath:
      path: /path
      type: Directory

volumeMounts:
  - mountPath: /mountPath
    name: my-vol
```


Creating a PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi

```

## Using PersistentVolume in a pod

```yaml
volumeMounts:
- mountPath: /path
  name: my-vol
volumes:
- name: my-vol
  persistentVolumeClaim:
    claimName: my-pvc 
```

## Review NetworkPolicies

- Kube-router, Calico, Romana, Weave-Net, ovn-k8s supports network policies.
- Flannel does not support NetworkPolicies.
- In NetworkPolicies, we have `podSelector` to select pod to which we want to apply the policies to and then `policyTypes` set to either one or both of `Ingress, Egress`.
- By default K8s cluster is set to `allow-all` to allow traffic from any nodes/pod/service to any other. NetPol allows to add security to that.
- If a pod has NetPol set on it, it will reject communication from everywhere else except what NetPol is set to allow.

e.g.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector: # select pods to apply this policy to
    matchLabels:
      role: db
  policyTypes: # types
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector: # select which namespace we will allow the traffic from
        matchLabels:
          project: myproject
    - podSelector: # and which pods within that namespace
        matchLabels:
          role: frontend
    ports: # and on which ports
    - protocol: TCP
      port: 6379
  egress:
  - to: # to where our pods can send communication, on what networks.
    - ipBlock:
        cidr: 10.0.0.0/24
    ports: # and on what ports
    - protocol: TCP
      port: 5978
```


## Edit Pod Images

It is possible to Edit the image a pod is running with.

You can just do `kubectl edit pod` and change the image name.

## Replicaset Rollout

Since Replicaset doesn't have same features as Deployments, to roll out a new set of pods, you need to manually delete existing pods so that Replicaset can give you new pods with new changes.


## DNS Format for Pod and Services

- `pod-ip-in-dash-notation.namespace.pod.domain` , so an example would be `my-pod.default.pod.cluster.local`
- `service-name.namespace.svc.domain` , so an example would be `my-svc.default.svc.cluster.local`


## Exposing a service using pod  in imperative way

```kubectl expose pod redis --port=6379 --name redis-service```


## Creating a pod and exposing container pod in imperative way


``` kubectl run custom-nginx --image=nginx --port=8080```

## Scheduling pods on a specific node by name

Add following to the pod spec:
```
nodeName: foo-node ## schedule pod to specific node

```


## Adding and Removing  Taints

Add Taint:
```
kubectl taint node node_name key=value:effect
```
e.g.
```
kubectl tain node node01 spray=mortein:NoSchedule
```


Remove Taint:
```
kubectl taint node node_name key=value:effect-
```
Observe the sign `-` at the end.


## Adding Toleration to a Pod


In the spec, add a List of Dictionary:

```yaml
tolerations:
  - key: key1
    value: value1 ## optional if using Exists operator
    effect: effect1
    operator: Equal/Exists
    tolerationSeconds: ## optional
```

## NodeSelectors

This is simpler way to select nodes for pods. NodeAffinities are further complex.

You can label a `NodeX` with `key=value` and then in your pod definition file you can add following:

```yaml

nodeSelector:
    key: value

```

## Adding NodeAffinity to Pod/Deployment/Replicaset

To add NodeAffinity use following syntax under `spec:` section:
```yaml
affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
```
Operators: `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`. You can use NotIn and DoesNotExist to achieve node anti-affinity behavior, or use node taints to repel pods from specific nodes.

If you specify both nodeSelector and nodeAffinity, both must be satisfied for the pod to be scheduled onto a candidate node.


## staticPodPath

Static pods are usually kept in `/etc/kubernetes/manifests/`, although to confirm that, you should check the status of kubelet.service and find config file, which should have the `staticPodPath` variable set. Find the file path in `--config` parameter in the `systemctl status kubelet.service`


## CustomScheduler

To add a new scheduler, you can add it as a staticPath, but you need to add a different value on the `--port` property and `--schedulerName=<name>` so that you can have multiple schedulers.


## To View the logs 

```
kubectl logs <object>
```

## To view monitoring data

```
kubectl top node[|pod]
```

## Updating image for a deployment

```
kubectl set image deployment/frontend www=image:v2
```
where `www` is the container name and `image:v2` is image name.



## Commands and Args in the Pods

Following way we can add Commands and Args in each Container in the pod as needed:
```yaml
command: ["printenv"]
args: ["HOSTNAME", "KUBERNETES_PORT"]
```


Using Env Var and use it in command:
```yaml
env:
- name: MESSAGE
  value: "hello world"
command: ["/bin/echo"]
args: ["$(MESSAGE)"]
```


Using ConfigMap as EnvVar:

```yaml


    env:
        # Define the environment variable
        - name: ENVVAR_NAME
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the `value` you want to assign to ENVVAR_NAME
              name: ConfigMapName
              # Specify the key associated with the `value`
              key: KeyFromConfigMap
```

Using Secret as EnvVars:

```yaml

env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
            name: mysecret
            key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
            name: mysecret
            key: password


```

Entire Secret can also be used using `envFrom` as :

```yaml
envFrom:
      - secretRef:
          name: mysecret

```


## InitContainers

sample snippet:
```yaml

spec:
  initContainers:
    - image:
      name:
      command:
    - image:
      name:
      command:

```


## Drain, Cordon, Uncordon

To drain a node, and mark is unschedulable, you can use:

`kubectl drain node_name`, you may use `--force` or `--ignore-daemonsets` as additional options.
Any pods that are not part of a ReplicaSet, StatefulSet, will be lost forever when draining a node with `--force`.


To mark a node UnSchedulable, but not drain it, use:

`kubectl cordon node_name` - this will prevent any new nodes to be scheduled on this node.

To mark a node Schedulable, use:

`kubectl uncordon node_name` - this will allow any new nodes to be scheduled on this node.



## Cluster Upgrade Process

Ref: https://v1-19.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

When upgrading k8s cluster, always go 1 version upgrade at a time. K8s community supports 3 minor versions at any point, e.g. 1.11,1.12,1.13, when 1.14 releases 1.11 support is dropped. So we need to plan our upgrades accordingly.


For the components in the k8s control plane use following strategy:

Api server can't be behind any other components.

Other components such as controller-manager and kube-scheduler can only be 1 version behind Apiserver. Kubelet and kube-proxy is allowed to be upto x-2 versions behind the apiserver. This allows us to perform live upgrades.


Kubectl command itself can be behind or ahead by +1/-1 compared to kube version.


When we list the nodes, the versions listed are the versions of kubelet and not of the other components in the cluster . 

Always Upgrade your `master` node followed by `worker` nodes.

### Start Upgrade Process

- Check Cluster Version:
```
kubeadm upgrade plan
```
This will dump info of all available versions. It is applicable only when you installed cluster using `kubeadm` tool. If you set it up `the hard way` then you will have to check and update the versions of the each components separately.

- Upgrade the `kubeadm` tool using `yum` or `apt` etc. package manager.
```
apt install kubeadm=1.19.0-00
```
### Master Upgrade
- Apply upgrade:
```
kubeadm upgrade apply v1.19.0
```

- Upgrade Kubelet and Kubectl:
```
apt install kubelet=1.19.0-00 kubectl=1.19.0-00
```

- Reload daemon and restart kubelet:
```
systemctl daemon-reload
systemctl restart kubelet
```

- Mark master ready to schedule again:
```
kubectl uncordon master
```

### Worker upgrade

- Drain worker node:
```
kubectl drain nodeX --ignore-daemonsets
```

- ssh to respective node & Upgrade kubeadm:
```
apt install kubeadm=1.19.0-00
```

- Upgrade node:
```
sudo kubeadm upgrade node
```

- Upgrade kubelet and kubectl: 
```
apt install kubelet=1.19.0-00 kubectl=1.19.0-00
```

- Reload daemon and restart kubelet:
```
systemctl daemon-reload
systemctl restart kubelet
```

- Exit SSH and Mark node ready to schedule again:
```
kubectl uncordon nodeX
```



## Certificates

- If you want to see details of a certificate:
```
openssl x509 -in /cert/file/path.crt -noout -text
```


### Allowing new Users to gain access to the K8s env:

- Create a new key:
```
openssl genrsa -out username.key 2048
```
- Create a Cert Signing Request(CSR):
```
openssl req -new -key username.key -subj "/CN=username" -out username.csr
```
- Send the CSR to the Admin
- Admin takes the CSR file and creates a CSR Objct:

```yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
    name: username
spec:
    groups:
    - system:authenticated
    usages:
    - server auth
    - digital signature
    - ....
    request: output from `cat username.csr|base64 -w 0` # -w 0 removes all \n and outputs singleline only

```

- Create, view and approve CSR:
```sh

kubectl create -f csr.yml
kubectl get csr
kubectl certificate approve username

```

- After certificate is approved, you can get that from the yaml:
```
kubectl get csr -o yaml username
```
- This will show you certificate in yaml format with bas64 encoding, extract it and decode it as:
```
echo "nfntwpfe" | base64 --decode
```

- Once done, share the output from above command and share it with the user.




## Kubeconfig file

In order to authenticate any request, kube-apiserver needs, client key, certificate and CA certifcate. In order to use kubectl command
or to curl k8s api server, you need those 3 things as well as the server address.

It is not great idea to type those things every single request you make. Hence KubeConfig files contain:

```
- server url
- client key
- client cert
- ca cert

```

Default location `$HOME/.kube/config`

- To check if a kube-config is valid, try using it as:
```
kubectl cluster-info --kubeconfig=<path>
```

- Kubeconfig file has 3 sections:
    - Users
    - Clusters
    - Contexts - which marries users+clusters together

- Yaml format:

```yaml
apiVersion: v1
kind: Config

clusters:
- name: cluster_name
  cluster:
    certificate-authority: ca.crt
    server: https://my-k8s.io:6443
contexts:
- name: my-k8s-admin@cluster_name
  context:
    cluster: cluster_name
    user: my-k8s-admin
users:
- name: my-k8s-admin
  user:
    client-certificate: admin.crt
    client-key: admin.key
```

- To view current kubeconfig file being used, run `kubectl config view`
- To select and set a context, run `kubectl config use-context` with proper `--kubeconfig` file if using other than `$HOME/.kube/config`


# Namespace resources vs Non-Namespace Resources

Resources that are namespace scoped can be seen by:

`kubectl api-resources --namespaced=true`

Else following gives you cluster scoped resources:
`kubectl api-resources --namespaced=false` 

**Cluster Roles can be created for namesapced objects such as pods, in that case it will grant the access to the all pods clusterwide**


## SecurityContexts

Security Context can be added at the Pod or Container level or at both.

e.g.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```

Although it is VERY IMPORTANT to NOTE that `CAPABILITIES` can only be added to container level securityContext. 


## Ingress - Annotations and rewrite-target

Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen here. I would like to explain one such option that we will use in our labs. The Rewrite target option.


Our watch app displays the video streaming webpage at http://<watch-service>:<port>/

Our wear app displays the apparel webpage at http://<wear-service>:<port>/

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/


Without the rewrite-target option, this is what would happen:

http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/watch

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/wear


Notice watch and wear at the end of the target URLs. The target applications are not configured with /watch or /wear paths. They are different applications built specifically for their purpose, so they don't expect /watch or /wear in the URLs. And as such the requests would fail and throw a 404 not found error.


To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under rules->http->paths->path which happens to be /pay in this case with the value in rewrite-target. This works just like a search and replace function.

For example: replace(path, rewrite-target)
In our case: replace("/path","/")
```yaml

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: test-ingress
      namespace: critical-space
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
          paths:
          - path: /pay
            backend:
              serviceName: pay-service
              servicePort: 8282

```
In another example given here, this could also be:

replace("/something(/|$)(.*)", "/$2")
```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$2
      name: rewrite
      namespace: default
    spec:
      rules:
      - host: rewrite.bar.com
        http:
          paths:
          - backend:
              serviceName: http-svc
              servicePort: 80
            path: /something(/|$)(.*)
```


## JSON Path in Kubectl

JSON Path is a query language for JSON objects.

Kubectl can give `-o json`, **kube-apiserver speaks json** , converts it to human-readable format and shows only necessary details.

`-o wide` prints some more details, but most other info is hidden in human-friendly format.

To see custom info, you need JSON Path, filter and format, as needed.

E.g.
```
kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'
```
Notice you do not need to prefix a `$` sign, k8s adds it implicitly, but you need a pair of `{}` and `''` to surround your jsonpath query, as `'{query}'`

You can also use `http://jsonpath.com/` to play with the queries.

You can also have multiple queries at one time as:
```
kubectl get nodes -o=jsonpath='{query1}{"\n"}{query2}' # optionally you can add \n or \t as shown
```


To use `loops`:

```
kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
```
Above will print as:
```
node1 <cpuCount>
node2 <cpuCount>
node3 <cpuCount>
```

`Custom Columns`, easier method than `loops`:

```
kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu --sort-by=.status.capacity.cpu
```

Notice the `--sort-by` does not require `'{}'` or `.items[*]` as it operates on the all items.

## Troubleshooting Tips

- Check all components in the control-plane (namespace kube-system) are up and running
- Check if all services in the control plane have valid endpoints, ports etc.
- Check the logs of the control plane components if anything is failing, or if no new pods are being scheduled, no deployments could be scaled or replicasets can be scaled, pods are pending(kube-scheduler may be broken).
- If the componenet is a static pod look into /etc/kubernetes/manifests/ and review its configuration
- If the component is not a static pod, look at the daemonset or replicasets, deployments, that deploys the component.
- For networking, make sure there is a networking solution in place in the control plane.
- For nodes, make sure the docker and kubelet services are running.
- For the logs of the failing services use `systemctl status` or `journalctl -u <unitname> -f`
- For each service that is being referenced by a Pod/Deployment, make sure the name is correct.
- Check environment variables
- Check if Weave or some plugin is configured on the cluster.
- Kube-proxy uses 2 configs from the configmap, one is `--conf` pointing to `config.conf`(mounted from ConfigMap) and other is `--kubeconfig` also mounted fro the `kube-proxy` configmap.
- Kube-dns itself has a service in kube-system namespace and if that is broken, kube-dns can't function properly. It needs 53/UDP, 53/TCP, 9153/TCP ports exposed as ClusterIP service.