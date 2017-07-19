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
