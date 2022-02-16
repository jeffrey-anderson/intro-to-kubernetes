# Week 1: Introduction to Kubernetes

## Before we begin

* [minikube](https://minikube.sigs.k8s.io/docs/) website
* [minikube start](https://minikube.sigs.k8s.io/docs/start/)

```
minikube --help
minikube start
```

## Configuration

* [Organizing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
* [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

```
kubectl help
kubectl set -h

kubectl config view
kubectl config get-contexts
kubectl config current-context
kubectl config use-context minikube
```

![Kubernetes architecture](https://user-images.githubusercontent.com/46822968/150684180-8550b242-6bef-40f5-80d4-f4ae4fcc34b3.png)

Figure 11.1. Kubernetes components of the Control Plane and the worker nodes from Kubernetes in Action. 

See [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/) for a description of each component.

```
kubectl get componentstatuses
```
NOTE: [ Hard-coded addresses of scheduler and controller manager causes unhealthy ComponentStatus #96848](https://github.com/kubernetes/kubernetes/issues/96848)

```
kubectl get all -n kube-system
ps -ef | grep kubelet
kubectl get nodes
kubectl describe node minikube
```

## Minikube Dashboard

* [Dashboard](https://minikube.sigs.k8s.io/docs/handbook/dashboard/)
* [Deploy and Access the Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```
minikube dashboard
```

## API server basics

* [The Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
* [Access Clusters Using the Kubernetes API](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/)
```
kubectl api-resources
kubectl api-resources --namespaced=false
kubectl api-resources --api-group=apps
kubectl api-resources --api-group=storage.k8s.io
```


## Namespaces

* [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
* [Namespaces Walkthrough](https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/)

```
kubectl explain namespaces
kubectl get namespaces
kubectl create namespace jeffs-space
kubectl get namespaces
kubectl delete namespace jeffs-space
kubectl get namespaces
kubectl get all --all-namespaces
```

## Configuration files

* [Declarative Management of Kubernetes Objects Using Configuration Files](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
* [Understanding Kubernetes Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)

Explore:
* Create a namespace with [jeffs-ns.yaml](jeffs-ns.yaml)
  ```
  kubectl apply -f jeffs-ns.yaml
  kubectl get namespaces
  ```

* Query the namespaces via the API:
  1. In a different console window: `kubectl proxy --port=8080`
  2. In the main console window: `curl http://localhost:8080/api/v1/namespaces`

* To delete it using the config file:
  ```
  kubectl delete -f jeffs-ns.yaml
  kubectl get namespaces
  ```

## Contexts

* [Organizing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
* [Create pods in each namespace](https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/)
* [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

Explore:

* Get existing contexts:
  ```
  kubectl config get-contexts
  ```

* Update the current context to use a new namespace:
  ```
  kubectl create namespace week1
  kubectl config set-context --current --namespace=week1
  kubectl config get-contexts
  kubectl config set-context --current --namespace=default
  kubectl config get-contexts
  ```

* Create and delete a new context:
  ```
  kubectl create namespace hello-kube
  kubectl config set-context hello-minikube --cluster=minikube --namespace=hello-kube --user minikube
  kubectl config get-contexts
  kubectl config current-context
  kubectl config use-context hello-minikube
  kubectl config delete-context hello-minikube
  kubectl config current-context
  kubectl config get-contexts
  kubectl get nodes
  kubectl config use-context minikube
  kubectl config get-contexts
  kubectl get namespaces
  kubectl delete namespace hello-kube
  kubectl config set-context --current --namespace=week1
  kubectl config get-contexts
  ```

## Pods

* [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
* [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
* [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
* [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

```
kubectl explain pods
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
kubectl get pods
kubectl get pods -o wide
kubectl describe pods nginx
kubectl logs nginx
```

Explore:

* Run commands:
  ```
  kubectl exec nginx -- ls -l /
  kubectl exec nginx -- echo "Hi from the nginx pod!" 
  ```

* Shell into the nginx pod and look around:
  ```
  kubectl exec -it nginx -- /bin/bash
  ```

* To hit the nginx pod, we a shell inside the cluster:
  ```
  kubectl run -i --tty busybox --image=busybox --restart=Never -- sh 
  ```
  Run commands:
  ```
  wget -q -O - <IP address of nginx pod>
  exit
  ```

* Use port forwarding in a second terminal window:
  ```
  kubectl port-forward -h
  kubectl port-forward nginx 8080:80
  ```
* Visit http://localhost:8080/

* Now that we hit the nginx web server check the logs again:
  ```
  kubectl logs nginx
  ```

* Clean up:

  * Stop the port forwarding
  * Remove the objects:
    ```
    kubectl get all
    kubectl delete pod/nginx pod/busybox
    ```

## Storage 

* [Storage](https://kubernetes.io/docs/concepts/storage/) documentation
* [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/) documentation
* [Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

## Init containers and storage

* [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

We will use [init-container-pod.yaml](./init-container-pod.yaml) for this demo. Run and connect:
```
kubectl apply -f init-container-pod.yaml
```

Explore:
* Get information about our pod:
  ```
  kubectl get all
  kubectl describe pod/demo-webapp
  ```

* Use port forwarding in a second terminal window:
  ```
  kubectl port-forward demo-webapp 8080:80
  ```

* Access the application locally: `curl localhost:8080`

* Visit http://localhost:8080/ refreshing the page if you see the nginx banner from the previous demo.

* Clean up:

  * Stop the port forwarding
  * Remove the objects: `kubectl delete pod/demo-webapp`

## Sidecars

![England's 20-Year-Old 'Two Fat Ladies' is Still the Best Cooking Show Ever Made](https://www.saveur.com/uploads/2019/03/18/QLOE3NHGN7OC7GAKJH42J6KWTI.jpg?auto=webp)

Photo credit Saveur - [England's 20-Year-Old 'Two Fat Ladies' is Still the Best Cooking Show Ever Made](https://www.saveur.com/two-fat-ladies-best-food-tv-show/)

* [How Pods manage multiple containers](https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers)
* The [Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/) page has a good example of this pattern.

We will use [sidecar-container-pod.yaml](sidecar-container-pod.yaml) for this demo. Run and connect:
```
kubectl apply -f sidecar-container-pod.yaml
```

Explore:
* Get information about our pod: 
  ```
  kubectl describe pod/lottery-app
  ```

* Use port forwarding in a second terminal window:
  ```
  kubectl port-forward lottery-app 8080:80
  ```

* Access the application locally: `curl localhost:8080`

* Visit http://localhost:8080/ refreshing the page if you see the nginx banner from the previous demo.

* Clean up:

  * Stop the port forwarding
  * Remove the objects: `kubectl delete pod/lottery-app`


## For further reading 

* [The Distributed System ToolKit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)

## Labels

* [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
* [Recommended Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
* [Well-Known Labels, Annotations and Taints](https://kubernetes.io/docs/reference/labels-annotations-taints/)

Explore:
* View labels on existing objects:
  ```
  kubectl get nodes --show-labels
  ```
* Create a new container and examine the labels:
  ```
  kubectl apply -f sidecar-container-pod.yaml
  kubectl get all --show-labels
  ```
* Get help on labels:
  ```
  kubectl label -h
  ```
* Label our new pod:
  ```
  kubectl label pods lottery-app some-key=some-value
  kubectl get pods --show-labels
  ```
* Try and change the label:
  ```
  kubectl label pods lottery-app some-key=some-other-value
  ```
* Fix the error:
  ```
  kubectl label pods lottery-app some-key=some-other-value --overwrite=true
  kubectl get pods --show-labels
  ```
* Delete the label:
  ```
  kubectl label pods lottery-app some-key-
  kubectl get pods --show-labels
  ```

## Labeling through configuration

Explore:
* Add an environment label and some additional [recommended labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/) in the config by applying [devl-sidecar-container-pod.yaml](devl-sidecar-container-pod.yaml). Note the additional entries in the `metadata.labels` section:

  Top of `devl-sidecar-container-pod.yaml`:
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: lottery-app
    labels:
      app: lottery-app
      environment: development
      app.kubernetes.io/name: lottery-app
      app.kubernetes.io/version: "1.1.0"
      app.kubernetes.io/component: webapp
  ...    
  ```
  Apply and check:
  ```
  kubectl apply -f devl-sidecar-container-pod.yaml
  kubectl get pods --show-labels
  ```

* Apply [prod-sidecar-container-pod.yaml](prod-sidecar-container-pod.yaml) to create a second pod with a different label:

  Top of `prod-sidecar-container-pod.yaml`:
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: prod-lottery-app
    labels:
      app: lottery-app
      environment: production
      app.kubernetes.io/name: lottery-app
      app.kubernetes.io/version: "1.0.0"
      app.kubernetes.io/component: webapp
  ...    
  ```
  Apply:
  ```
  kubectl apply -f prod-sidecar-container-pod.yaml
  kubectl get pods --show-labels
  ```

* Query for all pods:
  ```
  kubectl get pods -l app=lottery-app
  ```  
* Query for just production:
  ```  
  kubectl get pods -l environment=production
  ```
* Query for just development:
  ```
  kubectl get pods -l environment=development
  ```
* Query for pods in a list:
  ```  
  kubectl get pods -l 'environment in (production, qa)'
  ```

* Clean up:
  ```
  kubectl delete pods -l app=lottery-app
  ```

## Selectors

* [Resources that support set-based requirements](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#resources-that-support-set-based-requirements)
* [JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

Examine and deploy two demo pods

* [service-demo-pod-1.yaml](service-demo-pod-1.yaml)
* [service-demo-pod-2.yaml](service-demo-pod-2.yaml)

```
kubectl apply -f service-demo-pod-1.yaml
kubectl apply -f service-demo-pod-2.yaml
kubectl get all --show-labels
```

* Select each using labels:
  ```
  kubectl get pods -l app=service-demo-app
  kubectl get pods -l app.kubernetes.io/version=1.0.0,app=service-demo-app
  kubectl get pods -l app.kubernetes.io/version=2.0.0,app=service-demo-app
  ```

* Examine the output of version 1:
```
kubectl exec `kubectl get pods -l app.kubernetes.io/version=1.0.0,app=service-demo-app -A -o jsonpath="{.items[0].metadata.name}"` -- curl -vs http://localhost
```

* Examine the output of version 2:
```
kubectl exec `kubectl get pods -l app.kubernetes.io/version=2.0.0,app=service-demo-app -A -o jsonpath="{.items[0].metadata.name}"` -- curl -vs http://localhost
```

Leave the pods running for the next section.

## Services 

* [Services, Load Balancing, and Networking](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Connecting Applications with Services](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/)
* [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Access Services Running on Clusters](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-services/)
* Minikube [Accessing apps](https://minikube.sigs.k8s.io/docs/handbook/accessing/)


### Cluster IP

ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.

* Apply [service-demo-cluster-ip-svc.yaml](service-demo-cluster-ip-svc.yaml) and check:
  ```
  kubectl apply -f service-demo-cluster-ip-svc.yaml
  kubectl get all -o wide --show-labels
  ```

  In another terminal window run: `kubectl proxy` then access the service via the exposed API endpoint: http://127.0.0.1:8001/api/v1/namespaces/week1/services/demo-webapp-service/proxy/

__NOTE:__ Stop the `kubectl proxy` process but leave the pods running for the next section.


### NodePort

__NOTE:__ this is the same config as the service above except for the type on the last line.

* Use a specific NodePort (3007) by applying [service-demo-nodeport-svc.yaml](service-demo-nodeport-svc.yaml). __NOTE:__ this is the same config as the service above except for the nodePort on the second to last line.

* The bottom of `service-demo-nodeport-svc.yaml`:
  ```
  ...
  spec:
    selector:
      app: service-demo-app
      app.kubernetes.io/name: service-demo-app
    ports:
      - protocol: TCP
        port: 80
    type: NodePort
  ```

* Apply and run:
  ```
  kubectl apply -f service-demo-nodeport-svc.yaml
  kubectl get all -o wide --show-labels
  minikube service list
  minikube service --url demo-webapp-service -n week1
  ```

* Use a specific NodePort (3007) by applying [service-demo-nodeport-30007-svc.yaml](service-demo-nodeport-30007-svc.yaml). __NOTE:__ this is the same config as the service above except for the nodePort on the second to last line.

  The bottom of `service-demo-nodeport-30007-svc.yaml`:
  ```
  ...
  spec:
    selector:
      app: service-demo-app
      app.kubernetes.io/name: service-demo-app
    ports:
      - protocol: TCP
        port: 80
        nodePort: 30007
    type: NodePort

  ```

* Apply the update:
  ```
  kubectl apply -f service-demo-nodeport-30007-svc.yaml
  minikube service --url demo-webapp-service -n week1
  ```

* Clean up:
  ```
  kubectl get all
  kubectl delete service/demo-webapp-service
  kubectl delete pods -l app=service-demo-app
  kubectl get all
  ```

## Summary

### Configuration

* A file used to configure access to a cluster is called a *kubeconfig* file.
* By default, `kubectl` looks for a file named `config` in the `$HOME/.kube` directory. 
* You can specify other kubeconfig files by setting the `KUBECONFIG` environment variable or using the `--kubeconfig` flag.
* The kubeconfig has three parts: *clusters*, *users*, and *contexts*.
* See [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) for specifics on how to add clusters to your kubeconfig file.

### Contexts

* Contexts specify the cluster, user, and namespace used when performing operations.
* You will need at least one context per cluster, but you can specify more than one if you want a quick way to switch between users or namespaces within a cluster.
* See [Organizing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) for more.

### Namespaces

* Namespaces provide a mechanism for isolating groups of resources within a single cluster. 
* Names of resources need to be unique within a namespace, but not across namespaces. 
* Namespace-based scoping is applicable only for namespaced objects (e.g., Deployments, Services, etc.) and not for cluster-wide objects (e.g., StorageClass, Nodes, PersistentVolumes, etc.).
* Namespaces cannot be nested inside one another, and each Kubernetes resource can only be in one namespace.
* See [Share a Cluster with Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/) for more on creating, deleting, and using namespaces.

### The Kubernetes API

* The core of Kubernetes' control plane is the API server. 
* The API server exposes an HTTP API that lets end-users, different parts of your cluster, and external components communicate with one another.
* The Kubernetes API lets you query and manipulate the state of API objects in Kubernetes (for example, Pods, Namespaces, ConfigMaps, and Events).
* You can perform most operations using the [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) command-line interface or other command-line tools, such as [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/), which in turn use the API. However, you can also access the API directly using REST calls.
* Consider using one of the [client libraries](https://kubernetes.io/docs/reference/using-api/client-libraries/) if you are writing an application using the Kubernetes API.
* See [The Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/) for more.

### Pods

* Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.
* Pods are designed to support multiple cooperating processes (as containers) that form a cohesive service unit. 
* The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster. 
* The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.
* Only co-locate containers in the same Pod if they need to share resources and should be scaled together.
* Pods natively provide two kinds of shared resources for their constituent containers: [networking](https://kubernetes.io/docs/concepts/workloads/pods/#pod-networking) and [storage](https://kubernetes.io/docs/concepts/workloads/pods/#pod-storage).
* One rarely creates individual Pods directly in Kubernetes because Pods are designed as relatively ephemeral, disposable entities. 
* When a Pod gets created, it's scheduled to run on a Node in your cluster where it remains until it finishes execution, is deleted, is evicted for lack of resources, or the node fails.
* See [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) for more.

### Storage

* A Pod can specify a set of shared storage volumes that are available to all containers in the Pod.
* We used an [Init Container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) to download web content from GitHub and save it to an 
[emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) shared volume.
* In the lottery demo, we used a [Sidecar](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-1-sidecar-containers) container to update a volume shared with a web server dynamically.
* See [Storage](https://kubernetes.io/docs/concepts/storage/) for more information on how Kubernetes implements shared storage and makes it available to Pods.

### Labels

* Labels are key/value pairs attached to objects, such as Pods.
* Use labels to identify, organize, and group objects in a loosely coupled way.
* Labels can be attached to objects at creation time and subsequently added and modified at any time. 
* Each object can have a set of key/value labels defined. 
* Each key must be unique for a given object.
* Labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).
* See [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for more.


### Label selectors

* The label selector is the core grouping primitive in Kubernetes.
* Clients, users, and objects use selectors to identify a set of objects.
* The API currently supports two types of selectors: [equality-based](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#equality-based-requirement) and [set-based](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-based-requirement).
* Multiple labels can be in scope for a selector. If so, all must be satisfied to match.
* See [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for more.


### Services

* A Service is an abstraction that defines a logical set of Pods and a policy to access them (sometimes this pattern is called a micro-service). 
* A selector usually determines the set of Pods targeted by a Service.
* While the actual Pods that compose the backend set may change, the frontend clients should not need to be aware of that, nor should they need to keep track of the group of backends themselves. The Service abstraction enables this decoupling.
* In class we explored two [service types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types), `ClusterIP` and [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport).
* Key features of `ClusterIP`:
  * It exposes the Service on a cluster-internal IP.
  * Using a `ClusterIP` makes the Service only reachable from within the cluster. 
  * This is the default `ServiceType`.
* Key features of `NodePort`:
  * It exposes the Service on each Node's IP at a static port (the NodePort). 
  * The Service is exposed outside the cluster through the NodePort Service at `<NodeIP>:<NodePort>`.
  * Kubernetes automatically creates a corresponding ClusterIP Service to internally route traffic from the externally exposed NodePort Service.
* See [Services](https://kubernetes.io/docs/concepts/services-networking/service/) for more.

### Resources for debugging

* [Troubleshoot Clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)
* [Troubleshoot Applications](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)
* [Debug Running Pods](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/)
* [Debugging with container exec](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/#container-exec)