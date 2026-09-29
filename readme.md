# Introduction to Kubernetes

Kubernetes, often abbreviated to K8S (K & S, with 8 characters in between), is a container orchestration platform.

If you create a container using Docker (the most popular tool for creating and running containers) you’re presented with the running container, which is hopefully running your app. But you can also interact directly with the container by accessing it's Shell, then issue commands as you would with a normal Linux computer.

With Kubernetes we don’t interact directly with containers often, particularly since we may have thousands of them, so they’re abstracted away by logical layers and objects that we do interact with.

## Nodes and Pods

K8S has lots of objects that you need to be aware of, the first two are:

### Nodes

These are the servers or VMs, on which your pods run. Most deployments will have at least two nodes, one hosting your Pods, and one for your `Control Plane`.

### Pods

The building blocks and smallest deployable object in k8s:

- K8S creates a virtual network, and pods are given an IP on that network.
- Pods are ephemeral (temporary) and receive a new IP upon re-creation

Every running container is in a pod, it’s the environment for one or more containers, but when there is more than one they’re tightly coupled & share resources.

![*DIAGRAM*](pics/nodes-pods.jpg)

Each pod is assigned a unique IP, and the containers within share the network namespace, IP, & ports, and the pod can also specify shared storage volumes.

Containers within a pod may communicate via loopback.

## Kubernetes Concepts

There are two things two first appreciate about how Kubernetes (k8s) works; Firstly it uses an **object model**, i.e. everything is represented by objects. We don’t manage the item itself, we manage the object that represents that item.

The next point is that k8s uses **declarative management** which simply means that you don’t make statements about *what you want to do*, instead you declare *how things should be*.

Consider the difference between

>deploy a web server and a database

vs.

>there should be a web server and a database.

If you run the first statement multiple times you may end up with multiple servers and DB’s, whereas the second one will verify the existing items before taking action.

With kubernetes (and many other technologies) you declare your desired state, how things should be, and the actual steps to make that happen are abstracted away from you, and automated as much as possible.

## Kubernetes Objects

Each item K8S manages is represented by an object, there are many different types or ‘kinds’, but don’t worry about memorising them.

![*DIAGRAM*](https://miro.medium.com/v2/resize:fit:1400/1*viLY3qG94sI3p4dqCGeEwA.png)

Objects have two important elements:

- **Spec** = the desired state of the object
- **Status** = the current state of the object

In brief, Kubernetes’ job is to make these match.

Objects can describe:

- What containerised applications are running (and on which nodes)
- The resources available to those applications
- The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

To work with Kubernetes objects—whether to create, modify, or delete them—you'll need to use the Kubernetes API via the kubectl command-line interface

## Control Plane

The Control Plane is the collection of cooperating processes that make the cluster work, although typically you may only interact with a few of them.

A cluster will typically run across multiple servers, with one being the control plane controlling and coordinating the cluster, and the others are nodes running pods.

>There is a version of K8S which creates a local node containing the control plane and pods all together in a VM called Minikube. Used primarily for learning and development, we’ve created a Linux VM with it installed for you.

Let’s consider some of these points in a scenario to give context.

You want 3x nginx web servers running, each in their own container:

1. You need to declare some object to represent the containers, i.e. some pods. - This declaration is simply a text file known as a manifest.
1. When your deployment is new, k8s will look at your desired state of 3 containers, and the running state of 0, and look to remedy the mismatch. Specifically, the control plane will launch 3x containers.
1. It will then continuously monitor the cluster to maintain the desired state.

### Components on the Control Plane

Several critical components run on the control plane

#### Kube API server

The only component you interact with directly; The API server accepts cmds to view or change the state of the cluster, such as launching new pods. The API server also authenticates and authorises incoming requests.

You interact with the server using `kubectl` commands which utilise the Kubernetes API.

Any query or change to the cluster is addressed to the Kube API server

- **Etcd** - The cluster’s DB, stores the state of the cluster and additional information such as member pods, pod locations (nodes), etc. *Only the API server interacts with Etcd.*

- **Kube Proxy** - maintains connectivity between pods in the cluster.

- **Kube Scheduler** - schedules pods onto the nodes by evaluating the requirements of each pod and selecting the most suitable node. Kube Scheduler doesn’t launch the pods, but once a node is selected it writes the node name to the relevant pod object.

    Kube Scheduler knows the state of all pods, and accounts for any constraints you define, e.g. specifying that certain pods require a node with a minimum h/w spec’ and other policies or restrictions.

    You may also choose **affinity** or **anti-affinity** parameters, such as stating that specific pods may or may not run on the same node.

- **Kube Controller Manager** - Continuously monitors the state of the cluster through the API server; When the current and desired states don’t match, KCM will attempt to make changes to remedy.

    Called the Kube Controller Manager, because many components are maintained by code loops we call 'controllers'. For example, the **Node Controller** monitors and responds to changes in node states.

    We could also create **controller objects** to represent and manage workloads, such as the three nginx containers from the earlier scenario.

- **Kube Cloud Manager** - like KCM, is used to manage controllers, but in this case it’s the controllers which interact with an underlying cloud provider.

- **Kubelet** - A few Control Plane components run on each node, in a module called a Kubelet.

    When API Server needs to start a pod on a node it connects to the pod’s Kubelet.

    Kubelet uses the ‘container runtime’ on the node (similar to a VM’s hypervisor) to launch the pod, monitor it, probe it for readiness, and reports back to the Kube API Server.

### Container Runtimes

Container runtimes are software packages that enable you to launch containers on a host operating system; Containerd is the comtainer runtime which powers Docker and Kubernetes. Containerd is lightweight, reliable, and manages the entire container lifecycle from image transfer and storage to execution and supervision.

### Kubernetes Architecture

Here's a slightly more details architecture diagram showing how everything we've reviewed so far fits together.

![*DIAGRAM*](pics/k8s-architecture.jpg)

- Notice all developer requests are directed to the Kube API Server
- When it receives a request the API Server decides which of the Control Plane' components need to be engaged to complete the job.
- The relevant component completes the request by communicating with the pod(s) via it's Kubelet.

### Object Management

To instruct K8S to create & maintain some objects you can write manifest files in either JSON or YAML*. These are simply text files defining the state of the object(s), it’s name, and usage. Required fields include:

- Kubernetes API version
- The Kind - the type of object e.g. a service, a deployment, etc.
- metadata including:
  - name & unique ID
  - namespace (optional)
  - spec - your desired state

The object’s name should be unique in the workspace, and it will receive a unique ID generated by K8S.

Labels (KVPs) can be attached to objects to help identify and organise them. The `kubectl` command allows you to select and omit objects you wish to manage based on labels.

*[Click here for a comparison of JSON and YAML](/json-vs-yaml.md)

- The object’s name should be unique in the workspace, and it will receive a unique ID generated by k8s.
- Labels (KVPs) can be attached to objects to help identify and organise them. The kubectl command allows you to select and omit objects you wish to manage based on labels.
- If you require several related objects for a deployment, best practice is to define them in the same manifest file for easier management.

It is likely that over time your deployments will evolve as you improve, refine, and optimise them. For this reason it is recommended that you store them in a version control repository (e.g. GitHub) for easy tracking and management, which also makes it easier to rollback changes when necessary, as well as recreating or restoring your clusters.

### Deployment Controller

Deployments are good for long-running components, like web servers, and also facilitates managing them as a group.

If we declare our 3x nginx pods as a Deployment YAML file, the **Deployment Controller Object** will monitor and maintain the required pods.

When Kube Scheduler schedules pods for deployment it notifies the Kube API server. The Deployment Controller creates a child-object called a **ReplicaSet** which launches the desired pods and maintains a stable set of replicas.

If a pod fails the **ReplicaSet Controller** recognises the difference between the desired and current state, then launches new pods to rectify.

### Deployments

Within a deployment object specify:

- The number of replica pods
- Which containers should run in the pods
- Which volumes should be mounted
- These templates are then used by controllers to maintain the desired state in the cluster.

The below is an [example](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) of a Deployment which creates a ReplicaSet to bring up three nginx Pods:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

## Abstraction layers

Kubernetes has many different abstraction layers, designed to simplify management of complex environments, but they can get confusing themselves. Let’s try to lay them out…

1. We don’t interact with containers directly, that’s abstracted away by pods… but neither do we interact with pods directly, they’re abstracted away by nodes.
1. We also don’t send commands to nodes, instead we communicate with Kubelet which manages the pods on each node, but we don’t communicate with kublet directly.
1. We want our apps to be highly available, so the objects are monitored by controllers.
1. We don’t need to know commands for kubelet, controllers, and kube proxy. They’re all abstracted away through the Kubernetes API and we make calls to the Kube API server.
1. We don’t want to write complex API templates for the Kube API server every time, so that is abstracted away by another layer through the … wait for it … the `kubectl` command.

![DIAGRAM](pics/abstraction-layers.jpg)

## Kubectl

Kubectl is a command line utility (note: not a command line interface, but a utility accessed via a command line) used to communicate with the Kube API server on the control plane by transforming CLI commands into properly formatted API calls.

Administrators make requests to the cluster, and kubectl determines which part of the control plane to communicate with.

View a list of running pods in a cluster with `kubectl get pods`, when you do so:

1. Kubectl converts the cmd into an API call which is sent to the Kube API server on the cluster’s control plane using HTTPS.
1. The API svr processes the request by querying etcd, then returns the result back to kubectl, again over HTTPS.
1. Kubectl interprets and displays the response to the user through the CLI.

Before use kubectl must be configured with the location and credentials of the K8S clusters it will be managing, these are stored in `.kube` in the user’s home directory.

For clusters created on one of the main public cloud providers, each provider offers a CLI command to retrieve these credentials.

|Provider|Get Cluster Credentials command|
|---|---|
|AWS|`aws eks update-kubeconfig --name <cluster-name>`|
|GCP|`gcloud container clusters get-credentials <cluster-name>`|
|Azure|`az aks get-credentials --name <cluster-name> --resource-group <rg-name>`|

If you're manually creating/deploying a cluster you can use the equivalent Bash commands:

```sh
kubectl config set-credentials my-user --client-certificate=path/to/cert.crt --client-key=path/to/key.key
kubectl config set-cluster my-cluster --server=https://<api-server> --certificate-authority=path/to/ca.crt
kubectl config set-context my-context --cluster=my-cluster --user=my-user
kubectl config use-context my-context
```

Notice the extra complexity when deploying manually, compared to using a cloud provider's managed service.

|Cloud Provider|Managed Kubernetes Service|
|---|---|
|Azure|Azure Kubernetes Service|
|AWS|Google Kubernetes Engine|
|GCP|Elastic Kubernetes Service|

The `.kube` file will be updated each time the command is run against a new cluster.

Once correctly configured kubectl references this file and connects to the default cluster without prompting for credentials each time.

### Kubectl command structure

Kubectl uses a similar command structure and logic to Linux:

![DIAGRAM](pics/kubectl-structure.jpg)

## Minikube

K8S is a modern, complex tool, which is difficult to get your head around just with theory. Deploying a cluster in the cloud just to learn or develop with could be expensive, so instead you can use minikube.

Minikube is a version of K8S which deploys a control plane and worker node on a single VM. You can install it on Windows if you wish, but it can be a bit tricky, so since you know Linux, best to stick with it.

## Install MiniKube on CentOS Stream 9

There are a few options to configure for the VM in VirtualBox to make it work in VirtualBox:

- Ensure that `Nested VT-x/AMD-V` is enabled:
  1. Right click the VM's tab in VirtualBox (usually called `CentOS-1.1.0`) and choose `Settings...`
  2. On the left of the Settings window click `System`, then change the `Base Memory` to 4GB (`4096MB`).
  3. In order to access your deployed app(s) from the host configure the following: `Network` > `Attached to:` > `Bridged Adapter`
- If you intend to try some more advanced deployments you might also want to give the VM 4x CPU cores to improve performance. .You can do so by clicking the `Processor` tab on the same settings window.

### Install Required Dependencies

```bash
sudo dnf install -y curl wget conntrack
```

### Install Docker

```sh
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io
sudo systemctl enable --now docker
```

### Install Minikube

```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Add Your User Account to the Docker Group

```sh
sudo usermod -aG docker $USER && newgrp docker
```

### Start Minikube

```sh
minikube start --driver=docker
```

### Verify Installation

```sh
minikube status
```

### Install Kubectl

```sh
curl -LO https://dl.k8s.io/release/v1.33.2/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
echo 'export PATH=/usr/local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
kubectl version --client
```

## Trying it out

Once Minikube status shows your cluster is running with an output like the following:

>[centos@localhost ~]$ minikube status  
minikube  
type: Control Plane  
host: Running  
kubelet: Running  
apiserver: Running  
kubeconfig: Configured  

...then you're ready to try experimenting.

Try the following commands to begin with...

```sh
kubectl get pods
kubectl get nodes
kubectl describe nodes
kubectl create deployment nginx-depl --image=nginx
kubectl get deployment
kubectl get pod
kubectl get replicaset
kubectl edit deployment nginx-depl # edit with vi to add replicas
kubectl get pod
kubectl get deployment
kubectl delete deployment [name]
```

If everything is working, proceed through the following Lab.

## Kubernetes Practical Lab

### Objective

Deploy and manage a simple NGINX web application using Minikube.

By the end of this lab you will be able to:

- Verify a Kubernetes cluster
- Create a Deployment
- View Pods and logs
- Create a Service
- Access a containerised web application
- Scale an application
- Observe self-healing behaviour
- Perform rolling updates
- Roll back deployments
- Create resources using YAML manifests

---

### Verify the Cluster

Check Minikube is running:

```sh
minikube status
```

Expected output:

```sh
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Check the node:

```sh
kubectl get nodes
```

Expected output:

```sh
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   55m   v1.37.0
```

### View Existing Workloads

List all pods:

```sh
kubectl get pods -A
```

- `-A` means all namespaces.
- Review the system pods created by Kubernetes.

### Create a Deployment

Deploy an NGINX web server:

```sh
kubectl create deployment webserver --image=nginx
```

Expected output:

```sh
deployment.apps/webserver created
```

Verify:

```sh
kubectl get deployments
```

Expected output:

```sh
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
webserver   1/1     1            1           53s
```

View pods:

```sh
kubectl get pods
```

Expected output:

```sh
NAME                         READY   STATUS    RESTARTS   AGE
webserver-85f8d869b5-sqxcw   1/1     Running   0          76s
```

### Inspect the Deployment

```sh
kubectl describe deployment webserver
```

Questions:

- How many replicas exist?
- Which image is deployed?
- Which labels have been assigned?

<details><summary>Answers:</summary>

- 1
- nginx
- app=webserver

</details>

#### Inspect the Pod

```sh
kubectl describe pod [POD_NAME]
```

#### View Container Logs

```sh
kubectl logs [POD_NAME]
```

>Even though NGINX may not show much output, this is a useful command for troubleshooting applications.

### Create a Service

A Pod's IP address may change if it is recreated.

Services provide a stable endpoint for applications.

Create a NodePort Service:

```sh
kubectl expose deployment webserver \
  --port=80 \
  --type=NodePort
```

Expected output:

```sh
service/webserver exposed
```

View the service:

```sh
kubectl get svc
```

Expected output:

```sh
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        68m
webserver    NodePort    10.109.141.191   <none>        80:31322/TCP   37s
```

Retrieve the URL:

```sh
minikube service webserver --url
```

Example output:

```sh
http://192.168.49.2:31322
```

Test it:

```sh
curl $(minikube service webserver --url)
```

>The address returned by: `minikube service webserver --url` belongs to the Minikube node network, not the VM itself.
>
>This address cannot normally be reached from the host computer.
>
>To make the site accessible from a browser on the host machine we will create a port forwarding tunnel.

**Open a Second Terminal, keep your original terminal open.**

|Window|Purpose|
|---|---|
|Terminal 1|Normal Kubernetes commands|
|Terminal 2|Port forwarding process|

The port forwarding process must remain running while you access the website.

### Create a Port Forward

In **Terminal 2** run:

```sh
kubectl port-forward deployment/webserver \
8080:80 \
--address 0.0.0.0
```

Expected output:

```sh
Forwarding from 0.0.0.0:8080 -> 80
```

Leave this terminal running.

### Allow Traffic Through the Firewall

In **Terminal 1**:

Check whether port 8080 is allowed:

```sh
sudo firewall-cmd --query-port=8080/tcp
```

Expected:

```sh
no
```

allow it:

```sh
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

Verify:

```sh
sudo firewall-cmd --query-port=8080/tcp # now returns 'yes'
sudo firewall-cmd --list-ports # lists all permitted ports
```

### Access the Website

Identify the VM IP address:

```sh
ip a

# or

hostname -I
```

From your host computer open:

```sh
http://[VM_IP_ADDRESS]:8080
```

You should see `Welcome to nginx!`

### Scale the Application

Increase replica count:

```sh
kubectl scale deployment/webserver \
--replicas=3
```

Verify:

```sh
kubectl get deployments
```

List Pods:

```sh
kubectl get pods
```

Expected:

```sh
3 Running Pods
```

### Observe Self-Healing

List Pods:

```sh
kubectl get pods
```

Delete one:

```sh
kubectl delete pod [POD_NAME]
```

Immediately watch:

```sh
kubectl get pods
```

Notice:

- One Pod is removed.
- Kubernetes automatically creates a replacement.

Press `Ctrl+C` to stop watching

### View the ReplicaSet

ReplicaSets maintain the correct number of Pods.

View them:

```sh
kubectl get rs
```

Questions:

- How many replicas are desired?
- How many are currently running?

<details><summary>Answers:</summary>

- 3
- 3

</details>

### Perform a Rolling Update

Update the image:

```sh
kubectl set image deployment/webserver \
nginx=nginx:latest
```

Monitor progress:

```sh
kubectl rollout status deployment/webserver
```

View revision history:

```sh
kubectl rollout history deployment/webserver
```

### Roll Back

Undo the update:

```sh
kubectl rollout undo deployment/webserver
```

Verify:

```sh
kubectl rollout history deployment/webserver
```

### Create a Manifest

Export the deployment:

```sh
kubectl get deployment webserver -o yaml > deployment.yaml
```

Inspect:

```sh
cat deployment.yaml
```

Identify:

- apiVersion
- kind
- metadata
- spec

These are present in nearly every Kubernetes manifest.

### Recreate Using YAML

Delete the deployment:

```sh
kubectl delete deployment webserver
```

Verify:

```sh
kubectl get deployments
```

Recreate it:

```sh
kubectl apply -f deployment.yaml
```

Verify:

```sh
kubectl get deployments
```

Delete everything created during the exercise:

```sh
kubectl delete svc webserver
kubectl delete deployment webserver
```

Verify:

```sh
kubectl get all
```

The output should show no remaining user-created resources.

## Key Concepts

- `Pod`: smallest deployable Kubernetes object
- `Deployment`: manages Pods
- `ReplicaSet`: maintains correct number of Pods
- `Service`: provides stable networking
- `Port Forwarding`: exposes workloads outside the cluster
- `YAML Manifests`: declarative definition of Kubernetes resources
- `Self-Healing`: failed Pods are automatically recreated
- `Rolling Updates`: applications can be updated without downtime
- `Rollback`: deployments can be reverted if a change causes problems
