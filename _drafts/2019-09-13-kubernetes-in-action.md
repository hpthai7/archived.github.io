---
layout: post
title: Notes on Kubernetes 
tags: [jekyll, syntax]
categories:
- blog
---

* 1. [Installation](#Installation)
* 2. [Pods: a single deployable unit](#Pods:asingledeployableunit)
	* 2.1. [Pods](#Pods)
	* 2.2. [YAML](#YAML)
	* 2.3. [Label: for pod organization](#Label:forpodorganization)
	* 2.4. [Annotations](#Annotations)
	* 2.5. [Namespace to group resource](#Namespacetogroupresource)
* 3. [Replication controller](#Replicationcontroller)
	* 3.1. [Pod health](#Podhealth)
	* 3.2. [Replication controller: rarely used today](#Replicationcontroller:rarelyusedtoday)
		* 3.2.1. [Change pod label manually](#Changepodlabelmanually)
		* 3.2.2. [Change pod template](#Changepodtemplate)
		* 3.2.3. [Scaling](#Scaling)
	* 3.3. [ReplicaSet = ReplicationController + expressive pod selectors](#ReplicaSetReplicationControllerexpressivepodselectors)
	* 3.4. [DaemonSet: Run exactly one pod in each node](#DaemonSet:Runexactlyonepodineachnode)
	* 3.5. [Running pods that perform a single completable task](#Runningpodsthatperformasinglecompletabletask)
	* 3.6. [Cron Jobs](#CronJobs)
* 4. [Services](#Services)
	* 4.1. [Intro](#Intro)
	* 4.2. [Test service](#Testservice)
	* 4.3. [Local access to service](#Localaccesstoservice)
	* 4.4. [Connecting to services living outside cluster](#Connectingtoserviceslivingoutsidecluster)
		* 4.4.1. [Service endpoints](#Serviceendpoints)
		* 4.4.2. [Creating an FQDN for an external service instead of endpoints](#CreatinganFQDNforanexternalserviceinsteadofendpoints)
	* 4.5. [Exposing services to external clients](#Exposingservicestoexternalclients)
		* 4.5.1. [Using NodePort service](#UsingNodePortservice)
		* 4.5.2. [Using External Load Balancer service](#UsingExternalLoadBalancerservice)
		* 4.5.3. [Ingress service](#Ingressservice)
	* 4.6. [Readiness probe](#Readinessprobe)
	* 4.7. [Headless service](#Headlessservice)
* 5. [Volumes](#Volumes)
	* 5.1. [Share date between containers in one pod: emptydir/gitrepo](#Sharedatebetweencontainersinonepod:emptydirgitrepo)
	* 5.2. [Access files on worker's node system: hostpath](#Accessfilesonworkersnodesystem:hostpath)
	* 5.3. [Storage's encapsulation: PersistentVolumes and PersitentVolumeClaims](#Storagesencapsulation:PersistentVolumesandPersitentVolumeClaims)
* 6. [ConfigMaps & Secrets: configure applications](#ConfigMapsSecrets:configureapplications)
	* 6.1. [Configuring containerized applications](#Configuringcontainerizedapplications)
		* 6.1.1. [Passing CLI arguments to containers](#PassingCLIargumentstocontainers)
		* 6.1.2. [Setting environment variables for containers](#Settingenvironmentvariablesforcontainers)
		* 6.1.3. [Mounting config files via volumes](#Mountingconfigfilesviavolumes)
	* 6.2. [Decoupling configuration with ConfigMaps](#DecouplingconfigurationwithConfigMaps)
		* 6.2.1. [Update configMap without restarting app](#UpdateconfigMapwithoutrestartingapp)
	* 6.3. [Passing sensitive data to container using Secrets](#PassingsensitivedatatocontainerusingSecrets)
		* 6.3.1. [Default Secret](#DefaultSecret)
		* 6.3.2. [Using Secret in Pod](#UsingSecretinPod)
		* 6.3.3. [How image pull Secrets](#HowimagepullSecrets)
* 7. [Access pod metadata/resources from application](#Accesspodmetadataresourcesfromapplication)
* 8. [Deployments](#Deployments)
* 9. [StatefulSets](#StatefulSets)
* 10. [HELM](#HELM)
* 11. [Bibliography](#Bibliography)
	* 11.1. [Load Balancing](#LoadBalancing)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->Kubernetes

# Kubernetes

`YAML` and `CLI tool` co-exist in Kubernetes 

```bash
## pod
kubectl get pods/po
kubectl get nodes/no
# pod's yaml output 
kubectl get pods -o wide
kubectl get po XXX -o yaml
kubectl get services/svc
kubectl get rc
kubectl describe service kubia
# expose LoadBalancer service to external
kubectl expose rc kubia --type=LoadBalancer --name kubia-http
# get external IP address
minikube service kubia-http
# scale service
kubectl scale rc kubia --replicas=3
# get dashboard
kubectl cluster-info | grep dashboardlogistics
minikube dashboard # get dashboard
# discover possible API object fields
kubectl explain pods
kubectl explain pod.spec
# create pods from external files
kubectl create -f sample-pod.yaml
kubectl create -f sample-pod.json
# delete pods by sending SIGTERM (--> SIGKILL)
kubectl delete po kubia-gpu
kubectl delete po -l creation_method=manual
# delete all pods in namespace
kubectl delete po --all
# delete both pods AND namespace = delete namespace
kubectl delete ns custom-namespace
# delete all resources in namespace (rs, pods, services... except Secrets)
kubectl delete all --all

## log
kubectl logs sample-pod
kubectl logs sample-pod -c sample-container
# log previously terminated pod
kubectl logs sample-pod --previous
# [debugging] send requests to pods using port forwarding from localhost port to container port
# ... Forwarding from [::1]:8888 -> 8080
# curl localhost:8888
kubectl port-forward kubia-manual 8888:8080
## labeling
# add label at runtime
kubectl label po kubia-manual creation_method=manual
# override port at runtime
kubectl label po kubia-manual-v2 env=debug --overwrite
# list pods by labels
kubectl get po -L creation_method,env
# filter pods by labels
kubectl get po -l creation_method=manual
kubectl get po -l env
kubectl get po -l '!env'
kubectl get po -l creation_method!=manual
kubectl get po -l env in (prod,devel)
kubectl get po -l env notin (prod,devel)
kubectl get po -l app=pc,rel=beta
# label for node -> selectively schedule pods into nodes by certain criteria
kubectl label node gke-kubia-85f6-node-0rrx gpu=true
## annotations
kubectl annotate pod kubia-manual mycompany.com/someannotation="foo bar"

## namespace
kubectl get ns
kubectl get po -n kube-system
kubectl get po --namespace kube-system
# create namespace
kubectl create namespace custom-namespace
# create namespace using yaml
kubectl create -f custom-namespace.yaml
# to get resource in namespace, adding '-n custom-namespace' to every command
kubectl create -f kubia-manual.yaml -n custom-namespace # create pods in namespace
# or set current context's namespace to expected namespace
kubectl config set-context $(kubectl config currentcontext) --namespace expected-namespace
kubectl create -f kubia-manual.yaml # hassle-free command to get resource in expected-namespace

## rc
kubectl get rc
kubectl describe rc kubia-rc
# configure editor for yaml in .bashrc
export KUBE_EDITOR="/usr/bin/nano"
# edit visually yaml
kubectl edit rc kubia
# horizontal scaling
kubectl scale rc kubia --replicas=10
# deleting rc leaves pods unmanaged and running # possible to recreate rc to manage running pods
kubectl delete rc kubia --cascade=false

## rs
kubectl get rs
kubectl describe rs
# delete rs leads to deleting pods
kubectl delete rs kubia

## persistent volumes/claims
kubectl get pv
kubectl get pvc
```
##  1. <a name='Installation'></a>Installation
Minikube is used for tests. Remember to deactivate Secure Boot in BIOS (admin password needed) to solve virtualbox modprov activation failure.
```bash
The vboxdrv kernel module is not loaded. Either there is no module
         available for the current kernel (4.4.0-47-generic) or it failed to
         load. Please recompile the kernel module and install it by
 ```
In case proxy:
- https://kubernetes.io/docs/setup/learning-environment/minikube/#using-minikube-with-an-http-proxy
- Add minikube ip (ie. 192.168.99.100) to no_proxy/NO_PROXY configs in /etc/environment and relog-in
```bash
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
http_proxy="http://135.245.192.7:8000/"
https_proxy="http://135.245.192.7:8000/"
ftp_proxy="http://135.245.192.7:8000/"
no_proxy="localhost,127.0.0.1,localaddress,.localdomain.com,nsn-net.net,nsn-intra.net,192.168.99.100"
HTTP_PROXY="http://135.245.192.7:8000/"
HTTPS_PROXY="http://135.245.192.7:8000/"
FTP_PROXY="http://135.245.192.7:8000/"
NO_PROXY="localhost,127.0.0.1,localaddress,.localdomain.com,nsn-net.net,nsn-intra.net,192.168.99.100"
```
- Enable autocompletion in zsh: https://kubernetes.io/fr/docs/reference/kubectl/cheatsheet/#auto-compl%c3%a9tion-avec-kubectl

##  2. <a name='Pods:asingledeployableunit'></a>Pods: a single deployable unit
###  2.1. <a name='Pods'></a>Pods
All pods in any nodes are connected to a shared flat network
Pod logs are rotated on daily basis
Port forwarding from port in node/VM to port in pod:

![Alt text](/resources/images/k8s/1566767084429.png)

- Attention: all containers in one pod share one port pool
###  2.2. <a name='YAML'></a>YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual-v2
  labels:
    creation_method: manual
    env: prod
  annotations:
    kubernetes.io/created-by: |
      {"kind":"SerializedReference", "apiVersion":"v1",
      "reference":{"kind":"ReplicationController", "namespace":"default", ...
spec:
  nodeSelector:
    gpu: "true"
    kubernetes.io/hostname: "hostname" # not recommended
  containers:
  - image: luksa/kubia
    name: kubia
    livenessProbe:
      httpGet:
        path: /
        port: 8080
    ports:
    - containerPort: 8080
	  protocol: TCP
```

###  2.3. <a name='Label:forpodorganization'></a>Label: for pod organization
Label by service type (app=frontend,backend)
Label by release type (rel=canary, beta)

###  2.4. <a name='Annotations'></a>Annotations
Similar to labels but meant for adding long info

###  2.5. <a name='Namespacetogroupresource'></a>Namespace to group resource
- Avoid resource name conflicts, when resources need to be divided, i.e. into different deployment envs dev/staging/prod
- Resources are unique in namespace
- Node resource is `global` and `not` tied to a single namespace
- Namespace does not provide network/hardware/... isolation, 

```yaml
apiVersion: v1
kind: Namespace
metadata:
name: custom-namespace
```

##  3. <a name='Replicationcontroller'></a>Replication controller
- Create pods directly with CLI: containers are recreated in case container error, but not in case node error
- Create pods with RCs or Deployments, in case node error, pods are recreated

###  3.1. <a name='Podhealth'></a>Pod health
Apps see memory error/deadlocks/loops but the process is still on -> Kubernetes does not know pod's internal status -> failure
- Liveness probe -> recreate container if failure
	- server is responding
	- check only app's internal state
	- be lightweight
	- no loops
- Readiness probe

###  3.2. <a name='Replicationcontroller:rarelyusedtoday'></a>Replication controller: rarely used today

![Alt text](./images/luksa-pods-flat-network.png)

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    app: kubia
template:
  metadata:
    labels:
      app: kubia
  spec:
    replicas: 3
    containers:
	  - name: kubia
        image: luksa/kubia
        ports:
        - containerPort: 8080
```

####  3.2.1. <a name='Changepodlabelmanually'></a>Change pod label manually

Change the label manually to remove the pod from Replica Controller i.e in case a malfunctioning pod needs debugging. RC's label can be modified as well.

####  3.2.2. <a name='Changepodtemplate'></a>Change pod template

![Alt text](/resources/images/k8s/luksa-controller-rc-pod-template-changes.png)

```bash
# configure editor for yaml in .bashrc
export KUBE_EDITOR="/usr/bin/nano"
# edit visually yaml
kubectl edit rc kubia
```

Edition of pod template invokes afterward suppression of old pods and automatic creation of new pods. There are better ways in chap 9.

####  3.2.3. <a name='Scaling'></a>Scaling

- Manual scaling `kubectl scale rc kubia --replicas=3`
- YAML scaling

```yaml
spec:
  replicas: 3
```

###  3.3. <a name='ReplicaSetReplicationControllerexpressivepodselectors'></a>ReplicaSet = ReplicationController + expressive pod selectors

- rc selector: label=singular
- rs selector: label1=value1, label2=value2; label=*

```yaml
apiVersion: apps/v1beta2 # specific API version
kind: ReplicaSet
metadata:
	name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia
    matchExpressions:
      - key: app
        operator: In/NotIn/Exists/DoesNotExist
        values:
          - kubia
            kubi
  template:
    metadata:
      labels:
        app: kubia # can we skip label here?
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
```

###  3.4. <a name='DaemonSet:Runexactlyonepodineachnode'></a>DaemonSet: Run exactly one pod in each node

Log collector, resource monitoring, 

![Alt text](./images/luksa-controller-daemon-set.png)

![Alt text](./images/luksa-controller-daemon-set-label.png)

```yaml
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
  spec:
    nodeSelector:
      disk: ssd # deploy to nodes having this label
    containers:
    - name: main
      image: luksa/ssd-monitor
```

###  3.5. <a name='Runningpodsthatperformasinglecompletabletask'></a>Running pods that perform a single completable task

###  3.6. <a name='CronJobs'></a>Cron Jobs

##  4. <a name='Services'></a>Services

###  4.1. <a name='Intro'></a>Intro

![Alt text](./images/luksa-service-intro.png)

Two ways:
- `expose` command (to expose rc...)
- `kubectl create -f kubia-service.yaml`.  `Service`'s definition with reference to ports by `names`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  ports:
    - name: http
      port: 80 # port 80 is mapped to the container’s port called http.
      targetPort: http
    - name: https
      port: 443
      targetPort: https
  selector:
    app: kubia # include all pods having this label
```

Use in conjunction with `Pod`'s definition:

```yaml
kind: Pod
spec:
  containers:
  - name: kubia
    ports:
    - name: http # container’s port 8080 is called http
      containerPort: 8080
    - name: https
      containerPort: 8443
```

Example services:

```yaml
$ kubectl get svc
NAME CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubia 10.111.249.153 <none> 80/TCP 6m
```

###  4.2. <a name='Testservice'></a>Test service

- Send requests to service's cluster IP
- `curl` to cluster IP in nodes
- `kubectl exec` in pods

![Alt text](./images/luksa-service-test.png)

```bash
kubectl create -f kubia-service.yaml
kubectl get svc
kubectl exec kubia-7nog1 -- curl -s http://10.111.249.153
```

Use sessionAffinity: ClientIP to make all requests made by a certain client to be redirected to the same pod every time

```yaml
apiVersion: v1
kind: Service
spec:
  sessionAffinity: ClientIP
```

###  4.3. <a name='Localaccesstoservice'></a>Local access to service

- Service's cluster IP is detectable by manual inspection of pods. Kubernetes uses environment variables of service for pod's creation if the service is created beforehand (`Question: is this an endpoint?`)

```bash
$ kubectl exec kubia-3inly env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubia-3inly
KUBERNETES_SERVICE_HOST=10.111.240.1
KUBERNETES_SERVICE_PORT=443
...
KUBIA_SERVICE_HOST=10.111.249.153
KUBIA_SERVICE_PORT=80
```

- Using DNS to access service: the DNS server attached to `cluster?!?` (by declaring FQDN)

```bash
$ kubectl exec -it kubia-3inly bash
root@kubia-3inly:$ ping kubia-http # service
root@kubia-3inly:$ curl http://kubia.default.svc.cluster.local:8080
root@kubia-3inly:$ curl http://kubia.default:8080
```

###  4.4. <a name='Connectingtoserviceslivingoutsidecluster'></a>Connecting to services living outside cluster

####  4.4.1. <a name='Serviceendpoints'></a>Service endpoints

```bash
$ kubectl describe svc kubia
$ kubectl get endpoints kubia
NAME ENDPOINTS AGE
kubia 10.108.1.4:8080,10.108.2.5:8080,10.108.2.6:8080 1h
```

- Service <-> Endpoints <-> Pods
- Endpoints are also a resource, created from i.e. selectors
- Using selectors to have endpoints managed automatically, or:
- Manually configured endpoints -> more flexible endpoints and service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service # name of service must match name of Endpoints object
spec: # This service has no selector defined
  ports:
  - port: 80
```

external-endpoints.yaml

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service # name of service must match name of Endpoints object
subsets:
  - addresses:
    - ip: 11.11.11.11 # IPs of the endpoints that service will forward connections to
    - ip: 22.22.22.22
    ports:
    - port: 80 # The target port of the endpoints
```

![Alt text](./images/luksa-service-ext-endpoints.png)

####  4.4.2. <a name='CreatinganFQDNforanexternalserviceinsteadofendpoints'></a>Creating an FQDN for an external service instead of endpoints

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: someapi.somecompany.com
  ports:
  - port: 80
```

Service is available at external-service.default.svc.cluster.local for access from internals

###  4.5. <a name='Exposingservicestoexternalclients'></a>Exposing services to external clients

![Alt text](./images/luksa-service.png)

####  4.5.1. <a name='UsingNodePortservice'></a>Using NodePort service

- Reserve **one** specific port on **all** nodes.
- Requests coming to this node can be forwarded to other nodes to find the right pod
- If one node fails, the request to that node fails -> use LoadBalancer service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-nodeport
spec:
  type: NodePort
  externalTrafficPolicy: Local # not forwarding requests # pro: lower network delay # con: unbalance load balancing
  ports:
  - port: 80 # port of service’s internal cluster IP
    targetPort: 8080 # target port of pod
    nodePort: 30123 # port of each cluster node
  selector:
    app: kubia
```

![Alt text](./images/luksa-service-nodeport.png)

Access NodePort service in minikube:

```bash
minikube service <service-name> [-n <namespace>]
```

####  4.5.2. <a name='UsingExternalLoadBalancerservice'></a>Using External Load Balancer service

- Only in case cloud infrastructure provides load balancers
- Is an extension of NodePort service
- **IaaS** creates load balancer and write its IP to service

![Alt text](./images/luksa-service-loadbalancer.png)

####  4.5.3. <a name='Ingressservice'></a>Ingress service
- LoadBalancer service occupies one load balancer -> ineffective
- Ingress operates at HTTP level -> one ingress fits all

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: kubia.example.com # This Ingress maps the kubia.example.com domain name to your service.
    http:
      paths:
      - path: /
        backend:
          serviceName: kubia-nodeport # specify the nodeport service behind ingress
          servicePort: 80 # All requests / will be sent to port 80 of the kubia-nodeport service
```

![Alt text](./images/luksa-service-ingress.png)

And send request to Ingress:

```bash
$ curl --noproxy "*" kubia.example.com
```

###  4.6. <a name='Readinessprobe'></a>Readiness probe

- When pod is ready for connections

###  4.7. <a name='Headlessservice'></a>Headless service

- Whenever clients need to connect to all pods, headless service allows discovering 
DNS lookup for pod's IP

##  5. <a name='Volumes'></a>Volumes

###  5.1. <a name='Sharedatebetweencontainersinonepod:emptydirgitrepo'></a>Share date between containers in one pod: emptydir/gitrepo

Emptydir (and gitrepo) are temporary

![Alt text](./images/luksa-volumes-emptydir.png)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune
spec:
  containers:
  - image: luksa/fortune
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    emptyDir: {} # temporary volume for two containers
```

###  5.2. <a name='Accessfilesonworkersnodesystem:hostpath'></a>Access files on worker's node system: hostpath

- Hostpath is persistent storage
- Exists on the node itself
- Not appropriate for node-independent pods

![Alt text](./images/luksa-volumes-hostpath.png)

###  5.3. <a name='Storagesencapsulation:PersistentVolumesandPersitentVolumeClaims'></a>Storage's encapsulation: PersistentVolumes and PersitentVolumeClaims

- PersistentVolumes are `cluster`-level resource like nodes
- PersitentVolumeClaims are `namespace`-level resource like pods

![Alt text](./images/luksa-volumes-persistent-volumes.png)

![Alt text](./images/luksa-volumes-persistent-volumes-cluster-level.png)

Persistent Volume's manifest:

```YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi
  accessModes: # It can either be mounted by a single client for reading and writing or by multiple clients for reading only.
  - ReadWriteOnce
  - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain # After the claim is released, the PersistentVolume should be retained (not erased or deleted).
  gcePersistentDisk: # The PersistentVolume is backed by the GCE PersistentDisk you created earlier.
    pdName: mongodb
    fsType: ext4
```

Claim's manifest:

```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc # use when using the claim as the pod’s volume.
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce # You want the storage to support a single client (performing both reads and writes).
  storageClassName: "" # dynamic provisioning
```

Pod's manifest:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:
    claimName: mongodb-pvc # Referencing the PersistentVolumeClaim by name in the pod volume
```

**Recycling PersistentVolumes** based on different policies:
- Retain: keep volume data
- Delete: delete underlying storage
- Recycle: delete volume content


##  6. <a name='ConfigMapsSecrets:configureapplications'></a>ConfigMaps & Secrets: configure applications

###  6.1. <a name='Configuringcontainerizedapplications'></a>Configuring containerized applications

####  6.1.1. <a name='PassingCLIargumentstocontainers'></a>Passing CLI arguments to containers

```yaml
# Dockerfile
FROM ubuntu:latest
RUN apt-get update ; apt-get -y install fortune
ADD fortuneloop.sh /bin/fortuneloop.sh
ENTRYPOINT ["/bin/fortuneloop.sh"]
CMD ["10"]
```

Passing commands and arguments in Dockerfile:
- ENTRYPOINT (executable command) vs CMD (argument passed to ENTRYPOINT)
- ENTRYPOINT forms:
  - Shell form: `ENTRYPOINT node app.js`, creates 2 procs (`/bin/sh -c cmd` & `cmd`) 
  - **Exec form**: `ENTRYPOINT ["node", "app.js"]`, creates 1 proc (node app.js)

Overriding commands and arguments in Kubernetes:

```YAML
kind: Pod
spec:
  containers:
  - image: some/image
    command: ["/bin/command"]
    args: ["arg1", "arg2", "arg3"]
```

Inside `/bin/command.sh`:

```bash
#!/bin/bash
trap "exit" SIGINT
INTERVAL=$1
echo Configured to generate new fortune every $INTERVAL seconds
mkdir -p /var/htdocs
while :
do
  echo $(date) Writing fortune to /var/htdocs/index.html
  /usr/games/fortune > /var/htdocs/index.html
  sleep $INTERVAL
done
```

![Alt text](./images/luksa-config-docker-kubernetes-equivalence.png)

####  6.1.2. <a name='Settingenvironmentvariablesforcontainers'></a>Setting environment variables for containers

```bash
#!/bin/bash
trap "exit" SIGINT
echo Configured to generate new fortune every $INTERVAL seconds
mkdir -p /var/htdocs
while :
do
  echo $(date) Writing fortune to /var/htdocs/index.html
  /usr/games/fortune > /var/htdocs/index.html
  sleep $INTERVAL
done
```

```js
// JavaScript
process.env.INTERVAL
```

```python
# Python
os.environ['INTERVAL']
```

```java
// Java
System.getenv("INTERVAL")
```

```yaml
# Kubernetes
kind: Pod
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL
      value: "30"
    name: html-generator
```

####  6.1.3. <a name='Mountingconfigfilesviavolumes'></a>Mounting config files via volumes

Drawback: hard-coded env var in `yaml` is not practical for different deployment environments.

###  6.2. <a name='DecouplingconfigurationwithConfigMaps'></a>Decoupling configuration with ConfigMaps
- key/value pairs
- values of literals or config files
- linked to environment variables or mounted volumes
- namespace-level resource -> same config name for different env/namespaces

![Alt text](./images/luksa-configmap-reference.png)

![Alt text](./images/luksa-configmap-namespace-level.png)

```bash
kubectl create configmap fortune-config --from-literal=sleep-interval=25
kubectl get configmap fortune-config -o yaml
kubectl delete configmap fortune-config
kubectl create -f fortune-config.yaml
kubectl create configmap my-config --from-file=config-file.conf
kubectl create configmap my-config
  --from-file=foo.json
  --from-file=configmap-files # i.e. nginx configs
  --from-file=bar=foobar.conf
  --from-file=directory/
  --from-literal=some=thing
```

ConfigMap value types:

![Alt text](./images/luksa-configmap-values.png)

Refer to individual keys in config-map.yaml:

```YAML
apiVersion: v1
data:
  sleep-interval: "25"
kind: ConfigMap
metadata:
  name: fortune-config
  namespace: default
```

```YAML
# pod
apiVersion: v1
kind: Pod
metadata:
  name: fortune-env-from-configmap
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL # You’re setting the environment variable called INTERVAL.
      valueFrom:
        configMapKeyRef: # initializing it from a ConfigMap key
          name: fortune-config
          key: sleep-interval
    args: ["$(INTERVAL)"] # in case using env var in Docker image's ENTRYPOINT
```

Refer to entire configMap:

```yaml
spec:
  containers:
  - image: some-image
    envFrom:
    - prefix: CONFIG_
      configMapRef:
        name: my-config-map
```

**Expose config map entries as `files` using `configMap` volume:**

![Alt text](./images/luksa-configmap-directory.png)

```bash
kubectl create configmap fortune-config --from-file=configmap-files
```

```yaml
# kubectl get configmap fortune-config -o yaml
apiVersion: v1
data:
my-nginx-config.conf:
  server {
    listen 80;
    server_name www.kubia-example.com;
    gzip on;
    gzip_types text/plain application/xml;
    location / {
      root /usr/share/nginx/html;
      index index.html index.htm;
    }
  }
  sleep-interval:
    25
kind: ConfigMap
...
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune-configmap-volume
spec:
  containers:
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    ...
    - name: config
      mountPath: /etc/nginx/conf.d # all files are here
      readOnly: true
    ...
  volumes:
  ...
  - name: config # refers to fortune-config configMap
    configMap:
      name: fortune-config
```

It's possible to:
- pass configMap entry to container as environment variable
- pass all entries as environment variable at once
- pass entry as command line argument
- mount volume to expose configMap entries as files
- mount volume to expose *individual* configMap entry as file
- mount individual configMap entries as files without hiding other files in the destination directory
- set file permissions

####  6.2.1. <a name='UpdateconfigMapwithoutrestartingapp'></a>Update configMap without restarting app

When you update a ConfigMap, the files in all the volumes referencing it are updated. It’s then up to the process to detect that they’ve been changed and reload them. But Kubernetes will most likely eventually also support sending a signal to the container after updating the files.

> WARNING It takes a surprisingly long time for the files to be updated after you update the ConfigMap (up to one whole minute). TODO check again.

```bash
kubectl edit configmap fortune-config
kubectl exec fortune-configmap-volume -c web-server
  cat /etc/nginx/conf.d/my-nginx-config.conf
kubectl exec fortune-configmap-volume -c web-server -- nginx -s reload
```

> WARNING If app in pods does NOT supports dynamic reloading of configMap configs, modifying configMap is not recommended.

> WARNING One big caveat relates to updating ConfigMap-backed volumes. If you’ve mounted a single file in the container instead of the whole volume, the file will not be updated! At least, this is true at the time of writing this chapter.

###  6.3. <a name='PassingsensitivedatatocontainerusingSecrets'></a>Passing sensitive data to container using Secrets
- Secrets are secured version of ConfigMaps
- Secrets are available to related nodes only
- Secrets are encrypted in etcd
- Secrets are Base64-encoded in inspection result (~~encrypted~~)
- Secrets can contain plain text data

- Max size 1MB

```bash
kubectl create secret generic fortune-https --from-file=https.key --from-file=https.cert --from-file=foo
kubectl get secrets
```

####  6.3.1. <a name='DefaultSecret'></a>Default Secret
Per-pod secrets are to secure communication between pod and K8s API server.

![Alt text](./images/luksa-default-secrets.PNG)

####  6.3.2. <a name='UsingSecretinPod'></a>Using Secret in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune-https
spec:
  containers:
  - image: luksa/fortune:env
    name: html-generator
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: fortune-config
          key: sleep-interval
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    - name: config # mount the configMap to directory
      mountPath: /etc/nginx/conf.d
      readOnly: true
    - name: certs # mount the cert volume to directory
      mountPath: /etc/nginx/certs/
      readOnly: true
    ports:
    - containerPort: 80
    - containerPort: 443
  volumes:
  - name: html
    emptyDir: {}
  - name: config # define configMap volume
    configMap:
      name: fortune-config
      items:
      - key: my-nginx-config.conf
        path: https.conf
  - name: certs # define secret volume fortune-https
    secret:
    secretName: fortune-https
```

![Alt text](./images/luksa-secret-mount-in-pod.png)

####  6.3.3. <a name='HowimagepullSecrets'></a>How image pull Secrets
- Create Secret holding docker credentials
- Reference Secret in `imagePullSecrets` field of pod manifest
- Drawback: Secret is added to all pod manifests. Solution: add Secret to ServiceAccount (chap 12)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
spec:
  imagePullSecrets:
  - name: mydockerhubsecret
  containers:
  - image: username/private:tag
    name: main
```

##  7. <a name='Accesspodmetadataresourcesfromapplication'></a>Access pod metadata/resources from application
- How a pod’s name, namespace, and other metadata can be exposed to the process either through environment variables or files in a downwardAPI volume
- How CPU and memory requests and limits are passed to your app in any unit
the app requires
- How a pod can use downwardAPI volumes to get up-to-date metadata, which
may change during the lifetime of the pod (such as labels and annotations)
- How you can browse the Kubernetes REST API through kubectl proxy
- How pods can find the API server’s location through environment variables or
DNS, similar to any other Service defined in Kubernetes
- How an application running in a pod can verify that it’s talking to the API
server and how it can authenticate itself
- How using an ambassador container can make talking to the API server from
within an app much simpler
- How client libraries can get you interacting with Kubernetes in minutes

##  8. <a name='Deployments'></a>Deployments
Different ways:
- Delete all pods and create new ones: service interrupted
- Start new ones and delete old ones: new version must be backward-compatible
- Rolling update: zero-downtime, using:
  - ReplicationController or ReplicaSet: drawbacks:
    - kubectl client carry out the task
    - K8s itself modifies controller's label (discretion advised)
  - Deployment

![Alt text](./images/luksa-deployment-delete-create.png)

![Alt text](./images/luksa-deployment-blue-green.png)

![Alt text](./images/luksa-deployment-rolling-update.png)

![Alt text](./images/luksa-deployment.png)

**ReplicationController/ReplicaSet:**

Deploy rc/rs and roll out updates:

```bash
 kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2
 ```

Cons: create two rc/rs at the same time

**Deployment**

Two rs/rc needs a coordinator: deployment:

```yaml
apiVersion: apps/v1beta1 # apps API group, version v1beta1.
kind: Deployment
metadata:
  name: kubia #no need to include version in name of Deployment.
spec:
  replicas: 3
  template:
    metadata:
      name: kubia
      labels:
      app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
```

![Alt text](./images/k8s-deployment-strategies.png)


##  9. <a name='StatefulSets'></a>StatefulSets

##  10. <a name='HELM'></a>HELM

##  11. <a name='Bibliography'></a>Bibliography

###  11.1. <a name='LoadBalancing'></a>Load Balancing

![Alt text](./images/load-balancing-osi.png)

![Alt text](./images/load-balancing-difference.png)

![Alt text](./images/load-balancing-types.png)