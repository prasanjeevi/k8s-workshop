# Kubernetes


### What is Kubernetes?
- Kubernetes (K8s) is an open-source system for container orchestration
- Container Orchestration = automating deployment, scaling, and management of containerized applications
- K8s is infra for modern applications
---
### Container deployment challenges
1. resilency / self-healing
2. scaling
3. networking
4. storage
5. rolling update


## Core Concepts
- Cluster
- Nodes
- Pods
- Services
- Namespaces
- Labels & Selectors
---
### Cluster Architecture
<img src="K8S.png?raw=true" alt="k8s arch">

---
### Nodes 
- Masters aka control plane
- Workers aka compute nodes
---
#### Master node components
  - API Server
  - Scheduler
  - etcd
  - Controller Manager
  - Kubelet
  - Container Runtime
---
#### Worker node components
  - Kubelet
  - KubeProxy
  - Container Runtime
---
### Pods
- Pods are the smallest deployable units of computing that you can create and manage in Kubernetes
- Pods = one or more containers
---
### Services
- An abstract way to expose an application running on a set of Pods as a network service
- Services = Expose Pods, Virtual IP, Load Balancing
---
## Networking
- IP Management
- DNS
- Network plugins
- Network policies


###	Kubectl
```
kubectl get nodes
NAME    STATUS   ROLES    AGE   VERSION
node0   Ready    master   94m   v1.19.1
node1   Ready    none     71m   v1.19.1
node2   Ready    none     65m   v1.19.1
```
```
kubectl get pods -n kube-system
NAME                            READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-lhmdc         1/1     Running   0          85m
coredns-f9fd979d6-txcxz         1/1     Running   0          85m
etcd-node0                      1/1     Running   0          85m
kube-apiserver-node0            1/1     Running   0          85m
kube-controller-manager-node0   1/1     Running   0          85m
kube-proxy-2wwtq                1/1     Running   0          63m
kube-proxy-r4kxk                1/1     Running   0          85m
kube-proxy-rd2fz                1/1     Running   0          57m
kube-scheduler-node0            1/1     Running   0          85m
```
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
---
### Kube Config
```
# cat ~/.kube/config
# %USERPROFILE%\.kube\config
kubectl config view
```
```
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://example.com:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
current-context: kubernetes-admin@kubernetes
```
```
kubectl config use-context CONTEXT_NAME
```
---
### Get Kube Config File
```
az aks get-credentials CLUSTERNAME
gcloud container clusters get-credentials CLUSTER_NAME

tkgi get-credentials CLUSTER_NAME
oc login
```


### Object Management
```
# Read
kubectl get pods
kubectl describe pod webapp
kubectl get pod web-app -o yaml
```
```
# Write
## Imperative
kubectl run --generator=run-pod/v1 web-app --image nginx
kubectl create deployment web-app-deploy --image nginx
kubectl edit pod web-app
kubectl delete pod web-app
kubectl patch pod web-app -p '{"spec":{"containers":[{"name":"web-server","image":"nginx:1.18"}]}}'

## Declarative
kubectl create -f web-app.yaml
kubectl apply -f web-app.yaml
kubectl delete -f web-app.yaml
```
---
### API Resources
```
apiVersion:
kind:
metadata:
spec:
```
```
kubectl api-resources
kubectl api-versions
```


### Simple Pod
```
vi pod.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
```
```
kubectl apply -f pod.yaml
```
---
### Multi-Container Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app-utils
spec:
  containers:
  - name: web-server
    image: nginx
  - name: dnsutils
    image: gcr.io/kubernetes-e2e-test-images/dnsutils:1.3
    command:
      - sleep
      - "3600"
```


### Simple Deployment
```
vi deploy.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app-deploy
  template:
    metadata:
      labels:
        app: web-app-deploy
    spec:
      containers:
      - name: web-server
        image: nginx
```
```
kubectl apply -f deploy.yaml
kubectl create deployment web-app-deploy --image nginx
```


## Service
```
vi svc.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: web-app-service-internal
spec:
  selector:
    app: web-app-deploy
  ports:
    - port: 80
```
```
kubectl apply -f svc.yaml
kubectl expose deployment web-app-deploy --port=80 --name=web-app-service-internal
```
---
### How service works
```
kubectl get pods -o wide
NAME                             READY   STATUS    RESTARTS   AGE     IP                NODE    NOMINATED NODE   READINESS GATES
web-app                          1/1     Running   0          23m     192.168.104.1     node2   none             none  
web-app-deploy-9ff88bb58-6fss6   1/1     Running   0          9m57s   192.168.104.2     node2   none             none  
web-app-deploy-9ff88bb58-lhmw4   1/1     Running   0          9m57s   192.168.166.131   node1   none             none  
web-app-utils                    2/2     Running   0          16m     192.168.166.130   node1   none             none  
```
```
kubectl get svc
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes                 ClusterIP   10.96.0.1       none          443/TCP   5h7m
web-app-service-internal   ClusterIP   10.98.230.230   none          80/TCP    7m6s
```
```
kubectl get ep
NAME                       ENDPOINTS                             AGE
kubernetes                 192.168.1.30:6443                    5h7m
web-app-service-internal   192.168.104.2:80,192.168.166.131:80   6m44s
```
---
### DNS Pod-Service
```
kubectl exec -it web-app-utils -c dnsutils -- sh

nslookup web-app-service-internal
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web-app-service-internal.default.svc.cluster.local
Address: 10.98.230.230

nslookup 192-168-104-2.default.pod.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   192-168-104-2.default.pod.cluster.local
Address: 192.168.104.2
```
---
### Service Types
- ClusterIP (default)
- NodePort
- LoadBalancer
---
### Service Ports
- port (service port)
- targetPort (container port)
- nodePort (30000-32767)
---
### NodePort Service
```
vi svc-node.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  selector:
    app: web-app-deploy
  ports:
    - port: 80
  type: NodePort
```
---
### How to consume service
```
kubectl get svc
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes                 ClusterIP   10.96.0.1        none          443/TCP        5h12m
web-app-service            NodePort    10.110.109.234   none          80:31809/TCP   5s
web-app-service-internal   ClusterIP   10.98.230.230    none          80/TCP         11m
```
```
kubectl get no -o wide
NAME    STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
node0   Ready    master   5h13m   v1.19.1   192.168.1.30   none          Ubuntu 20.04.1 LTS   5.4.0-47-generic   docker://19.3.12
node1   Ready    none     4h50m   v1.19.1   192.168.1.31   none          Ubuntu 20.04.1 LTS   5.4.0-47-generic   docker://19.3.12
node2   Ready    none     4h45m   v1.19.1   192.168.1.32   none          Ubuntu 20.04.1 LTS   5.4.0-47-generic   docker://19.3.12
```
```
# web-app-service-internal
curl 10.98.230.230
# web-app-service
curl 10.110.109.234
curl 192.168.1.30:31809
```
---
### Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shopping-service-ingress
spec:
  rules:
  - http:
      paths:
      - path: /products
        backend:
          serviceName: product-service
          servicePort: 80
  - http:
      paths:
      - path: /orders
        backend:
          serviceName: order-service
          servicePort: 80
```
https://kubernetes.io/docs/concepts/services-networking/ingress/
---
### Headless Service
```
apiVersion: v1
kind: Service
metadata:
  name: web-app-headless-service
spec:
  selector:
    app: web-app-sts
  ports:
    - port: 80
  clusterIP: None
```
```
nslookup web-app-headless-service
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web-app-headless-service.default.svc.cluster.local
Address: 192.168.166.136
Name:   web-app-headless-service.default.svc.cluster.local
Address: 192.168.104.20
```


## Scheduling
---
### Manual Scheduling / No Scheduler
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
  nodeName: node1
```
---
### Multiple Schedulers
```
kubectl get pod kube-scheduler-node0 -n kube-system -o yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
  schedulerName: default-scheduler
```
---
### Resources Requests and limits
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
    resources:
      requests:
        cpu: "500m"
        memory: "64Mi"
      limits:
        cpu: 1
        memory: "128Mi"
```
- 1 CPU unit = 1 vCPU/Core or 1 hyperthread | 1CPU = 1000m | 500m = 0.5CPU


### Probes
The kubelet uses
- liveness probes to know when to restart a container
- readiness probes to know when a container is ready to start accepting traffic
- startup probes to know when a container application has started.
---
#### Pod with Probes
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/


## Replication
- Deployment
- ReplicaSet
- ReplicationController
- DaemonSet
- StatefulSet
---
### Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app-deploy
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: web-app-deploy
    spec:
      containers:
      - name: web-server
        image: nginx
        imagePullPolicy: Always
```
---
### Replica Set
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-app-deploy-7465998875
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app-deploy
      pod-template-hash: "7465998875"
  template:
    metadata:
      labels:
        app: web-app-deploy
        pod-template-hash: "7465998875"
    spec:
      containers:
      - name: web-server
        image: nginx
```
---
### Rolling update & Rollback
```
kubectl rollout history deployment web-app-deploy

# nginx 1.19.2(latest)
kubectl apply -f deploy.yaml

# nginx 1.18
kubectl set image deployment/web-app-deploy web-server=nginx:1.18 --record

# update deploy.yaml image nginx 1.17.10
kubectl apply -f deploy.yaml --record

kubectl rollout undo deployment web-app-deploy --to-revision=1
```
---
### Daemon Set
```
kubectl get ds kube-proxy -n kube-system -o yaml
```
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-proxy
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: kube-proxy
  template:
    metadata:
      labels:
        k8s-app: kube-proxy
    spec:
      containers:
        name: kube-proxy
        image: k8s.gcr.io/kube-proxy:v1.19.1
```


### ConfigMaps & Secrets
---
#### ConfigMaps
```
kubectl create configmap web-app-config --from-literal=APPNAME=MyPortal --from-literal=LOGO=company.png
kubectl create configmap db-config --from-literal=DBPORT=3306 --from-literal=POOLSIZE=10
```
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-app-config
data:
  APPNAME: MyPortal
  LOGO: company.png
```
---
### Use ConfigMap in Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
    env:
    - name: TITLE
      valueFrom:
        configMapKeyRef:
          name: web-app-config
          key: APPNAME
    envFrom:
    - configMapRef:
        name: db-config
```
---
#### Secrets
```
kubectl create secret generic web-app-secret --from-literal=APIKEY=API2020X --from-literal=SALT=Web@2020
kubectl create secret generic db-secret --from-literal=DBPASSWORD=DB2021X --from-literal=DBSALT=Db@2020
```
```
apiVersion: v1
kind: Secret
metadata:
  name: web-app-secret
type: Opaque
data:
  APIKEY: QVBJMjAyMFg=
  SALT: V2ViQDIwMjA=
```
---
### Use Secret in Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
    env:
    - name: APPSALT
      valueFrom:
        secretKeyRef:
          name: web-app-secret
          key: SALT
    envFrom:
    - secretRef:
        name: db-secret
```


## Storage
- Volumes
- PersistentVolume
---
### Volumes - emptyDir
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
    volumeMounts:
    - name: cache-volume
      mountPath: /cache
  volumes:
  - name: cache-volume
    emptyDir: {}
```
---
### Volumes - hostPath
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
    volumeMounts:
    - name: shared
      mountPath: /usr/share/nginx/html
  volumes:
  - name: shared
    hostPath:
      path: /node-data
      type: Directory
```
- Type = Directory | DirectoryOrCreate | File | FileOrCreate
---
### ConfigMap as Volume
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
    - name: web-server
      image: nginx
      volumeMounts:
        - name: config-vol
          mountPath: /config
  volumes:
    - name: config-vol
      configMap:
        name: web-app-config
```
---
### Secret as Volume
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
    - name: web-server
      image: nginx
      volumeMounts:
        - name: secret-vol
          mountPath: /vault
  volumes:
    - name: secret-vol
      configMap:
        name: db-config
```
---
### Persistent Volumes
- StorageClass
- PersistentVolume
- PersistentVolumeClaim
---
### Persistent Volume Claim
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: web-app-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
```
---
### Persistent Volume
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: samplepv
spec:
  capacity:
    storage: 200Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /persist
```
---
### Use PVC in Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: /usr/share/nginx/html
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: web-app-pvc
```
---
### Stateful Set
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web-app-sts
spec:
  selector:
    matchLabels:
      pp: web-app-sts
  serviceName: "nginx"
  replicas: 2
  template:
    metadata:
      labels:
        app: web-app-sts
    spec:
      containers:
      - name: web-server
        image: nginx
        volumeMounts:
        - name: static
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: static
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 100Mi
```


## Security
- Authentication
  - TLS Certificates
  - ServiceAccounts
- Authorization
  - Role & Rolebinding
  - CluserRole & ClusterRolebinding
- Secrets
---
### Certificate Signing Request
```
openssl genrsa -out john.key 4096
openssl req -new -key john.key -out john.csr
CSR_FILE_BASE64=$(cat john.csr | base64 | tr -d "\n")
```
```
cat <&lt;EOF >> john-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  groups:
  - system:authenticated
  request: $CSR_FILE_BASE64
  usages:
  - client auth
EOF
```
```
kubectl get csr
kubectl certificate approve john
kubectl get csr john -o jsonpath='{.status.certificate}' | base64 -d > john.crt
kubectl config set-credentials john --client-key=john.key --client-certificate=john.crt --embed-certs=true
```
---
### Role
```
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - create
  - get
  - list
  - update
  - delete
```
---
### Role Binding
```
kubectl create rolebinding developer-john --role=developer --user=john
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-john
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: john
```
---
### Cluster Role
```
kubectl create clusterrole cluster-admin --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=nodes
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - create
  - get
  - list
  - update
  - delete
```
---
### Cluster Role Binding
```
kubectl create clusterrolebinding cluster-admin-bob --clusterrole=cluster-admin --user=bob
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-bob
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: bob
```
---
### Security Context
```
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web-server
    image: nginx
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
```
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#securitycontext-v1-core
- https://man7.org/linux/man-pages/man7/capabilities.7.html
---
### Network Policy
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```
https://kubernetes.io/docs/concepts/services-networking/network-policies/


## Logging & Monitoring
```
# NODES -> STATUS
kubectl get nodes

# PODS -> READY, STATUS, RESTARTS
kubectl get pods -n kube-system
kubectl get pods -A

# POD LOGS -> /var/log/pods/
kubectl log web-app
kubectl logs web-app-utils -c web-server
kubectl log web-app -p

systemctl status kubelet
journalctl -u kubelet
kubectl cluster-info dump

kubectl top
https://github.com/kubernetes-sigs/metrics-server
```


## Troubleshooting
```
kubectl exec -it pod web-app -- sh
kubectl logs web-app-utils -c dnsutils
kubectl describe # -> Conditions, Events, Termination Message, Probes
kubectl get events

ImagePullBackOff -> Wrong image / Registry access issues
CrashLoopBackOff -> Container crashing after launch
RunContainerError -> Missing ConfigMaps/Secrets/Volumes
PodExceedsFreeMemory -> Insufficient memory on the node
PodExceedsFreeCPU -> Insufficient CPU on the node
OOM Killed -> Out of memory
```
