# Kubernetes Workloads

* [Workloads](https://kubernetes.io/docs/concepts/workloads/)


## Getting started:

Update the current context to use a new namespace:
```
kubectl create namespace week2
kubectl config set-context --current --namespace=week2
```

## Job

 A [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate.

### Regular jobs

* Run the [Pi example job](https://kubernetes.io/docs/concepts/workloads/controllers/job/#running-an-example-job)
  ```
  kubectl apply -f pi-job.yaml; watch kubectl get all
  ```

* Explore:
  ```
  kubectl describe jobs/pi
  ```

* View the output using [JSONPath](https://kubernetes.io/docs/reference/kubectl/jsonpath/):
  ```
  kubectl logs `kubectl get pods --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}'`
  ```

* Create our own job from [lottery-job.yaml](./lottery-job.yaml):
  ```
  kubectl apply -f lottery-job.yaml; watch kubectl get all
  ```

* View the output:
  ```
  for pod in `kubectl get pods -l job-name=lottery-job -A -o jsonpath="{.items[*].metadata.name}"`; do echo "Pod $pod:"; kubectl logs $pod; done
  ```

* Attempt to rerun the job: 
  ```
  kubectl apply -f lottery-job.yaml
  kubectl create -f lottery-job.yaml
  ```

* Remove the terminated job: 
  ```
  kubectl delete -f lottery-job.yaml
  ```

* Now rerun the job: 
  ```
  kubectl create -f lottery-job.yaml; watch kubectl get all
  ``` 


### Parallel jobs

See [Parallel execution for Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/#parallel-jobs) for more.

* Run [parallel-lottery-job.yaml](./parallel-lottery-job.yaml):
 
* Apply and wait for 7 pods to appear with a completed status:
  ```
  kubectl apply -f parallel-lottery-job.yaml; watch kubectl get all
  ```

* View the details:
  ```
  kubectl get jobs,pods --show-labels
  kubectl get jobs,pods -l job-name=parallel-lottery-job
  ```

* Check the output:
  ```
  for pod in `kubectl get pods -l job-name=parallel-lottery-job -A -o jsonpath="{.items[*].metadata.name}"`; do echo "Pod $pod:"; kubectl logs $pod; done
  ```

Explore:
* Notice the difference in execution time: 
  ```
  kubectl get jobs
  ```

* Attempt to rerun the job: 
  ```
  kubectl apply -f parallel-lottery-job.yaml
  kubectl create -f parallel-lottery-job.yaml
  ```

* Remove the terminated job: 
  ```
  kubectl delete jobs,pods -l job-name=parallel-lottery-job
  ```

* Now rerun the job: 
  ```
  kubectl apply -f parallel-lottery-job.yaml; watch kubectl get all
  ``` 

* Print out the job names:
  ```
  kubectl get pods -o jsonpath='{range .items[*]}{@.status.podIP}{" "}{@.metadata.name}{"\n"}{end}'
  ```

* Look at the labels for the job:
  ```
  kubectl get all --show-labels
  ```

* Clean up:
  ```
  kubectl delete jobs,pods -l 'job-name in (pi, lottery-job, parallel-lottery-job)'
  ```
* Verify all jobs have been deleted:
  ```    
  kubectl get all
  ```


## CronJob

A [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) creates Jobs on a repeating schedule.

* Apply [lottery-cronjob.yaml](lottery-cronjob.yaml), run and check the logs:
  ```
  kubectl apply -f lottery-cronjob.yaml; watch kubectl get all --show-labels
  ```
  __NOTE:__ It will take up to one minute for output to appear.

* Describe the pod:
  ```
  kubectl describe cronjob.batch/lottery-cron-job
  ```

* Once a pod appears, check the output:
  ```
  for pod in `kubectl get pods -l job-class=lottery-number-picker -o jsonpath="{.items[*].metadata.name}"`; do echo "Pod $pod:"; kubectl logs $pod; done
  ```

* Use parallelism for our lotter cronjob. Apply [parallel-lottery-cronjob.yaml](parallel-lottery-cronjob.yaml):
  ```
  kubectl apply -f parallel-lottery-cronjob.yaml; watch kubectl get all --show-labels
  ```
  __NOTE:__ It will take up to one minute for new jobs to appear.

* View details:
  ```
  kubectl get all --show-labels
  kubectl describe cronjob.batch/parallel-lottery-cron-job
  ```
  __Question:__ How many jobs run each time?

* Check the output:
  ```
  for pod in `kubectl get pods -l job-class=parallel-lottery-number-picker -o jsonpath="{.items[*].metadata.name}"`; do echo "Pod $pod:"; kubectl logs $pod; done
  ```

* Clean up: 
  ```
  kubectl delete jobs,cronjobs --all
  ```

## ReplicaSet

* A [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)'s purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

kuard-rs.yaml:
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kuard-replica-set
  labels:
    app: app-kuar-demo
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      app: app-kuar-demo
  
  template:
    metadata:
      labels:
        app: app-kuar-demo
    
    spec:
      containers:
        - name: app-kuar-demo
          image: gcr.io/kuar-demo/kuard-arm64:blue # use -amd64 on Intel or AMD processors
```

Apply:
```
kubectl apply -f kuard-rs.yaml
kubectl get all --show-labels
```

Shell into a replica and look around:
```
kubectl exec -it `kubectl get pods -l app=app-kuar-demo -o jsonpath="{.items[0].metadata.name}"` -- sh

$ wget -O - localhost:8080
$ hostname
$ wget -O - `hostname`:8080
$ exit
```

Explore:

* Scale by changing the replica count to 2 in `kuard-rs.yaml` and reapplying the config. 
* Checkout what happens with: `kubectl get all --show-labels`
* Scale the replicas via the command line: `kubectl scale replicaset.apps/kuard-replica-set --replicas=4`
* Checkout what happens with: `kubectl get all --show-labels`
* Kill a replica and watch what happens: 
```
kubectl delete pods `kubectl get pods -l app=app-kuar-demo -o jsonpath="{.items[0].metadata.name}"`; watch kubectl get all
```

### Alternatives to ReplicaSet

#### Deployment (recommended)

Deployment is an object which can own ReplicaSets and update them and their Pods via declarative, server-side rolling updates. While ReplicaSets can be used independently, today they're mainly used by Deployments as a mechanism to orchestrate Pod creation, deletion and updates. When you use Deployments you don't have to worry about managing the ReplicaSets that they create. Deployments own and manage their ReplicaSets. As such, it is recommended to use Deployments when you want ReplicaSets.

## Load Balancer

* [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)
* [Minikube load balancer access](https://minikube.sigs.k8s.io/docs/handbook/accessing/#loadbalancer-access)

kuard-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: kuard-service
spec:
  selector:
    app: app-kuar-demo
  ports:
    - protocol: TCP
      port: 8080
  type: LoadBalancer
```

Apply the config:

```
kubectl apply -f kuard-service.yaml
kubectl get all
```

__NOTE:__ Run `minikube tunnel` in a second terminal window and provide your password when prompted.

Finding the service IP address:
```
kubectl get service kuard-service
kubectl get all -o wide
```

Explore:
* Point your web browser to port 8080 of the EXTERNAL-IP address from `kubectl get service kuard-service`
* Disable caching and refresh. Notice the host name and IP address

See [Services](https://kubernetes.io/docs/concepts/services-networking/service/) for more.

__NOTE:__ Leave the service running for the next section but delete the ReplicaSet:
```
kubectl delete replicaset.apps/kuard-replica-set
```

## Deployment

* A [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) provides declarative updates for Pods and ReplicaSets.

__NOTE:__ Make sure the ReplicaSet from the previous section is not running:
```
kubectl delete replicaset.apps/kuard-replica-set
```

kuard-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuard-deployment
spec:
  selector:
    matchLabels:
      app: app-kuar-demo
  replicas: 3
  strategy:
    type: Recreate  
  template:
    metadata:
      labels:
        app: app-kuar-demo
    spec:
      containers:
      - image: gcr.io/kuar-demo/kuard-arm64:blue # use -amd64 on Intel or AMD processors
        name: app-kuar-demo
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "750Mi"
            cpu: "500m"        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          periodSeconds: 2
          initialDelaySeconds: 0
          failureThreshold: 3
          successThreshold: 1
        livenessProbe:
          httpGet:
            path: /healthy
            port: 8080
          initialDelaySeconds: 5
          timeoutSeconds: 1
          periodSeconds: 10
          failureThreshold: 3  
        ports:
        - containerPort: 8080
```

See also:
* [Deployment strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy)
* [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
* [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

Apply the config:
```
kubectl apply -f kuard-deployment.yaml; watch kubectl get all -o wide
kubectl get all -o wide --show-labels
kubectl describe deployment.apps/kuard-deployment
```

Explore:
* Check the rollout history: `kubectl rollout history deployment kuard-deployment`
* Annotate the change cause: `kubectl annotate deployment kuard-deployment kubernetes.io/change-cause="for a good reason"`
* Point your web browser to port 8080 of the EXTERNAL-IP address from `kubectl get service kuard-service`
* Disable caching and refresh. Notice the host name and IP address
* In the command window, run `watch kubectl get all -o wide`. With the watch output running, Click on the "Liveness Probe" tab then click on the "Fail" link.
* With the watch output still running, Click on the "Readiness Probe" tab then click on the "10" link.
* Enable the metrics API: `minikube addons enable metrics-server`
* Watch CPU and memory in use: `watch kubectl top pods`
* With the watch output still running, Click on the "KeyGen Workload" tab, enable it and click submit.
* With the watch output still running, Click on the "Memory" tab then click on "Allocate 500 MiB" link.
* Run `kubectl edit deployment.apps/kuard-deployment` then locate and remove these 3 lines:
```
          limits:
            memory: "750Mi"
            cpu: "500m"        
```
* Rerun the `watch kubectl top pods` and play around with the memory and CPU load. Can you push it farther than you did before?
* Run: `kubectl top nodes`

### Updating a deployment

* [Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)

1. Edit `kuard-deployment.yaml` and change the image from `gcr.io/kuar-demo/kuard-arm64:blue` to `gcr.io/kuar-demo/kuard-arm64:green`, using the -amd64 version if you are on an Intel or AMD based machine.
1. Apply the changes
   ```
   kubectl apply -f kuard-green-deployment.yaml --record=true; watch kubectl get all -o wide
   ```
1. Check the status: 
   * `watch kubectl get all`
   * Refresh the browser page and notice the version name
   * `kubectl describe deployment.apps/kuard-deployment`
   * `kubectl get all`
   * `kubectl describe deployment.apps/kuard-deployment`

### Rolling back a deployment

* [Rolling Back a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)

1. Edit `kuard-deployment.yaml` and change the image from `gcr.io/kuar-demo/kuard-arm64:green` to `gcr.io/kuar-demo/kuard-arm64:red`, using the -amd64 version if you are on an Intel or AMD based machine.
1. Apply the changes.
   ```
   kubectl apply -f kuard-red-deployment.yaml --record=true; watch kubectl get all -o wide
   ```
1. Check the status: 
   * `watch kubectl get all`. Is the deployment successful?
   * `kubectl describe deployment.apps/kuard-deployment`
   * `kubectl get all`
1. Roll back the deployment:
   ```
   kubectl rollout undo deployment kuard-deployment; watch kubectl get all -o wide
   ```
1. Check the status: 
  * `kubectl describe deployment.apps/kuard-deployment`
  * `kubectl get all`
  * Check the rollout history: `kubectl rollout history deployment kuard-deployment`


### Clean up:
```
kubectl delete services,deployments --all
```


## Secrets

* [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

```
kubectl create secret generic postgres-db-password --from-literal=password='Use-a-Better-Passw0rd'

kubectl get secret
kubectl describe secret postgres-db-password
```


## StatefulSet

* Like a Deployment, a [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling. 

* [Run a Single-Instance Stateful Application](https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/)
* [PersistentVolumeClaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)


postgres-stateful-set.yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  ports:
  - port: 5432
    nodePort: 30432  
  type: NodePort  
  selector:
    app: postgres
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pv-claim
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  selector:
    matchLabels:
      app: postgres
  serviceName: "postgres"
  replicas: 1 
  minReadySeconds: 10
  template:
    metadata:
      labels:
        app: postgres
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - image: postgres
        name: postgres
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "750Mi"
            cpu: "500m"            
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-db-password
              key: password
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - name: postgres-persistent-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-persistent-storage
        persistentVolumeClaim:
          claimName: postgres-pv-claim
```

Apply:
```
kubectl apply -f postgres-stateful-set.yaml
kubectl get all
kubectl get pvc,pv,secrets,pods,svc,statefulset
minikube service list
```

Explore:
* Using a SQL client like [DBeaver](https://dbeaver.io/download/), connect to the database using the password "Use-a-Better-Passw0rd" along with the IP address and port number from the service list above:

![A screenshot of the DBeaver connection dialog box](https://user-images.githubusercontent.com/46822968/152623068-12d2ab50-7f4f-4088-8485-1716b4cfb1f1.png)

* Verify this query `select * from products;` fails with a products table does not exist error.
* Run this script in a new SQL Editor:
  ```
  drop table if exists products;

  CREATE TABLE products (
      product_no integer,
      name text,
      price numeric
  );

  INSERT INTO products (product_no, name, price) VALUES
      (1, 'Cheese', 9.99),
      (2, 'Bread', 1.99),
      (3, 'Milk', 2.99);
  ```
* Verify the table has data: `select * from products;`
* Disconnect from the database
* Scale down to zero to kill the pod: 
  ```
  kubectl scale statefulset.apps/postgres --replicas=0
  kubectl get all --show-labels
  ```

* Scale back to 1 instance:
  ```
  kubectl scale statefulset.apps/postgres --replicas=1
  kubectl get all --show-labels
  ```

* In DBeaver, reconnect to the database
* Verify the table still exisits and has data: `select * from products;`
* Quit DBeaver


### Clean up
```
kubectl get pvc,pv,secrets,pods,svc,statefulset
kubectl delete statefulset.apps/postgres service/postgres
kubectl delete secret postgres-db-password
kubectl get pvc,pv,secrets,pods,svc,statefulset
```

## DaemonSet

* A [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected.

### Demo using Jeff's three node cluster

```
kubectl config get-contexts
kubectl config set-context microk8s-jeff
kubectl get nodes -o wide
kubectl get nodes --show-labels
```

* Follow the example in [Writing a DaemonSet Spec](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec)
* View the outcome: `kubectl get all --all-namespaces -o wide`

Label the nodes:
```
kubectl label nodes kube02 super-power=shapeshifter
kubectl label nodes kube03 super-power=time-travel
kubectl label nodes kube04 super-power=shapeshifter
```

### Add a nodeSelector:

* [Running Pods on select Nodes](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#running-pods-on-select-nodes)

daemonset-shapeshifter.yaml:
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      nodeSelector:
        super-power: shapeshifter
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

__Apply and explore:__

```
kubectl apply -f daemonset-shapeshifter.yaml
kubectl get all --all-namespaces -o wide
kubectl describe daemonset.apps/fluentd-elasticsearch -n kube-system
```

### Clean up:

```
kubectl delete daemonset.apps/fluentd-elasticsearch -n kube-system
kubectl label nodes kube02 super-power-
kubectl label nodes kube03 super-power-
kubectl label nodes kube04 super-power-
```

## Delete the namespace

Optional but deleting the namespace deletes everything in it for a fast, easy cleanup.

```
kubectl delete namespace week2
```


# Week 5 Lab

https://github.com/ColumbusStateWorkforceInnovation/infrastructure-kubernetes-part2