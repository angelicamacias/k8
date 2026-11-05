# k8

## Commands

### Replica Set

```
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
  replicas: 3
  selector: 
    matchLabels: 
      type: front-end
```

```
kubectl create -f replicaset-definition.yaml
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl scale --replicas=6 replicaset myapp-replicaset
kubectl get replicaset
kuebectl delete replicaset myapp-replicaset
kubectl replace -f replicaset-definition.yaml
k get rs
```

### Deployment

```
apiVersion: apps/v1
kind: Deployment
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
  replicas: 3
  selector: 
    matchLabels: 
      type: front-end
```
```
kubectl get deployment
kubectl create -f deployment-definition.yaml
kubctl get all (to see all pods replicasset deployments)
```

### Generate YAML templates

Using the kubectl run command can help in generating a YAML template.

Create an NGINX Pod
```
kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)
```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

Create a deployment
```
kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.
```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

```
kubectl create -f nginx-deployment.yaml
```
OR

In k8s version 1.19+, we can specify the –replicas option to create a deployment with 4 replicas.

```
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

### Service

- Nodeport: 

 ```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports: 
  - targetPort:80
    port: 80
    nodePort:30008

  selector: 
    app: myapp
    type: front-end
```
```
kubectl create -f service-definition.yaml
kuebctl get services
```

- cluster ip: 
```
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports: 
  - targetPort: 80
    port: 80
    nodePort: 30008

  selector: 
    app: myapp
    type: back-end
```

- LoadBalancer 
```
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: LoadBalancer
  ports: 
  - targetPort:80
    port: 80
    nodePort:30008

  selector: 
    app: myapp
    type: back-end
```


### NameSpace

db-service.dev.svc.cluster.local 

db-service = service name
dev = namespace
svc = service 
cluster.local = domain 

```
kubectl get pods --namespace=kube-system 
kubectl crate -f pod.definition.yaml --namespace=dev
kubectl expose pod redis --port=6379 --name=redis-service --type=ClusterIP
```

add namespace in a pod

```
apiVersion: v1
kind: Pod

metadata:
  name: myapp-pod
  namespace: dev <---- namespace
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx
      image: nginx
```

namespace definition file

```
apiVersion: v1
kind: Namespace 
metadata: 
  name: dev
```

```
kubectl create -f namespace-dev.yaml
kubectl create namespace dev
kubectl config set-contect $(kubectl config current-context) --namespace=dev
kubectl get pods --all-namespace
k get namespace
```

resource quota

```
apiVersion: v1
kind: Namespace 
metadata: 
  name: dev
  namespace: dev

spec: 
  hard: 
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

### Exam tips 

```
k run --image=nginx nginx
k create deployment --image=nginx nginx
k expose deployment nginx --port 80
k edit deployment nginx 
k scale deployment nginx --replicas=5
k set image deployment nginx nginx=nginx:1.18
```

api resourcces

```
kubectl api-resources
k explain pods 
k explain pods.spec
k explian pods --recursive
kubectl run custom-nginx --image=nginx --port=8080
```

### Manual scheduling 

manual scheduling in a pod

```
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

```
k get pods --watch
```

### Labels and slectors
pod:
```
apiVersion: v1
kind: Pod
metada: 
  name: simpple-webapp
  labels: 
    app: App1
    function: fron-end
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports: 
      - containerPort: 8080
```

```
k get pods --selectror app=App1
k get all --selector env=prod
k get pods --selector env=prod --no-headers | wc -l 
k get all --selector env=prod,bu=finance,tier=frontend
```

replicaset

```
apiVersion: apps/v1
kind: ReplicaSet
metada: 
  name: simple-webapp
  labels:
    app: App1
    functions: front-end
  annotations:
    buildverson: 1.34
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: App1
   template:
     metada:
       labels:
         app: app1
         function: front-end
    spec:
      containers: 
      - name: simple-webapp
        image: simple-webapp
```

### Tains and rolerations 

```
k taint nodes node-name key=value:taint-effect
```
taint-effect: what happens to POD's that do not tolerate this taint? 
 - noshcedule 
 - preferNoschedule
 - enexecute


```
k taint node node1 app=blue:NoSchedule
```

```
apiVersion v1
kind: Pod
metada: 
  name: myapp-pod
  spec:
    containers: 
    - name: nginx-container
      image: nginx
    tolearions: 
      - key: app
        operator: "Equal"
        value: blue
        effect: NodeSchedule 
```

### Node selector

```
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

```
k label nodes node-1 size=Large 
```

### Node Affinity 

```
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

node affinity types: 

- requiredDuringSchedulingIgnoredDuringExecution 
- preferredDuringSchedulingIgoneredDuringExecution


```
kubectl edit deployment blue
```

### Resources requests

resources pod 

```
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
        memory: "4Gi"
        cpu: 2
      limits: 
        memory: "2Gi" 
        cpu: 2 
```

what happen when a pod exceed limits? 
pod terminated with:
oom ---> out of memory 


### LimitRange
- cpu
```
apiVersion: v1
kind: LimitRange
metadata: 
  name: cpu-resource-constraint
spec: 
  limits:
  - default: 
      cpu: 500m 
    defaultRequests: 
      cpu: 500m
    max: 
      cpu: 1
    min:
      cpu: 100m
    type: Container
```

- memory
```
apiVersion: v1
kind: LimitRange
metadata: 
  name: memory-resource-constraint
spec: 
  limits:
  - default: 
      memory: 500m 
    defaultRequests: 
      memory: 500m
    max: 
      memory: 1
    min:
      memory: 100m
    type: Container
```

### Resource Quotas

```
apiVersion: v1
kind: ResourceQuota
metadata: 
  name: my-resource-quota
spec: 
  hard: 
    requets.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```

### DaemonSet

```
apiVersion: apps/v1
kind: DaemonSet
metadata: 
  name: monitoring-deamon
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

```
k create -f daemon-set.yaml
k get daemonsets
k get daemonsets --all-namespaces
k describe deamonsetskubectl describe daemonset kube-flannel-ds --namespace=kube-flannel

```



kubectl create deployment elasticsearch --image=registry.k8s.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml


### Static pods

What is the path of the directory holding the static pod definition files? 
 -> /etc/kubernetes/manifests

 First, let's identify the node in which the pod called static-greenbox is created. To do this, run:
controlplane:~# kubectl get pods --all-namespaces -o wide  | grep static-greenbox
default       static-greenbox-node01                 1/1     Running   0          19s     10.244.1.2   node01       <none>           <none>
controlplane:~#
From the result of this command, we can see that the pod is running on node01.


Next, SSH to node01 and identify the path configured for static pods in this node.

Important: The path need not be /etc/kubernetes/manifests. Make sure to check the path configured in the kubelet configuration file.
controlplane:~# ssh node01 
node01:~# ps -ef |  grep /usr/bin/kubelet 
root        4147       1  0 14:05 ?        00:00:00 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9
root        4773    4733  0 14:05 pts/0    00:00:00 grep /usr/bin/kubelet

node01:~# grep -i staticpod /var/lib/kubelet/config.yaml
staticPodPath: /etc/just-to-mess-with-you

node01:~# 
Here the staticPodPath is /etc/just-to-mess-with-you


Navigate to this directory and delete the YAML file:
node01:/etc/just-to-mess-with-you# ls
greenbox.yaml
node01:/etc/just-to-mess-with-you# rm -rf greenbox.yaml 
node01:/etc/just-to-mess-with-you#
wait for 30 seconds.

### PRIORITY CLASSES

```
k get priorityclass
```

- priority class create: 

```
apiVersion: shceduling.k8s.io/v1
kind: PriorityClass
metadata: 
  name: high-priority   <---
value: 1000000000
description: "Priority class for mission critial pods"
```

to define the prority in a pod we set the priority in the pod manifest: 

```
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
  priorityClassName: high-priority  <---
```


```
k get priorityclass
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
```

### Multiple Schedulers

```
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles: 
- shcedulerNmae: my-scheduler 
```

scheduler pod: 

```
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
   image: k8s.gcr.io/kube-scheduelr-amd64:v1.xx.x
   name: kube-system
```


pod definition 

```
apiVersion: v1
kind: Pod
metadata: 
  name: nginc
spec: 
  containers: 
  - name: nginx
    image: nginx
  schedulerName: my-custom-scheduler
```

see scheduler logs 
```
k get my-consum-scheduler --namespace=kube-system
```

### Logging and monitoring 

```
k top node
k top pod
```

- App logs 

```
k logs -f <podName>

```

### Rollout command

```
k rollout status deployment/myapp-deployment
k rollout history deployment/myapp-deployment
```

- deployment strategy 

--> rolling update 

```
k rollout undo deployment myapp-deployment
```

### Commands and arguments (docker)

```
apiVersion: vi
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

entrypoint -> sleep2.0
CMD -> 10 


ENTRYPOINT-> command -> The main executable/binary to run

CMD	-> args	 -> The default arguments passed to that executable

### Env variables 

```
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
1: create configMap

    - imperative way
```
k create configmap <config-name> \
  --from-literal=<key>=<value>\
```

```
k create configmap app-config \
  --from-literal=APP_COLOR=blue\
  --from-literal=APP_MOD=prod
```

    - declarative

```
k create -f 
```

```
apiVersion: v1
kind: ConfigMap
metadata: 
  name: app-config
data: 
  APP_COLOR: blue
  APP_MODE: prod
```

```
k create -f config-map.yaml
```


2: inject into pod
```
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

```
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

- imperative 
```
k create secret generic
```

```
k create secret generic <config-name> \
  --from-literal=<key>=<value> 
```


```
k create secret generic app-secret \
  --from-literal=DB_Host=mysql
```

from a file
```
k create secret generic app-secret \
  --from-file=<path-to-file>
```

```
k create secret generic app-secret \
  --from-file=app_secret.properties
```


- declarative
```
k create -f 
```

```
apiVersion: v1
kind: Secret
metadata: 
  name: app-secret
data: 
  DB_Host: mysql 
  DB_User: root
  DB_Password: passwrd
```

```
k create -f secret-data.yaml
```

```
echo -n 'mysql' | base64
```

```
k get secrets
k describe secrets
k get secret app-secret -o yaml
```

-- Env 
pod define: 

```
apiVersion: v1
kind: Pod 
metadata: 
  name: simple-webapp-color
lablels: 
  name: simple-webapp-color
spec: 
  containers: 
  - name: simple-webapp-color
    image: simple-webapp-color
    ports: 
      - containerPort: 8080
    envFrom: 
      - secretRef: 
        name: app-secrets
```

```
k create -f pod-definition.yaml
```






-- single ENV
pod define: 

```
apiVersion: v1
kind: Pod 
metadata: 
  name: simple-webapp-color
lablels: 
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


-- volume 



```
apiVersion: v1
kind: Pod 
metadata: 
  name: simple-webapp-color
lablels: 
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

### Multi-container pods

- co-located containers: no hability to find which one starts first
- regular init containers
- sidecar container: specify wich the starting order 


- co-located containers: 
```
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

- regular init containers:
```
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
    image: busyboc
    command: 'wait-for-db-to-start.sh'
  - name: api-checker
    image: busybox
    command: 'wait-for-another-api.sh'
```


- sidecar containers:
```
apiVersion: v1
kind: pod
metatada: 
  name: simple-webap
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
    command: 'setup-log-shipper.sh'
    restartPolicy: Always 
```

```
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

```
kubectl -n elastic-stack logs app -c sidecar
```

see logs in the containers: 

```
kubectl -n elastic-stack exec app -- cat /log/app.log
```


### Autoscaling 



scaling cluster infra
- Manual 
    - horizontal: kubeadmjoin 
- automated
    - horizontal: cluster autoscaler

scaling workloads
- manual 
  hoizontal: k scale ..
  vertical: k edit ...
- automated
    - horizontal: horizontal pods autoscaler(HPA)
    - vertival: Vertical pod autoscaler (VPA)


### Horizonal pod autoscaler

```
k top pod my-app-pod

k scale deployment my-app --replicas=3

k autoscale deployment my-app \
  --cpu-percent=50 --min=1 --max=10

k get hpa

k delelete hpa
```

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: 
  name: my-app-hpa
spec: 
  scaleTargetRef: 
    apiVersion: apps/v1
    kind: Deloyment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics: 
   - type: Resource
     resource: 
       name: cpu 
       target: 
         tyep: Utilization 
         averageUtilization: 50
```


```
kubectl autoscale deployment nginx-deployment --max=3 --cpu=80%
```


### In-place po resizing 

```
apiVersion: apps/v1
```

### Vertical pos autoscaling 

```
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
            cpu: "1"
          limits: 
            cpu: "500m"
```

```
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
        miniAllowed: 
          cpu: "250m"
        macAllowed: 
          cpu: "2"
        controlledResources: ["cpu"]
```
- vpa admission controller
- vpan updater 
- vpa recommender 

```
k describe vpa my-app-vpa
```

### Os upgrades
- pod eviction timeout 


```
k drain node-1
k uncordon node-1
k cordon node-2
```

### Kubernetes Release

```
kubeadm upgrade plan 
kubeadm upgrade apply
```

### Backup and restore mathods


### Authentication 

- User auth mechanisms
    - static token files (is not a recommenden approche)
    - certificates
        - tls certificates
           garatie trust in two partys 
           - private key 
             server.key 
             server.key.pem
           - publick lock 
             server.crt
             client.pem
             server.pem
           - public key infrastructure (PKI)
           ---
           - root certificates: config in the CA server
           - client certificates: config on the clients
           - server certificates: conig in the servers
         
    - identity services


### TLC in k8
- kube-apiserver(http server to manage the k8 components): 
    apiserver.crt: 
    apiserver.key:
- etcd server (store info about our cluster)
    etcd-x-server.crt
    etcd-x-server.key
- kubelete server (http server to talk with kubeapiserver to interca with the woker nodses)
    kubelete.crt
    kubelete.key
- client compunents (us):
    amdin.cr
    admin.key
- kube-scheduler (look for pods to kubeapiserver)
    scheduler.crt 
    amdin.key
- kube-controller-manager (acces kube-apiserver)
    controller-manager.crt
    controller-manager.key 
- kube-proxy (acces to kube-apiserver)
    kube-proxy.crt
    kube-procy.key 


-----

cert info: 

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml

openssel x509 -in /etc/kubernetes/pki/apiserver.crt - text -noout


journalctl -u etcd.service -l

kubectl logs etcd-master

crictl ps -a

crictl logs <containerID>
```

### Create certificateSigningRequest Object


1. user create a key 

```
open ssl genrsa -out jane.key 2048
```

2. send the request to the admin 

```
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr 
.
.
.
```

3. the admin take the key and create an certificat signing request object 

```
apiVersion: certificates.k8.io/v1
kind: CertificateSigningRequest
metadata: 
  name: jane-csr
spec: 
    singerName: Kubernetes.io/kube-apisercer-client
    expirationSeconds: 600
    usages: 
    - clent auth 
    request: <BASE64_ENCODED_CSR>
    conditions: 
    - tye: Approved
      status: "True"
      resons: KubectlApprove
      message: "This CSR was approved by kubectl certificate approve" 
      lastUpdateTime": "2026-02-03T11:46:10Z"
```


```
cat jane.csr | base64
.
.
.
```

4. List and approve cert 

```
k get csr

k certificate approve jane

k get csr jane -o yaml 

kubectl certificate deny agent-smith

k delete csr agent-smith
```

the controllor manager is the responsible for the certs 
    - csr-approving 
    - csr-signing

```
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```


### Security kubeconfig 

```
k get pods 
  --server my-kube-playground:6443
  --client-key admin.key
  --client-certificate admin.crt
  --certificate-authority ca.crt
```

config file $HOME/.kube/config:

```
k get pods --kubeconfig config
```
file
```
  --server my-kube-playground:6443
  --client-key admin.key
  --client-certificate admin.crt
  --certificate-authority ca.crt
```

config file have three section 

- clusters
    development
    productions
    google
- contect 
    admin
    dev user
    prod user
- user 
    admin@procuction 
    dev@google


```
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

user: 
    - name: my-kube-admin 
      user: 
        client-certificate: admin.crt
        client-key: admin.key 
```

```
k config view
k config view -kubeconfig=my-custom-config
k config use-context prod-user@production 
k config -h
```

### Api Groups 

- kube-proxy 
    enable connectivity between pods and services across different nodes in the cluster. 

- kubectl proxy 
    http proxy service created by kubectl utiity to access the kube-apiserver 


### Authorization 

- node authorizer: part of the system:nodes required for a kubelet 
- ABAC: attribure base access control 
    asosiates a user with a set a permisstions with a policy file 
- RBAC: role based access control 
    role with some permitions and asociated the developers to that role 
    more standar approche 
- webhook 

alwaysallow
alwaysdeny

```
--authorization-mode=Node,RBAC,Webhook
```

- Role based access control (RBAC)

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
    name: developer
rules: 
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","create","update","delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```

```
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

```
k get roles

k get rolebindings

k describe role developer 

k describe rolebinding devuser-developer-binding


k auth can-i create deployments 

k auth can-i delete nodes

k auth can-i create deployments --as dev-user

k auth can-i create pods --as dev-user

k auth can-i create pods --as dev-user --namespace test
```


access to a spesific pods

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
    name: developer
roles: 
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get","create","delete"]
      resourceName: ["blue","orange"]
```


apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
    name: developer
rules: 
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","create","delete"]



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




apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2026-10-13T05:04:13Z"
  name: dev-user-binding
  namespace: blue
  resourceVersion: "1579"
  uid: cb5d11f8-0cc1-4d3c-bedb-c0e179ea04fe
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: dev-user



#
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2026-10-13T05:04:13Z"
  name: developer
  namespace: blue
  resourceVersion: "1578"
  uid: ae737765-226c-44c7-8e50-a366fddad6c0
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


### cluster role 

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata: 
    name: cluster-administrator
rules: 
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get","list","create","update","delete"]
```

```
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


### service account
service account

```
k get serviceaccount

k describe serviceaccount default 

k describe pod my-kubernates-dashboard


```


projected volum: dinamic directory created inside the pod, created automaticlly 

mounted in: /var/run/secrets/kubernetes.io/serviceaccount 

```
~> k exec -it my-kubernates-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount

ca.crt namespace token 
```

costume service account: 

cli:
```
k create serviceaccount dashboard-sa
```

file: 
```
apiVersion:v1
kind: ServiceAccount
metadata: 
  name: dashboard-sa
  namespace: default
automountServiceAccountToken: false (remove the automounted sa token)
```

asociated the service account to the pod

```
apiVersion: v1
kind: Pod
metadata: 
  name: my-kubernates-dashboard
spec: 
  containers: 
    - name: my-kuberntes-dashboard
      image: my-kubernates-dashboard
  serviceAccountName: dashboard-sa
  automountServiceAccountToken: false (remove the automounted sa token)
```

```
k deescribe pod my-kubernates-dashboard


k exec -it my-kubernates-dashboard ls /var/run...
```


```
k create token dashboard-sa

k create token dashboard-sa --duration 2h

k create serviceaccount dashboard-sa

```


deployment configured to use a projected volume: 

```
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


```
kubectl exec $(kubectl get pod -l name=web-dashboard -o jsonpath='{.items[0].metadata.name}') -- ls /var/run/secrets/kubernetes.io/serviceaccount/
```

### Image

to have access to the repositories we create a secretes

```
k create secret docker-registry private-reg-cred --docker-server=myprivateregistry.com:5000 --docker-username=dock_user --docker-password=dock_password --docker-email=registry-user@org.com
```


then we pass that secret to the pod definition 

```
apiVersion: v1
kind: Pod
metadata: 
  name: nginx-pod
spec: 
  containers: 
  - name: nginx
    image: private-registry.io/apps/internal-app
  imagePullSecrets: 
  - name: regcred
```

### Traffic 

Ingress: recibe repond 

Egress: deliver reponde 

### Network security 

-> network policy 
the policy is only applied to the pod where it was set it up 

```
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
    - podsSelector: 
        matchLabels: 
        name: api-pod
    - namespaceSelector: 
        matchLabels: 
         kubernetes.io/metadata.name: prod
    - ipBlock: 
      cidr: 192.168.5.10/32
    ports: 
    - protocol: TCP 
        PORT: 3306
```
```
k create -f policy-definition.yaml 
```


```
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
    - podsSelector: 
        matchLabels: 
        name: api-pod
    ports: 
    - protocol: TCP 
        PORT: 3306
  egress: 
  - to: 
    - ipBlock: 
      cidr: 192.168.5.10/32
    ports: 
    - protocol: TCP 
      port: 80
```

```
kubectl get netpol
```


### Custom Resource Definition 

It is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation.

- custom resource
- custom controler 


custom resource definiton (CRD)

```
apiVersion: apiextension.k8.io/v1
kind: CustomResourceDefinition 
metada: 
  name: flighttickets.flights.com 
spec: 
    scop: Namespace 
    group: flights.com 
    names: 
      kind: FlighTicket
      singular: flightticket
      plural: flighttickets
      shortNames:
        -ft
      version: 
         - name: v1
           served: true
           storage: true
           schema: 
             openAPIV3Schema: 
              type: object
              properties: 
                spec: 
                  tyep: objec
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

- custom controller 

writh some code to create the controller to look for changes in etcd to monitor the CRD 

### operator fremwork 

file system: 

docker store data info by defalut here: 

```
/var/lib/docker 
    /aufs
    /containers
    /image
    /volumes
```

how docker stores the info? 

```
docker build Dockerfile -f mmushad/my-custom-app
```

1: base ubunto layer 
2: changes in apt packages
3: change sin pip packages
4: source code 
5: update entrypoint 

when you update a docker images, docker use the cache, and bc of that is faster 

- volumes

```
docker volume create data_volume
```

var/lib/docker
   /volumnes
   /data_volume

- volume mount 
```
docker run -v data_volume:/var/lib/mysql mysql 
```

- bind mount 
to change the dafualt dir volume in the image: 

```
docker run -v /data/mysql:/va/lib/mysql mysql 
```

new way 

```
docker run /
  --mount type=bind,source=/data/mysql,targer=/var/lib/mysql mysql 
```

-> stores drivers
- AUFS
- ZFS
- BTRFS
- Device Mapper
- Overlay 
- Overlay2

### Docker volumes

Volumnes are not handled by storage drivers, volumes are handel by volume plugin 

```
docker run -it \
  --name mysql 
  --volume-driver rexay/ebs
  --mount src=ebs-vol,target=/var/lib/mysql
  mysql 
```

### Container Storage Interface

container networking interface = CNI 
container storage interface = CSI 



helm install topolvm topolvm --version 17.2.0 --namespace topolvm-system --create-namespace --values <file>


### volumes in k8

volume to the pod to remian the data even if the pod is deleted

storage in a host's directory 
```
apiVersion: v1
kind: Pod
metadata: 
  name: random-number-generator 
spec: 
  containers: 
  - image: alpine
    name: alpine 
    command: ["/bin/sh","-c"]
    args: ["suf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts: 
    - mountPath: /opt
      name: data-volume
  volumes: 
  - name: data-volume
    hostPath: 
      path: /data
      type: Directory 
```


### Persistent volumes 

```
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv-vol1
spec: 
  accessModes: 
  - ReadWriteOnce
  cpacity: 
    storage: 1Gi
  awsElasticBlockStore: 
    volumeID: <volume-id>
    fsType: ext4
```

```
k create -f pv-definition.yaml

k get persistentvolume
```

peristen volume: admin creates it
persistent colume claims: the user creates it to use the storage



```
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: myclaim 
spec: 
  accessModes: 
  - ReadWriteOnce
  resources: 
    requests: 
      storae: 500Mi
```

```
apiVersion: v1
kind: PersistentColume 
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


```
k get persistentvolumeclaim
```

```
k delete peristentvolumeclaim myclaim
```


persistentVolumeReclaimPolicy: Retain 

Retain: keeps pv and it's data
delte: deletes pv 

Once you create a PVC use it in a Pod definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this:


```
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


```
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
   persistentVolumeClaim:
    path: /var/log/webapp
    type: Directory
```



```
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

```
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


 volumes:
  - name: webapp
   persistentVolumeClaim:
    claimName: claim-log-1

### Storage Class

- Static Provisioning: when you provition the volume manually 

- Dynamic provisioning: 
  with storage class 


```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: 
  name: google-storage
 provisioner: kubernetes.io/gce-pd
```

```
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
```
apiVersion: v1
kind: Pod
metadata: 
  name: random-number-generator 
spec: 
  containers: 
  - image: alpine
    name: alpine 
    command: ["/bin/sh","-c"]
    args: ["suf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts: 
    - mountPath: /opt
      name: data-volume
  volumes: 
  - name: data-volume
    persistentVolumeClaim: 
      claimName: myclaim 
```

the pv is created automatilly by the storage class

```
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

```
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

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: 
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

### Networking 

- networking prerequisites 

```
ip link 

ip addr

ip addr add 192.168.1.10/24 dev eth0

ip route

route 

ip route add 192.168.1.0/24 via 192.168.2.1

cat /proc/sys/net/ipv4/ip_forward
```

### Pod networing 

every pod should have an ip adress
every pod should be able to communicate with every other pods in the same node
every pod shoul be able to communicate with wvery other pod on other nodes without NAT

```
ip link add v-net-0 type brigde 

ip addr add 192.168.15.5/24 dev v-net-0 

ip link set veth-red netns red 

ip -n red link set veth-red up 

ip netns exec clue ip route add 192.168.1.0/24 via 192.168.15.5 

ip link set dev v-net-0 up 

ip link add veth-red type veth peer name veth-red-br

ip -n red addr add 192.168.15.1 dev veth-red

ip link set veth-red-br master v-net-0

iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE 
```

cin automate the routing using an script with an add section and a delete section 


### CNI in k8 

conteiner run time: responsible of create components 

the cni is installed in: /opt/cni/bin 

but the cni plugins are in: /etc/cni/net.d
            - bridge.conflist
            - flannel.conflist


#### IPM - IP addres management 


### Service Networking 

services are a cluster wide concept 
it exist in all the nodes in a cluster 


```
k get pods -o wide 

k get service 

kube-api-server --service-cluster-ip-range ipNet (Default: 10.0.0.0/24)

ps aux | grep kube-api-server
```



### Core Dns in kubernetes

core dns is deployed in pods 

kubelete manage to created autoamtilly the reoslution of core dns

to do that it create the file: 

/etc/resolv.conf
nameserver      10.96.0.10


cat /var/lib/kubelet/config.yaml 
...
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
...





cambiar infra solo cuando hay cambios de la infra

si cdkpuede desplegar al mismo tiempo, pasar el pipeline de pasar cloud 2026 y que corrarn en paralelo

el cdk no porque se necesita la infra antes 

infra y luego las tres al mismo tiempo

que deployee la app que se haya cambiado 

no se tiene que mandar todo siempre 


so we need to find out a way to deploy the infra and then the other 3 things in paraller , we do not need to change everthing always

cambiar infra solo cuando hay cambios de la infra

si cdkpuede desplegar al mismo tiempo, like: https://app.harness.io/ng/account/avaa2RG-SuCKMBrVkODAgA/module/ci/orgs/InStrideHQ/projects/SRE/pipelines/Cloud_22026_Promotion_Deploy_Code/pipeline-studio/?repoName=harness-run&connectorRef=account.instride_dev_github_connector&storeType=REMOTE&branch=main

the harness triggers are pretty habilidozos so check that 


### Ingress

```
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

### Getway 

- ingress
- load balancing
- service mesh API's

---

infra providers: gatway class

```
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata: 
  name: nginx
spec: 
  controllerName: example.com/gatewary-controller
```

cluster operators: gateway

```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata: 
  name: nginx-gateway
  namespace: nginx-gateway
spec: 
  gatewayClassName: nginx
  listerners: 
  - name: http 
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All
```

app develper: HTTPRoute TCPRoute

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata: 
  name: exmaple-httproute
spec: 
  parentRefs: 
  - name: exmaple-gateway
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

### Kubeadm 

kubeadm: 

1 multiples sistems: 1 master 2 worker nodes
2 kubeadm tool 
3 inicialates the master server
4 pod network
5 join nodes to the master node 



### Helm 

package manager of k8

see the whole think a packages 

```
helm install wordpress
helm upgrade wordpress
helm hollback wordpress
helm uninstall wordpress
```

installing helm 

sudo snap install helm --classic
  
snapshot fo the app
meta data about data: save the matadata as k8 secrets 

templatidng 


### helm chart

helm chart structure

```
- hello-world-chart
  /templates #templates directory 
  values.yaml #configurable values
  chart.yaml #chart info
  LINCENSE #chart license
  README.md #readme file
  /charts #dependency charts
```

```
helm --help

helm search wordpress

helm search hub wordpress
```

```
helm repo add bitnami https://charts.bitnami.com/bitnami 

helm repo remove hashicorp

helm install my-release bitnami/wordpress

helm list

helm uninstall my-release

helm repo
helm repo list 
helm repo update

```


### customizing chart parameters

```
helm pull b itnami/wordpress

helm pull -untar bitnami/wordpress


ls wordpress 
....

helm install my-release ./wordpress
```

### helm release
```
helm install my-site bitnami/wordpress

helm install nginx-release bitnami/nginx --version 7.1.0

helm list

helm history nginx-release

helm rollback nfinx-release 1 

helm upgrade wordpress-release bitnami/wordpress 

helm upgrade dazzling-web bitnami/nginx --version 18.3.6

helm rollback dazzling-web
```



primero hacer un pipeline que corramos por fuera, piensar en lo que la ai me ayuda 

si se cerea en el pipelne the release y no jala

correrlo en unpipeline fura 

#### Kustomization 



```
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

- patches json6902

in line, it means saved in kustomization.yaml file:
```
patches: 
  - target: 
    kind: Deployment 
    name: api-deployment 

    patch: |-
      - op: replace
        path: /metadata/name
        value: web-deployment
```

or:

overlays/QA/kustomization.yaml
```
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: caddy
```

kubectl apply -k /root/code/k8s/overlays/QA

separate file
```
patches: 
  - path: replicas-patch.yaml
    target: 
      kind: Deployment
      name: nginx-deployment
```

- stragic merge patch 

```
patches: 
  - patch: |- 
      apiVersion: apps/v1
      kind: Deployment
      metadata: 
        name: api-deployment
      spec: 
        replicas: 5
```

- praches dictionary 

json 6902

```
patches: 
  - target: 
    kind: Deployment
    name: api-deploymment 
    patch: |-
      - op: replace 
        patch: 
        /spec/template/metadata/labels/component
        value: web
```

strategic memrge patch 

```
patches: 
  - label-patch.yaml 
```
```
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

- add json 6902

```
patches: 
  - target: 
    kind: Deployment
    name: api-deployment 
    patch: |-
      - op: add
      path: /spec/template/metadata/lavels/org
        value: KodeKloud
```

- strategic merge patch

strategic memrge patch 

```
patches: 
  - label-patch.yaml 
```
```
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

- list json6902

```
patches: 
  - target: 
    kind: Deployment
    name: api-deployment 
    patch: |-
      - op: replace
      path: /spec/template/spect/containers/0
        value:  
          name: haproxy
          image: haproxy
```

```
patches: 
  - target: 
    kind: Deployment
    name: api-deployment 
    patch: |-
      - op: add
      path: /spec/template/spect/containers/-
        value:  
          name: haproxy
          image: haproxy
```

- = put the new at the end of the list 

- strategic merge patch

strategic memrge patch 

```
patches: 
  - label-patch.yaml 
```
```
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: api-deployment
spec: 
  template: 
    spect: 
      containers: 
        - name: haproxy 
          image: haproxy
```
- strategic merge patch delete
```
patches: 
  - label-patch.yaml 
```
```
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: api-deployment
spec: 
  template: 
    spect: 
      containers: 
        - $patch: delete 
          name: database
```


- overlays structure

```
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

```
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

- components

```
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



---

k get nodes

k get pods 

k get pods -n kube-system 

service kube-apiserver status

service kube-controller-manager status

service kube-scheduler status

service kubelet status

service kube-proxy status

k logs kube-apiserver-master -n kube-system 

sudo journalctl -u

---

Note

check env variables in a pod: 
```
kubectl exec mc-pod -n mc-namespace -c mc-pod-1 -- sh -c 'echo $NODE_NAME'
```

```
kubectl exec mc-pod -n mc-namespace -c mc-pod-1 -- env
```

```
kubectl logs mc-pod -n mc-namespace -c mc-pod-3
```

```
k exec mc-pod -n mc-namespace -c mc-pod-2 -- cat /var/log/shared/date.log
```