<a id="top"></a>

# ☸️ Kubernetes Study Notes

Commands, manifests and exam tips collected while studying Kubernetes administration.

> [!TIP]
> Most commands use `k` as a short alias for `kubectl`:
>
> ```bash
> alias k=kubectl
> ```

📌 **Related notes:** [Determining Service port numbers](determinate_port_number.md)

## 📚 Table of Contents

| | Part | Topics |
| :---: | :--- | :--- |
| 🧱 | [1. Core Concepts](#1-core-concepts) | [ReplicaSet](#replicaset) · [Deployment](#deployment) · [Generate YAML Templates](#generate-yaml-templates) · [Services](#services) · [Namespaces](#namespaces) · [Exam Tips](#exam-tips) |
| 🗓️ | [2. Scheduling](#2-scheduling) | [Manual Scheduling](#manual-scheduling) · [Labels and Selectors](#labels-and-selectors) · [Taints and Tolerations](#taints-and-tolerations) · [Node Selector](#node-selector) · [Node Affinity](#node-affinity) · [Resource Requests and Limits](#resource-requests-and-limits) · [LimitRange](#limitrange) · [ResourceQuota](#resourcequota) · [DaemonSet](#daemonset) · [Static Pods](#static-pods) · [Priority Classes](#priority-classes) · [Multiple Schedulers](#multiple-schedulers) |
| 📈 | [3. Logging and Monitoring](#3-logging-and-monitoring) | [Monitoring](#monitoring) · [Application Logs](#application-logs) |
| 🔄 | [4. Application Lifecycle](#4-application-lifecycle) | [Rollouts and Rollbacks](#rollouts-and-rollbacks) · [Commands and Arguments](#commands-and-arguments) · [Environment Variables](#environment-variables) · [ConfigMaps](#configmaps) · [Secrets](#secrets) · [Multi-Container Pods](#multi-container-pods) · [Autoscaling](#autoscaling) · [Horizontal Pod Autoscaler](#horizontal-pod-autoscaler) · [In-Place Pod Resizing](#in-place-pod-resizing) · [Vertical Pod Autoscaler](#vertical-pod-autoscaler) |
| 🛠️ | [5. Cluster Maintenance](#5-cluster-maintenance) | [OS Upgrades](#os-upgrades) · [Kubernetes Releases](#kubernetes-releases) · [Backup and Restore](#backup-and-restore) |
| 🔐 | [6. Security](#6-security) | [Authentication](#authentication) · [TLS in Kubernetes](#tls-in-kubernetes) · [Certificate Signing Requests](#certificate-signing-requests) · [Kubeconfig](#kubeconfig) · [API Groups](#api-groups) · [Authorization](#authorization) · [Role-Based Access Control](#role-based-access-control) · [Cluster Roles](#cluster-roles) · [Service Accounts](#service-accounts) · [Image Security](#image-security) · [Network Policies](#network-policies) |
| 🧩 | [7. Extending Kubernetes](#7-extending-kubernetes) | [Custom Resource Definitions](#custom-resource-definitions) · [Operator Framework](#operator-framework) |
| 💾 | [8. Storage](#8-storage) | [Docker Storage](#docker-storage) · [Docker Volume Drivers](#docker-volume-drivers) · [Container Storage Interface](#container-storage-interface) · [Volumes](#volumes) · [Persistent Volumes](#persistent-volumes) · [Persistent Volume Claims](#persistent-volume-claims) · [Storage Classes](#storage-classes) |
| 🌐 | [9. Networking](#9-networking) | [Networking Prerequisites](#networking-prerequisites) · [Pod Networking](#pod-networking) · [CNI in Kubernetes](#cni-in-kubernetes) · [IP Address Management](#ip-address-management) · [Service Networking](#service-networking) · [CoreDNS in Kubernetes](#coredns-in-kubernetes) · [Ingress](#ingress) · [Gateway API](#gateway-api) |
| 🏗️ | [10. Cluster Installation](#10-cluster-installation) | [Kubeadm](#kubeadm) |
| ⎈ | [11. Helm](#11-helm) | [Helm Basics](#helm-basics) · [Helm Charts](#helm-charts) · [Customizing Chart Parameters](#customizing-chart-parameters) · [Helm Releases](#helm-releases) |
| 🧬 | [12. Kustomize](#12-kustomize) | [Kustomization File](#kustomization-file) · [Patches](#patches) · [Patching Dictionaries](#patching-dictionaries) · [Patching Lists](#patching-lists) · [Overlays](#overlays) · [Components](#components) |
| 🩺 | [13. Troubleshooting](#13-troubleshooting) | [Cluster Components](#cluster-components) · [Inspecting Multi-Container Pods](#inspecting-multi-container-pods) |

---

## 1. Core Concepts

### ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

```bash
kubectl create -f replicaset-definition.yaml
kubectl get replicaset                      # short: k get rs
kubectl replace -f replicaset-definition.yaml
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl scale --replicas=6 replicaset myapp-replicaset
kubectl delete replicaset myapp-replicaset
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

```bash
kubectl create -f deployment-definition.yaml
kubectl get deployment
kubectl get all        # pods, replicasets and deployments at once
```

### Generate YAML Templates

`kubectl run` and `kubectl create` can generate a YAML template for you.

| Goal | Command |
| :--- | :--- |
| Create an NGINX pod | `kubectl run nginx --image=nginx` |
| Generate a pod manifest without creating it | `kubectl run nginx --image=nginx --dry-run=client -o yaml` |
| Create a deployment | `kubectl create deployment --image=nginx nginx` |
| Generate a deployment manifest without creating it | `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml` |
| …and save it to a file | `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml` |
| Generate a deployment with 4 replicas (k8s 1.19+) | `kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml` |

Make the changes you need in the file (for example, more replicas), then create the deployment:

```bash
kubectl create -f nginx-deployment.yaml
```

> [!TIP]
> `-o yaml` prints the manifest and `--dry-run=client` stops kubectl from creating the object.

### Services

> 📌 Not sure which port goes where? See [Determining Service port numbers](determinate_port_number.md).

| Type | Reachable from |
| :--- | :--- |
| `ClusterIP` (default) | Inside the cluster only |
| `NodePort` | Outside the cluster, at `<Node-IP>:<nodePort>` (`30000–32767`) |
| `LoadBalancer` | Outside the cluster, through a cloud provider's load balancer |

#### NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

```bash
kubectl create -f service-definition.yaml
kubectl get services
```

#### ClusterIP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
  - targetPort: 80
    port: 80
  selector:
    app: myapp
    type: back-end
```

#### LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: LoadBalancer
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: back-end
```

### Namespaces

A Service's full DNS name: **`db-service.dev.svc.cluster.local`**

| Part | Meaning |
| :--- | :--- |
| `db-service` | Service name |
| `dev` | Namespace |
| `svc` | Service subdomain |
| `cluster.local` | Cluster domain |

```bash
kubectl get pods --namespace=kube-system
kubectl create -f pod-definition.yaml --namespace=dev
kubectl expose pod redis --port=6379 --name=redis-service --type=ClusterIP
```

Set the namespace inside the pod definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev   # <---- namespace
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx
      image: nginx
```

Namespace definition file:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```bash
kubectl create -f namespace-dev.yaml
kubectl create namespace dev
kubectl config set-context $(kubectl config current-context) --namespace=dev   # switch default namespace
kubectl get pods --all-namespaces
k get namespace
```

Limit resources in a namespace with a resource quota:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

### Exam Tips

Imperative commands save a lot of time:

```bash
k run --image=nginx nginx
k create deployment --image=nginx nginx
k expose deployment nginx --port 80
k edit deployment nginx
k scale deployment nginx --replicas=5
k set image deployment nginx nginx=nginx:1.18
```

Explore the API resources:

```bash
kubectl api-resources
k explain pods
k explain pods.spec
k explain pods --recursive
kubectl run custom-nginx --image=nginx --port=8080
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 2. Scheduling

### Manual Scheduling

Assign a pod to a node yourself with `nodeName`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node01
  containers:
  - image: nginx
    name: nginx
```

```bash
k get pods --watch
```

### Labels and Selectors

Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: front-end
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
```

```bash
k get pods --selector app=App1
k get all --selector env=prod
k get pods --selector env=prod --no-headers | wc -l
k get all --selector env=prod,bu=finance,tier=frontend
```

ReplicaSet:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: front-end
  annotations:
    buildversion: "1.34"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: front-end
    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp
```

> [!IMPORTANT]
> The `selector.matchLabels` must match the labels in the pod `template`.

### Taints and Tolerations

```bash
k taint nodes node-name key=value:taint-effect
k taint node node1 app=blue:NoSchedule
```

The **taint effect** says what happens to pods that do **not** tolerate the taint:

| Effect | What happens |
| :--- | :--- |
| `NoSchedule` | New pods are not scheduled on the node |
| `PreferNoSchedule` | The scheduler tries to avoid the node, but it's not guaranteed |
| `NoExecute` | New pods are not scheduled and existing pods are evicted |

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: app
    operator: "Equal"
    value: blue
    effect: NoSchedule
```

### Node Selector

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large
```

```bash
k label nodes node-1 size=Large
```

### Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn
            values:
            - Small
```

| Node affinity type | During scheduling | During execution |
| :--- | :--- | :--- |
| `requiredDuringSchedulingIgnoredDuringExecution` | Required | Ignored |
| `preferredDuringSchedulingIgnoredDuringExecution` | Preferred | Ignored |

Add affinity to an existing deployment:

```bash
kubectl edit deployment blue
```

### Resource Requests and Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```

> [!WARNING]
> **What happens when a pod exceeds its limits?**
> - **Memory** over the limit → the container is terminated with **`OOMKilled`** (out of memory).
> - **CPU** over the limit → the container is throttled.

### LimitRange

Default, minimum and maximum values per container in a namespace.

**CPU**

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      cpu: 500m
    max:
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

**Memory**

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      memory: 500Mi
    defaultRequest:
      memory: 500Mi
    max:
      memory: 1Gi
    min:
      memory: 100Mi
    type: Container
```

### ResourceQuota

Total limits for all the pods in a namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```

### DaemonSet

Runs one copy of a pod on every node.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

```bash
k create -f daemon-set.yaml
k get daemonsets
k get daemonsets --all-namespaces
kubectl describe daemonset kube-flannel-ds --namespace=kube-flannel
```

> [!TIP]
> There's no `kubectl create daemonset`. Generate a deployment and edit it into a DaemonSet:
>
> ```bash
> kubectl create deployment elasticsearch --image=registry.k8s.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml
> ```

### Static Pods

The default directory for static pod definition files is **`/etc/kubernetes/manifests`**.

> [!IMPORTANT]
> The path doesn't have to be `/etc/kubernetes/manifests`. Always check the path configured in the kubelet configuration file.

#### Lab: delete the static pod `static-greenbox`

**1. Find the node the pod runs on**

```console
controlplane:~# kubectl get pods --all-namespaces -o wide  | grep static-greenbox
default       static-greenbox-node01                 1/1     Running   0          19s     10.244.1.2   node01       <none>           <none>
```

The pod is running on `node01`.

**2. SSH to the node and find the static pod path**

```console
controlplane:~# ssh node01
node01:~# ps -ef |  grep /usr/bin/kubelet
root        4147       1  0 14:05 ?        00:00:00 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9
root        4773    4733  0 14:05 pts/0    00:00:00 grep /usr/bin/kubelet

node01:~# grep -i staticpod /var/lib/kubelet/config.yaml
staticPodPath: /etc/just-to-mess-with-you
```

Here the `staticPodPath` is `/etc/just-to-mess-with-you`.

**3. Go to that directory and delete the YAML file**

```console
node01:/etc/just-to-mess-with-you# ls
greenbox.yaml
node01:/etc/just-to-mess-with-you# rm -rf greenbox.yaml
```

Wait about 30 seconds for the pod to disappear.

### Priority Classes

```bash
k get priorityclass
```

Create a priority class:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority   # <---
value: 1000000000
description: "Priority class for mission critical pods"
```

Set the priority in the pod manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
     - containerPort: 8080
  priorityClassName: high-priority   # <---
```

```bash
k get priorityclass
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
```

### Multiple Schedulers

Scheduler configuration:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler
```

Scheduler pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --config=/etc/kubernetes/my-scheduler-config
    image: k8s.gcr.io/kube-scheduler-amd64:v1.xx.x
    name: kube-scheduler
```

Pod that uses the custom scheduler:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  schedulerName: my-custom-scheduler
```

See the scheduler logs:

```bash
k logs my-custom-scheduler --namespace=kube-system
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 3. Logging and Monitoring

### Monitoring

```bash
k top node
k top pod
```

### Application Logs

```bash
k logs -f <podName>
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 4. Application Lifecycle

### Rollouts and Rollbacks

```bash
k rollout status deployment/myapp-deployment
k rollout history deployment/myapp-deployment
k rollout undo deployment myapp-deployment
```

> [!NOTE]
> The default deployment strategy is **rolling update**.

### Commands and Arguments

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args: ["10"]
```

| Dockerfile | Pod spec | Purpose | Example |
| :--- | :--- | :--- | :--- |
| `ENTRYPOINT` | `command` | The main executable/binary to run | `sleep2.0` |
| `CMD` | `args` | The default arguments passed to that executable | `10` |

### Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
       - containerPort: 8080
     env:
       - name: APP_COLOR
         value: pink
```

### ConfigMaps

#### 1. Create the ConfigMap

**Imperative**

```bash
k create configmap <config-name> \
  --from-literal=<key>=<value>

k create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=prod
```

**Declarative**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```bash
k create -f config-map.yaml
```

#### 2. Inject it into a pod

All the keys as environment variables:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
       - containerPort: 8080
     envFrom:
       - configMapRef:
           name: app-config
```

A single key:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      valueFrom:
       configMapKeyRef:
         name: webapp-config-map
         key: APP_COLOR
    image: kodekloud/webapp-color
    name: webapp-color
```

### Secrets

#### 1. Create the Secret

**Imperative**

```bash
k create secret generic <secret-name> \
  --from-literal=<key>=<value>

k create secret generic app-secret \
  --from-literal=DB_Host=mysql
```

From a file:

```bash
k create secret generic <secret-name> \
  --from-file=<path-to-file>

k create secret generic app-secret \
  --from-file=app_secret.properties
```

**Declarative.** Values under `data` must be base64-encoded:

```bash
echo -n 'mysql' | base64     # bXlzcWw=
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=          # mysql
  DB_User: cm9vdA==          # root
  DB_Password: cGFzc3dyZA==  # passwrd
```

```bash
k create -f secret-data.yaml
```

**View secrets**

```bash
k get secrets
k describe secrets
k get secret app-secret -o yaml
```

> [!WARNING]
> Secrets are only **base64-encoded, not encrypted**. Anyone who can read the Secret can decode it.

#### 2. Inject it into a pod

All the keys as environment variables:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - secretRef:
          name: app-secret
```

```bash
k create -f pod-definition.yaml
```

A single key:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    env:
      - name: DB_Password
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: DB_Password
```

As a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
  volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret
```

### Multi-Container Pods

| Pattern | Defined under | Start order |
| :--- | :--- | :--- |
| Co-located containers | `containers` | No way to control which one starts first |
| Regular init containers | `initContainers` | Run one at a time, to completion, before the main containers |
| Sidecar containers | `initContainers` + `restartPolicy: Always` | Start before the main containers, in the order you specify, and keep running |

**Co-located containers**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: web-app
    image: web-app
    ports:
      - containerPort: 8080
  - name: main-app
    image: main-app
```

**Regular init containers**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: web-app
    image: web-app
    ports:
      - containerPort: 8080
  initContainers:
  - name: db-checker
    image: busybox
    command: ['wait-for-db-to-start.sh']
  - name: api-checker
    image: busybox
    command: ['wait-for-another-api.sh']
```

**Sidecar containers**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: web-app
    image: web-app
    ports:
      - containerPort: 8080
  initContainers:
  - name: log-shipper
    image: busybox
    command: ['setup-log-shipper.sh']
    restartPolicy: Always
```

#### Lab: log-shipping sidecar

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: app
  name: app
  namespace: elastic-stack
spec:
  initContainers:
  - name: sidecar
    image: kodekloud/filebeat-configured
    restartPolicy: Always
    volumeMounts:
      - name: log-volume
        mountPath: /var/log/event-simulator

  containers:
  - image: kodekloud/event-simulator
    name: app
    resources: {}
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - hostPath:
      path: /var/log/webapp
      type: DirectoryOrCreate
    name: log-volume
```

```bash
kubectl -n elastic-stack logs app -c sidecar          # sidecar logs
kubectl -n elastic-stack exec app -- cat /log/app.log  # logs inside the container
```

### Autoscaling

| What scales | Manual | Automated |
| :--- | :--- | :--- |
| **Cluster infrastructure** (horizontal) | `kubeadm join` (add nodes) | Cluster Autoscaler |
| **Workloads** — horizontal | `k scale ...` | Horizontal Pod Autoscaler (HPA) |
| **Workloads** — vertical | `k edit ...` | Vertical Pod Autoscaler (VPA) |

### Horizontal Pod Autoscaler

```bash
k top pod my-app-pod
k scale deployment my-app --replicas=3

k autoscale deployment my-app \
  --cpu-percent=50 --min=1 --max=10

k get hpa
k delete hpa my-app

kubectl autoscale deployment nginx-deployment --max=3 --cpu=80%
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
   - type: Resource
     resource:
       name: cpu
       target:
         type: Utilization
         averageUtilization: 50
```

### In-Place Pod Resizing

> 🚧 *To do.*

### Vertical Pod Autoscaler

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: nginx
          resources:
            requests:
              cpu: "250m"
            limits:
              cpu: "500m"
```

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: "my-app"
        minAllowed:
          cpu: "250m"
        maxAllowed:
          cpu: "2"
        controlledResources: ["cpu"]
```

VPA components:

- **VPA Recommender**
- **VPA Updater**
- **VPA Admission Controller**

```bash
k describe vpa my-app-vpa
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 5. Cluster Maintenance

### OS Upgrades

| Command | Effect |
| :--- | :--- |
| `k drain node-1` | Evicts the pods and marks the node unschedulable |
| `k cordon node-2` | Marks the node unschedulable (running pods stay) |
| `k uncordon node-1` | Makes the node schedulable again |

> [!NOTE]
> **Pod eviction timeout:** how long Kubernetes waits for a node that's down before its pods are terminated.

### Kubernetes Releases

```bash
kubeadm upgrade plan
kubeadm upgrade apply
```

### Backup and Restore

> 🚧 *To do.*

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 6. Security

### Authentication

User authentication mechanisms:

- **Static token files** — *not a recommended approach*
- **Certificates**
- **Identity services**

#### TLS certificates

TLS certificates guarantee trust between two parties.

| Key | Example files |
| :--- | :--- |
| Private key | `server.key`, `server-key.pem` |
| Public key (certificate) | `server.crt`, `client.pem`, `server.pem` |

**Public Key Infrastructure (PKI)**

| Certificate | Configured on |
| :--- | :--- |
| Root certificates | The CA server |
| Client certificates | The clients |
| Server certificates | The servers |

### TLS in Kubernetes

| Component | Type | What it does | Certificate | Key |
| :--- | :--- | :--- | :--- | :--- |
| kube-apiserver | Server | HTTPS server that manages the k8s components | `apiserver.crt` | `apiserver.key` |
| etcd server | Server | Stores the cluster info | `etcdserver.crt` | `etcdserver.key` |
| kubelet | Server | HTTPS server on the worker nodes that talks with the kube-apiserver | `kubelet.crt` | `kubelet.key` |
| Admin (us) | Client | Users who access the cluster | `admin.crt` | `admin.key` |
| kube-scheduler | Client | Asks the kube-apiserver for pods to schedule | `scheduler.crt` | `scheduler.key` |
| kube-controller-manager | Client | Accesses the kube-apiserver | `controller-manager.crt` | `controller-manager.key` |
| kube-proxy | Client | Accesses the kube-apiserver | `kube-proxy.crt` | `kube-proxy.key` |

**Inspect certificates**

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml

openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

journalctl -u etcd.service -l
kubectl logs etcd-master

crictl ps -a
crictl logs <containerID>
```

### Certificate Signing Requests

**1. The user creates a key**

```bash
openssl genrsa -out jane.key 2048
```

**2. The user creates a signing request and sends it to the admin**

```bash
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```

**3. The admin creates a `CertificateSigningRequest` object**

```bash
cat jane.csr | base64 | tr -d "\n"
```

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 600
  usages:
  - client auth
  request: <BASE64_ENCODED_CSR>
```

Once it's approved, the object's `status` shows:

```yaml
status:
  conditions:
  - type: Approved
    status: "True"
    reason: KubectlApprove
    message: "This CSR was approved by kubectl certificate approve"
    lastUpdateTime: "2026-02-03T11:46:10Z"
```

**4. List and approve the certificate**

```bash
k get csr
k certificate approve jane
k get csr jane -o yaml

kubectl certificate deny agent-smith
k delete csr agent-smith
```

> [!NOTE]
> The **controller manager** is responsible for certificates, through its `csr-approving` and `csr-signing` controllers:
>
> ```bash
> cat /etc/kubernetes/manifests/kube-controller-manager.yaml
> ```

### Kubeconfig

Without a kubeconfig you'd pass everything on every command:

```bash
k get pods \
  --server my-kube-playground:6443 \
  --client-key admin.key \
  --client-certificate admin.crt \
  --certificate-authority ca.crt
```

Instead, put it in a config file (default `$HOME/.kube/config`):

```bash
k get pods --kubeconfig config
```

The config file has three sections:

| Section | Examples |
| :--- | :--- |
| `clusters` | development, production, google |
| `users` | admin, dev user, prod user |
| `contexts` (which user uses which cluster) | admin@production, dev@google |

```yaml
apiVersion: v1
kind: Config
clusters:
    - name: my-kube-playground
      cluster:
        certificate-authority: ca.crt
        server: https://my-kube-playground:6443

contexts:
    - name: my-kube-admin@my-kube-playground
      context:
        cluster: my-kube-playground
        user: my-kube-admin

users:
    - name: my-kube-admin
      user:
        client-certificate: admin.crt
        client-key: admin.key
```

```bash
k config view
k config view --kubeconfig=my-custom-config
k config use-context prod-user@production
k config -h
```

### API Groups

| Tool | What it is |
| :--- | :--- |
| `kube-proxy` | Enables connectivity between pods and services across the nodes in the cluster |
| `kubectl proxy` | HTTP proxy service created by the kubectl utility to access the kube-apiserver |

### Authorization

| Mode | Description |
| :--- | :--- |
| **Node** | Node authorizer for kubelets (part of the `system:nodes` group) |
| **ABAC** | Attribute-based access control: associates a user with a set of permissions through a policy file |
| **RBAC** | Role-based access control: create a role with permissions and associate users with it. The standard approach |
| **Webhook** | Delegates the decision to an external service |
| **AlwaysAllow** / **AlwaysDeny** | Allows or denies every request |

```bash
--authorization-mode=Node,RBAC,Webhook
```

### Role-Based Access Control

**Role**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
    name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","create","update","delete"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
```

**RoleBinding**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
    name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

```bash
k get roles
k get rolebindings
k describe role developer
k describe rolebinding devuser-developer-binding
```

**Check access**

```bash
k auth can-i create deployments
k auth can-i delete nodes
k auth can-i create deployments --as dev-user
k auth can-i create pods --as dev-user
k auth can-i create pods --as dev-user --namespace test
```

**Access to specific pods only**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
    name: developer
rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get","create","delete"]
      resourceNames: ["blue","orange"]
```

#### Lab: give `dev-user` access to the `blue-app` pod in namespace `blue`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: blue
rules:
- apiGroups:
  - ""
  resourceNames:
  - blue-app
  resources:
  - pods
  verbs:
  - get
  - watch
  - create
  - delete
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
  namespace: blue
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: dev-user
```

### Cluster Roles

For cluster-scoped resources such as nodes.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
    name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get","list","create","update","delete"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
    name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

### Service Accounts

```bash
k get serviceaccount
k describe serviceaccount default
k describe pod my-kubernetes-dashboard
```

The service account token is mounted as a **projected volume**: a dynamic directory created automatically inside the pod, at `/var/run/secrets/kubernetes.io/serviceaccount`.

```console
~> k exec -it my-kubernetes-dashboard -- ls /var/run/secrets/kubernetes.io/serviceaccount
ca.crt  namespace  token
```

#### Custom service account

CLI:

```bash
k create serviceaccount dashboard-sa
```

File:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: default
automountServiceAccountToken: false   # don't auto-mount the SA token
```

Associate the service account with the pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa
  automountServiceAccountToken: false   # don't auto-mount the SA token
```

```bash
k describe pod my-kubernetes-dashboard
k exec -it my-kubernetes-dashboard -- ls /var/run/secrets/kubernetes.io/serviceaccount
```

#### Tokens

```bash
k create token dashboard-sa
k create token dashboard-sa --duration 2h
```

Deployment configured to use a projected volume:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dashboard
  namespace: default
  labels:
    name: web-dashboard
spec:
  replicas: 1
  selector:
    matchLabels:
      name: web-dashboard
  template:
    metadata:
      labels:
        name: web-dashboard
    spec:
      containers:
      - name: web-dashboard
        image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
        ports:
        - containerPort: 8080
        env:
        - name: PYTHONUNBUFFERED
          value: "1"
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount/
          name: token
          readOnly: true
      serviceAccountName: dashboard-sa
      automountServiceAccountToken: false
      volumes:
      - name: token
        projected:
          sources:
          - serviceAccountToken:
              path: token
```

```bash
kubectl exec $(kubectl get pod -l name=web-dashboard -o jsonpath='{.items[0].metadata.name}') -- ls /var/run/secrets/kubernetes.io/serviceaccount/
```

### Image Security

To pull from a private registry, create a `docker-registry` secret:

```bash
k create secret docker-registry private-reg-cred \
  --docker-server=myprivateregistry.com:5000 \
  --docker-username=dock_user \
  --docker-password=dock_password \
  --docker-email=registry-user@org.com
```

Then pass that secret to the pod definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: private-registry.io/apps/internal-app
  imagePullSecrets:
  - name: private-reg-cred
```

### Network Policies

| Traffic | Direction |
| :--- | :--- |
| **Ingress** | Incoming: requests received |
| **Egress** | Outgoing: requests sent |

A network policy only applies to the pods it selects.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
```

> [!TIP]
> Each `-` item under `from` is a separate rule (**OR**). To require **both** a pod label and a namespace (**AND**), put `podSelector` and `namespaceSelector` in the same item: no `-` before `namespaceSelector`.

```bash
k create -f policy-definition.yaml
```

Ingress and egress:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```

```bash
kubectl get netpol
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 7. Extending Kubernetes

### Custom Resource Definitions

A CRD extends the Kubernetes API with resources that aren't available in a default installation. You need two pieces:

- **Custom resource**: defined by a CRD
- **Custom controller**: code that watches for changes to the custom resource and acts on them

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  scope: Namespaced
  group: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
      - ft
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  type: integer
                  minimum: 1
                  maximum: 10
```

### Operator Framework

> 🚧 *To do.*

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 8. Storage

### Docker Storage

By default, Docker stores its data here:

```text
/var/lib/docker
├── aufs
├── containers
├── image
└── volumes
```

#### Image layers

```bash
docker build Dockerfile -t mmushad/my-custom-app
```

| Layer | Content |
| :---: | :--- |
| 1 | Base Ubuntu layer |
| 2 | Changes in apt packages |
| 3 | Changes in pip packages |
| 4 | Source code |
| 5 | Update entrypoint |

> [!TIP]
> When you rebuild an image, Docker reuses the cached layers that didn't change, so the build is faster.

#### Volumes

```bash
docker volume create data_volume
```

```text
/var/lib/docker
└── volumes
    └── data_volume
```

| Mount type | Source | Command |
| :--- | :--- | :--- |
| **Volume mount** | A volume in `/var/lib/docker/volumes` | `docker run -v data_volume:/var/lib/mysql mysql` |
| **Bind mount** | Any directory on the host | `docker run -v /data/mysql:/var/lib/mysql mysql` |

The newer `--mount` syntax:

```bash
docker run \
  --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

**Storage drivers:** AUFS · ZFS · BTRFS · Device Mapper · Overlay · Overlay2

### Docker Volume Drivers

Volumes are not handled by storage drivers. They're handled by **volume driver plugins**.

```bash
docker run -it \
  --name mysql \
  --volume-driver rexray/ebs \
  --mount src=ebs-vol,target=/var/lib/mysql \
  mysql
```

### Container Storage Interface

| Interface | Stands for |
| :--- | :--- |
| CNI | Container Network Interface |
| CSI | Container Storage Interface |

Example: installing a CSI driver with Helm:

```bash
helm install topolvm topolvm --version 17.2.0 --namespace topolvm-system --create-namespace --values <file>
```

### Volumes

A volume keeps the data even if the pod is deleted. Here it's stored in a directory on the host:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```

### Persistent Volumes

| Object | Created by |
| :--- | :--- |
| **PersistentVolume (PV)** | The admin |
| **PersistentVolumeClaim (PVC)** | The user, to use the storage |

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

```bash
k create -f pv-definition.yaml
k get persistentvolume
```

**Reclaim policy** (`persistentVolumeReclaimPolicy`): what happens to the PV when its claim is deleted.

| Policy | Effect |
| :--- | :--- |
| `Retain` | Keeps the PV and its data |
| `Delete` | Deletes the PV |

### Persistent Volume Claims

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```bash
k get persistentvolumeclaim
k delete persistentvolumeclaim myclaim
```

Once the PVC exists, use it in a pod by putting the claim name under `persistentVolumeClaim` in the `volumes` section:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

> [!TIP]
> A PVC only binds to a PV with matching access modes and enough capacity.

#### Lab: move a pod's logs from `hostPath` to a PVC

**1. Pod writing logs to a host directory**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
      volumeMounts:
      - mountPath: /log
        name: log-volume
  volumes:
    - name: log-volume
      hostPath:
        path: /var/log/webapp
        type: Directory
```

**2. PersistentVolume**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```

**3. PersistentVolumeClaim**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

**4. Point the pod's volume at the claim**

```yaml
  volumes:
    - name: log-volume
      persistentVolumeClaim:
        claimName: claim-log-1
```

### Storage Classes

| Provisioning | How |
| :--- | :--- |
| **Static** | You provision the volume manually |
| **Dynamic** | A StorageClass creates the PV automatically |

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: google-storage
  resources:
   requests:
     storage: 500Mi
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: myclaim
```

The PV is created automatically by the storage class.

#### Example: `local-path`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-path
  resources:
   requests:
     storage: 500Mi
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    volumeMounts:
    - mountPath: /var/www/html
      name: data-volume
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: local-pvc
```

#### Example: delayed binding

With `WaitForFirstConsumer`, the volume isn't bound until a pod uses the claim.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 9. Networking

### Networking Prerequisites

```bash
ip link                                          # list interfaces
ip addr                                          # list IP addresses
ip addr add 192.168.1.10/24 dev eth0             # assign an IP to an interface
ip route                                         # show the routing table (or: route)
ip route add 192.168.1.0/24 via 192.168.2.1      # add a route
cat /proc/sys/net/ipv4/ip_forward                # 1 = IP forwarding enabled
```

### Pod Networking

The Kubernetes networking model:

- ✅ Every pod gets its own IP address.
- ✅ Every pod can reach every other pod on the same node.
- ✅ Every pod can reach every other pod on other nodes **without NAT**.

Wiring a network namespace to a bridge by hand:

```bash
# 1. Create a bridge on the host and bring it up
ip link add v-net-0 type bridge
ip link set dev v-net-0 up
ip addr add 192.168.15.5/24 dev v-net-0

# 2. Create a veth pair (a virtual cable) and plug each end in
ip link add veth-red type veth peer name veth-red-br
ip link set veth-red netns red
ip link set veth-red-br master v-net-0

# 3. Give the namespace end an IP and bring it up
ip -n red addr add 192.168.15.1 dev veth-red
ip -n red link set veth-red up

# 4. Route to other networks through the bridge, and NAT outgoing traffic
ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
```

> [!TIP]
> You can automate this with a script that has an **ADD** section and a **DELETE** section. That's what a CNI plugin is.

### CNI in Kubernetes

The **container runtime** creates the containers and calls the CNI plugin to connect them to the network.

| What | Where |
| :--- | :--- |
| CNI plugin binaries | `/opt/cni/bin` |
| CNI configuration | `/etc/cni/net.d` (e.g., `bridge.conflist`, `flannel.conflist`) |

### IP Address Management

> 🚧 *To do.*

### Service Networking

Services are a **cluster-wide** concept: they exist on all the nodes in the cluster.

```bash
k get pods -o wide
k get service

kube-api-server --service-cluster-ip-range ipNet   # default: 10.0.0.0/24
ps aux | grep kube-api-server
```

### CoreDNS in Kubernetes

CoreDNS is deployed as pods. The kubelet automatically configures each pod to resolve names through CoreDNS by writing its `/etc/resolv.conf`:

```text
nameserver      10.96.0.10
```

The address comes from the kubelet config:

```console
$ cat /var/lib/kubelet/config.yaml
...
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
...
```

### Ingress

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: critical-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
            name: pay-service
            port:
              number: 8282
```

### Gateway API

Covers **ingress**, **load balancing** and **service mesh** APIs, with one resource per role:

| Role | Resource |
| :--- | :--- |
| Infrastructure provider | `GatewayClass` |
| Cluster operator | `Gateway` |
| Application developer | `HTTPRoute`, `TCPRoute` |

**GatewayClass** (infrastructure provider)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: example.com/gateway-controller
```

**Gateway** (cluster operator)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All
```

**HTTPRoute** (application developer)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
  parentRefs:
  - name: example-gateway
  hostnames:
  - "www.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /login
    backendRefs:
    - name: example-svc
      port: 8080
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 10. Cluster Installation

### Kubeadm

Steps to set up a cluster with kubeadm:

1. Provision multiple systems: 1 master and 2 worker nodes.
2. Install the kubeadm tool.
3. Initialize the master server.
4. Set up the pod network.
5. Join the worker nodes to the master node.

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 11. Helm

### Helm Basics

Helm is the **package manager for Kubernetes**: it treats all the objects of an app as one package.

```bash
helm install wordpress
helm upgrade wordpress
helm rollback wordpress
helm uninstall wordpress
```

Install Helm:

```bash
sudo snap install helm --classic
```

- **Revisions**: each change is a snapshot of the app.
- **Metadata**: Helm saves the release metadata as Kubernetes secrets.
- **Templating**: values are injected into the templates.

### Helm Charts

Chart structure:

```text
hello-world-chart/
├── templates/     # templates directory
├── values.yaml    # configurable values
├── Chart.yaml     # chart info
├── LICENSE        # chart license
├── README.md      # readme file
└── charts/        # dependency charts
```

```bash
helm --help
helm search hub wordpress
```

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo remove hashicorp
helm repo list
helm repo update

helm install my-release bitnami/wordpress
helm list
helm uninstall my-release
```

### Customizing Chart Parameters

Download the chart, edit it, then install from the local directory:

```bash
helm pull bitnami/wordpress
helm pull --untar bitnami/wordpress

ls wordpress

helm install my-release ./wordpress
```

### Helm Releases

```bash
helm install my-site bitnami/wordpress
helm install nginx-release bitnami/nginx --version 7.1.0

helm list
helm history nginx-release
helm rollback nginx-release 1

helm upgrade wordpress-release bitnami/wordpress
helm upgrade dazzling-web bitnami/nginx --version 18.3.6
helm rollback dazzling-web
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 12. Kustomize

### Kustomization File

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - db/db-config.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
  - message-broker/rabbitmq-config.yaml
  - message-broker/rabbitmq-depl.yaml
  - message-broker/rabbitmq-service.yaml
  - nginx/nginx-depl.yaml
  - nginx/nginx-service.yaml
```

### Patches

There are two ways to write a patch:

| Type | Style |
| :--- | :--- |
| **JSON 6902** | A list of operations (`op`, `path`, `value`) on a `target` |
| **Strategic merge** | A partial Kubernetes manifest that gets merged into the original |

**JSON 6902, inline** in `kustomization.yaml`:

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /metadata/name
        value: web-deployment
```

In an overlay, `overlays/QA/kustomization.yaml`:

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: caddy
```

```bash
kubectl apply -k /root/code/k8s/overlays/QA
```

**In a separate file**

```yaml
patches:
  - path: replicas-patch.yaml
    target:
      kind: Deployment
      name: nginx-deployment
```

**Strategic merge, inline**

```yaml
patches:
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: api-deployment
      spec:
        replicas: 5
```

### Patching Dictionaries

#### Replace a key

JSON 6902:

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/metadata/labels/component
        value: web
```

Strategic merge:

```yaml
patches:
  - path: label-patch.yaml
```

```yaml
# label-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        component: web
```

#### Add a key

JSON 6902:

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: add
        path: /spec/template/metadata/labels/org
        value: KodeKloud
```

Strategic merge:

```yaml
patches:
  - path: label-patch.yaml
```

```yaml
# label-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        org: kodekloud
```

### Patching Lists

#### Replace an item

JSON 6902:

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0
        value:
          name: haproxy
          image: haproxy
```

#### Add an item

JSON 6902. The `-` at the end of the path appends the new item at the end of the list:

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: add
        path: /spec/template/spec/containers/-
        value:
          name: haproxy
          image: haproxy
```

Strategic merge:

```yaml
patches:
  - path: label-patch.yaml
```

```yaml
# label-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: haproxy
          image: haproxy
```

#### Delete an item

Strategic merge:

```yaml
patches:
  - path: label-patch.yaml
```

```yaml
# label-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - $patch: delete
          name: database
```

### Overlays

```text
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── nginx-depl.yaml
│   ├── service.yaml
│   └── redis-depl.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── config-map.yaml
    ├── stg/
    │   ├── kustomization.yaml
    │   └── config-map.yaml
    └── prod/
        ├── kustomization.yaml
        └── config-map.yaml
```

With subdirectories:

```text
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── db/
│   │   ├── db-depl.yaml
│   │   ├── db-service.yaml
│   │   └── kustomization.yaml
│   └── api/
│       ├── api-depl.yaml
│       ├── api-service.yaml
│       └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   ├── db/
    │   │   ├── db-patch.yaml
    │   │   └── kustomization.yaml
    │   └── api/
    │       ├── api-patch.yaml
    │       └── kustomization.yaml
    └── prod/
        ├── kustomization.yaml
        ├── db/
        │   ├── db-patch.yaml
        │   └── kustomization.yaml
        └── api/
            ├── api-patch.yaml
            └── kustomization.yaml
```

### Components

Reusable pieces of configuration that overlays can opt into:

```text
k8s/
├── base/
│   ├── kustomization.yaml
│   └── api-depl.yaml
├── components/
│   ├── caching/
│   │   ├── kustomization.yaml
│   │   ├── deployment-patch.yaml
│   │   └── redis-depl.yaml
│   └── db/
│       ├── kustomization.yaml
│       ├── deployment-patch.yaml
│       └── postgres-depl.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── premium/
    │   └── kustomization.yaml
    └── standalone/
        └── kustomization.yaml
```

<p align="right"><a href="#top">⬆ back to top</a></p>

---

## 13. Troubleshooting

### Cluster Components

```bash
k get nodes
k get pods
k get pods -n kube-system

service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status
service kubelet status
service kube-proxy status

k logs kube-apiserver-master -n kube-system
sudo journalctl -u <service-name>
```

### Inspecting Multi-Container Pods

```bash
# Print one environment variable in a container
kubectl exec mc-pod -n mc-namespace -c mc-pod-1 -- sh -c 'echo $NODE_NAME'

# Print all environment variables in a container
kubectl exec mc-pod -n mc-namespace -c mc-pod-1 -- env

# Logs of one container
kubectl logs mc-pod -n mc-namespace -c mc-pod-3

# Read a file inside a container
k exec mc-pod -n mc-namespace -c mc-pod-2 -- cat /var/log/shared/date.log
```

<p align="right"><a href="#top">⬆ back to top</a></p>
