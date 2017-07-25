# Kubernetes in Action

Everything in Kubernetes is either YAML or JSON meaning its configuration files (called *manifests*)

**Namespace**

A *Namespace* is simply a naming convention for a resources, also a method to partition/divide into logical groups like production, dev, staging or teams, customers, etc.

Create a yaml file named `dev-ns.yaml` with the below content
```
kind: "Namespace"
apiVersion: "v1"
metadata:
  name: "dev"
  labels:
    name: "dev"
```
Now create the *namespace* 
```
$ kubectl create -f dev-ns.yaml
namespace "dev" created
```
To list the created *namespace*
```
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    6d
dev           Active    2m
```

**Pod**

A *Pod* is a basic unit of kubernetes, is a collection of one or more containers. The containers in the *pod* uses same namespace, IP address,port space and will find and communicate each other via localhost.

Create a yaml file named `httpd.yaml` with the below content
```
apiVersion: v1
kind: Pod
metadata:
  name: httpd
spec:
  containers:
  - name: httpd-server
    image: httpd
    ports:
    - containerPort: 80
```
Now create the *pod*,
```
$ kubectl create -f httpd.yaml
pod "httpd" created
```
To list the created *pod*
```
$ kubectl get pods
NAME            READY     STATUS              RESTARTS   AGE
httpd           0/1       ContainerCreating   0          57s
```
The STATUS ContainerCreating means  it is pulling httpd image & runs the container`. Now re-run the command again
```
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
httpd     1/1       Running   0          1m
```
Great :+1: it is running now and is one minute old :open_mouth:
Now check the details of the *pod*
```
$ kubectl describe pod httpd
Name:		httpd
Namespace:	default
Node:		k8solo-01/192.168.64.2
Start Time:	Wed, 19 Jul 2017 22:21:18 +0200
Labels:		<none>
Status:		Running
IP:		10.244.32.12
Controllers:	<none>
Containers:
  httpd-server:
    Container ID:	docker://f01ed00c678959209bf347bb4572a0221c253015561ce22a8a41aaef6102a730
    Image:		httpd
    Image ID:		docker-pullable://httpd@sha256:1989458ded44c5cfdb4e5b4e37ac435937564d52450db79c972b1b59dda0c7db
    Port:		80/TCP
    State:		Running
      Started:		Wed, 19 Jul 2017 22:22:41 +0200
    Ready:		True
    Restart Count:	0
    Volume Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5ldjw (ro)
    Environment Variables:	<none>
Conditions:
  Type		Status
  Initialized 	True
  Ready 	True
  PodScheduled 	True
Volumes:
  default-token-5ldjw:
    Type:	Secret (a volume populated by a Secret)
    SecretName:	default-token-5ldjw
QoS Class:	BestEffort
Tolerations:	<none>
Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath			Type		Reason		Message
  ---------	--------	-----	----			-------------			--------	------		-------
  9m		9m		1	{kubelet k8solo-01}	spec.containers{httpd-server}	Normal		Pulling		pulling image "httpd"
  9m		9m		1	{default-scheduler }					Normal		Scheduled	Successfully assigned httpd to k8solo-01
  8m		8m		1	{kubelet k8solo-01}	spec.containers{httpd-server}	Normal		Pulled		Successfully pulled image "httpd"
  8m		8m		1	{kubelet k8solo-01}	spec.containers{httpd-server}	Normal		Created		Created container with docker id f01ed00c6789; Security:[seccomp=unconfined]
  8m		8m		1	{kubelet k8solo-01}	spec.containers{httpd-server}	Normal		Started		Started container with docker id f01ed00c6789
```
We can delete the *pod* using `kubectl delete pod httpd`

**Replication controllers**

*Replication controllers* manage the lifecycle of pods. They insure a specified number of specific pods are running at any given time. They do this by creating or deleting pods as required. For this reason, it's recommended you use a replication controller even if you are creating a single pod.

Create a yaml file named `httpdrc.yaml` with the below content
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-httpd
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd
        ports:
        - containerPort: 80
```
Now create the **rc** ,
```
$ kubectl create -f httpdrc.yaml
replicationcontrollers/my-httpd
```
Now that we have only one replicas as mentioned in the definition, we can scale pods by using the command,
```
$ kubectl scale --replicas=3 rc my-httpd
replicationcontroller "my-httpd" scaled
$ kubectl get rc
CONTROLLER  CONTAINER(S)  IMAGE(S)  SELECTOR   REPLICAS
my-httpd    httpd         httpd     app=httpd  3
```
The replication controller will now ensure that number of replicas will be run at all times.

**Services**

*Services* provide a single stable name and address for a set of pods. They act as basic load balancers.

Most pods are designed to be long-running, but once the single process dies, the pod dies with it. If it dies, the replication controller replaces it with a new pod. But every time pod is started by the replication controller, the pod gets a new IP address.
A service is attached to a replication controller. Each service gets assigned a virtual IP address, which remains constant. As long as we know the service IP address, the service itself will keep track of the pods created by the replication controller, and will distribute requests to them.
Create a yaml file named `httpdsvc.yaml` with the below content
```
apiVersion: v1
kind: Service
metadata:
  name: httpdsvc
  labels:
    app: httpd
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: httpd
```
Now create the *service*, `$ kubectl create -f httpdsvc.yaml`
```
$ kubectl get svc httpdsvc
NAME      LABELS     SELECTOR   IP(S)          PORT(S)
httpdsvc  app=httpd  app=httpd  10.100.168.74  80/TCP
```
Because every node in a Kubernetes cluster runs a kube-proxy, the kube-proxy watches the Kubernetes API server for the addition and removal of services. For each new service, kube-proxy opens a (randomly chosen) port on the local node. Any connections made to that port are proxied to one of the corresponding backend pods.

**Labels**
Labels are used to organize and select groups of objects based on key-value pairs. They are used by every Kubernetes component. For example: the replication controller uses them for service discovery.
Let’s see how labels are used to connect the replication controller `my-httpd` with the `httpdsvc` service. Going back to our rc `my-httpd` config,
```
...
 metadata:
   labels:
     app: httpd
...
```
The `app` label is set to `httpd` and this label is used in our service config `httpdsvc`,
```
...
  selector:
    app: httpd
...
```
This label then functions to connect the service to the replication controller. You can see which selector a service is using like so:
```
$ kubectl get svc nginxsvc
NAME      LABELS     SELECTOR   IP(S)          PORT(S)
httpdsvc  app=httpd  app=httpd  10.100.168.74  80/TCP
```
