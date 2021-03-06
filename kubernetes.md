---
useful links
* [main tutorial](https://kubernetes.io/docs/tutorials/)
* [git source](https://github.com/kubernetes/dashboard)
* [architecture](https://kubernetes.io/docs/reference/glossary/?architecture=true)
* [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [tutorial](https://kubernetes.io/docs/tutorials/)
* [tutorial external](https://academy.level-up.one/p/learn-kubernetes-from-a-devops-guru-kubernetes-docker)
* [interactive course](https://www.katacoda.com/courses/kubernetes)
* [interactive course](https://www.katacoda.com/contino/courses/kubernetes)
* [interactive course helm](https://www.katacoda.com/aptem/scenarios/helm)
* [docs task by section](https://kubernetes.io/docs/tasks/)
* [k8s examples](https://github.com/wardviaene/kubernetes-course)
* [playground](https://labs.play-with-k8s.com/)
* [playground](https://www.katacoda.com/courses/kubernetes/playground)
* [ibmcloud](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started)
* [troubleshooting](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)
* some tricks
  * completion
  ```bash
  source <(kubectl completion bash)
  source <(kubectl completion zsh)
  ```
  * trace logging
  ```bash
  rm -rf ~/.kube/cache
  kubectl get pods -v=6
  kubectl get pods -v=7
  kubectl get pods -v=8
  ```
  * explain yaml schema
  ```
  kubectl explain pods
  kubectl explain pods --recursive
  kubectl explain pods --recursive --api-version=autoscaling/v2beta1
  ```
* python client
  ```bash
  pip install kubernetes
  ```
---
# Architecture
![architecture](https://i.postimg.cc/RFpnbwgc/k8s-architecture.png)
![architecture](https://i.postimg.cc/ZnbpGZzJ/k8s-architecture-overview.png)
![architecture](https://i.postimg.cc/6pKPX9KY/k8s-architecture-overview.png)
![nodes with software](https://i.postimg.cc/QCHz6vqH/k8s-architecture.png)
![kubernetes](https://i.postimg.cc/CL1Z9Lnv/kubernetes.png)

---
# workflow
![deployment workflow](https://i.postimg.cc/bvJn06Tz/update-workflow.png)

1. The user deploys a new app by using the kubectl CLI. Kubectl sends the request to the API server.
2. The API server receives the request and stores it in the data store (etcd). After the request is written to the data store, the API server is done with the request.
3. Watchers detect the resource changes and send notifications to the Controller to act on those changes.
4. The Controller detects the new app and creates new pods to match the desired number of instances. Any changes to the stored model will be used to create or delete pods.
5. The Scheduler assigns new pods to a node based on specific criteria. The Scheduler decides on whether to run pods on specific nodes in the cluster. The Scheduler modifies the model with the node information.
6. A Kubelet on a node detects a pod with an assignment to itself and deploys the requested containers through the container runtime, for example, Docker. Each node watches the storage to see what pods it is assigned to run. The node takes necessary actions on the resources assigned to it such as to create or delete pods.
7. Kubeproxy manages network traffic for the pods, including service discovery and load balancing. Kubeproxy is responsible for communication between pods that want to interact.

---
# [k3s - Lightweight Kubernetes](https://k3s.io/)

---
# microk8s
## installation
* https://github.com/ubuntu/microk8s
* https://microk8s.io/

```
sudo snap install microk8s --classic
sudo snap install microk8s --classic --edge 
```
enable addons
```
microk8s.start
microk8s.enable dns dashboard
```
check installation
```
microk8s.inspect
```
check journals for services
```
journalctl -u snap.microk8s.daemon-docker
```
* snap.microk8s.daemon-apiserver
* snap.microk8s.daemon-controller-manager
* snap.microk8s.daemon-scheduler
* snap.microk8s.daemon-kubelet
* snap.microk8s.daemon-proxy
* snap.microk8s.daemon-docker
* snap.microk8s.daemon-etcd
---
# minikube
## installation
```
sudo snap install minikube
```

## installation
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl

export MINIKUBE_WANTUPDATENOTIFICATION=false
export MINIKUBE_WANTREPORTERRORPROMPT=false
export MINIKUBE_HOME=$HOME
export CHANGE_MINIKUBE_NONE_USER=true
mkdir $HOME/.kube || true
touch $HOME/.kube/config

export KUBECONFIG=$HOME/.kube/config
sudo -E ./minikube start --vm-driver=none

# wait that Minikube has created
for i in {1..150}; do # timeout for 5 minutes
   ./kubectl get po &> /dev/null
   if [ $? -ne 1 ]; then
      break
  fi
  sleep 2
done
```
### set up env
```
minikube completion bash
```

## start
```
minikube start
```

## uninstall kubectl, uninstall minikube
```
kubectl delete node --all
kubectl delete pods --all
kubectl stop
kubectl delete

launchctl stop '*kubelet*.mount'
launchctl stop localkube.service
launchctl disable localkube.service

sudo kubeadm reset
rm -rf ~/.kube ~/.minikube
sudo rm -rf /usr/local/bin/localkube /usr/local/bin/minikube
sudo rm -rf /etc/kubernetes/

# sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*
sudo apt-get purge kube*
sudo apt-get autoremove

docker system prune -af --volumes
```

## start without VirtualBox/KVM
```
export MINIKUBE_WANTUPDATENOTIFICATION=false
export MINIKUBE_WANTREPORTERRORPROMPT=false
export MINIKUBE_HOME=$HOME
export CHANGE_MINIKUBE_NONE_USER=true

export KUBECONFIG=$HOME/.kube/config
sudo -E minikube start --vm-driver=none
```

## kubectl using minikube context
permanently
```
kubectl config use-context minikube
```

temporary
```
kubectl get pods --context=minikube
```

## access cluster
* reverse proxy 
  activate proxy from current node
  ```sh
  kubectl proxy --port 9090
  # execute request against kubectl via reverse-proxy
  curl {current node ip}:9090/api
  ```
* token access
   ```sh
  $TOKEN=$(kubectl describe secret $(kubectl get secrets | grep ^default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d " ")
  echo $TOKEN | tee token.crt
  echo "Authorization: Bearer "$TOKEN | tee token.header
  # execute from remote node against ( cat ~/.kube/config | grep server )
  curl https://{ip:port}/api --header @token.crt --insecure
  ```
  

## check namespaces
```
kubectl get namespaces
```
at least three namespaces will be provided
```
default       Active    15m
kube-public   Active    15m
kube-system   Active    15m
```

## create namespace
```
kubectl create namespace my-own-namespace
```
or via yaml file 
```
kubectl apply -f {filename}
```
```
kind: Namespace
apiVersion: v1
metadata:
  name: test
```

### create limits for namespace
example for previous namespace declaration 
```json
apiVersion: v1
kind: LimitRange
metadata:
  name: my-own-namespace
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```
### limits for certain container
```json
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
> also can be limited: pods, pv/pvc, services, configmaps...

### print limits
```
kubectl get quota --namespace my-own-namespace
kubectl describe quota/compute-quota --namespace my-own-namespace
kubectl describe quota/object-quota --namespace my-own-namespace

kubectl describe {pod-name} limits 
kubectl describe {pod-name} limits --namespace my-own-namespace
```

## delete namespace
```
kubectl delete namespace {name of namespace}
```

## users
* normal user
  * client certificates
  * bearer tokens
  * authentication proxy
  * http basic authentication
  * OpenId
* service user
  * service account tokens
  * credentials using secrets
  * specific to namespace
  * created by objects
  * anonymous user ( not authenticated )

## configuration, configmap
### create configmap
example of configuration
```properties
color.ok=green
color.error=red
textmode=true
security.user.external.login_attempts=5
```
* create configuration on cluster
  ```
  kubectl create configmap my-config-file --from-env-file=/local/path/to/config.properties
  ```
  will be created next configuration
  ```yaml
  ...
  data:
    color.ok=green
    color.error=red
    textmode=true
    security.user.external.login_attempts=5
  ```
* or configuration with additional key, additional abstraction over the properties ( like Map of properties )
  ```
  kubectl create configmap my-config-file --from-file=name-or-key-of-config=/local/path/to/config.properties
  ```
  created file is:
  ```yaml
  data:
    name-or-key-of-config:
      color.ok=green
      color.error=red
      textmode=true
      security.user.external.login_attempts=5
  ```
* or configuration with additional key based on filename ( key will be a name of file )
  ```
  kubectl create configmap my-config-file --from-file=/local/path/to/
  ```
  created file is:
  ```yaml
  data:
    config.properties:
      color.ok=green
      color.error=red
      textmode=true
      security.user.external.login_attempts=5
  ```
* or inline creation
  ```
  kubectl create configmap special-config --from-literal=color.ok=green --from-literal=color.error=red
  ```

### get configurations, read configuration in specific format
```
kubectl get configmap 
kubectl get configmap --namespace kube-system 
kubectl get configmap --namespace kube-system kube-proxy --output json
```

### using configuration, using of configmap
* one variable from configmap
  ```yaml
  spec:
    containers:
      - name: test-container
        image: k8s.gcr.io/busybox
        command: [ "/bin/sh", "echo $(MY_ENVIRONMENT_VARIABLE)" ]
        env:
          - name: MY_ENVIRONMENT_VARIABLE
            valueFrom:
              configMapKeyRef:
                name: my-config-file
                key: security.user.external.login_attempts
  ```
* all variables from configmap
   ```yaml
   ...
        envFrom:
        - configMapRef:
            name: my-config-file
   ```

## start readiness, check cluster
```
kubectl cluster-info dump
kubectl get node
minikube dashboard
```

## addons
```
minikube addons list
minikube addons enable ingress
```

## deployment
![main schema](https://i.postimg.cc/6pfGpWvN/deployment-high-level.png)

## start dummy container
```
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
```

## create deployment ( with replica set )
```
kubectl run http --image=katacoda/docker-http-server:latest --replicas=1
```

## scale deployment 
```
kubectl scale --replicas=3 deployment {name of the deployment}
```

## create from yaml file
```
kubectl create -f /path/to/controller.yml
```
# update yaml file
```
kubectl apply -f /path/to/controller.yml
```

## create service fastly
```
kubectl expose deployment helloworld-deployment --type=NodePort --name=helloworld-service
kubectl expose deployment helloworld-deployment --external-ip="172.17.0.13" --port=8000 --target-port=80
```

## reach out service
```
minikube service helloworld-service
minikube service helloworld-service --url
```

## service port range
```
kube-apiserver --service-node-port-range=30000-40000
```

## describe resources
```
kubectl describe deployment {name of deployment}
kubectl describe service {name of service}
```

## describe users, user token
```
kubectl --namespace kube-system describe secret admin-user
```

## get resources
```
kubectl get all --all-namespaces
kubectl get pods
kubectl get pods --namespace kube-system
kubectl get pods --show-labels
kubectl get pods --output=wide --selector="run=load-balancer-example" 
kubectl get service --output=wide
kubectl get service --output=wide --selector="app=helloworld"
kubectl get deployments
kubectl get replicasets
kubectl get nodes
kubectl get cronjobs
kubectl get daemonsets
kubectl get pods,deployments,services,rs,cm,pv,pvc -n demo
```

## determinate cluster 'hostIP' to reach out application(s)
```
minikube ip
```
open 'kube-dns-....'/hostIP
open 'kube-proxy-....'/hostIP

## edit configuration of controller
```
kubectl edit pod hello-minikube-{some random hash}
kubectl edit deploy hello-minikube
kubectl edit ReplicationControllers helloworld-controller
kubectl set image deployment/helloworld-deployment {name of image}
```

## rollout status
```
kubectl rollout status  deployment/helloworld-deployment
```

## rollout history
```
kubectl rollout history  deployment/helloworld-deployment
kubectl rollout undo deployment/helloworld-deployment
kubectl rollout undo deployment/helloworld-deployment --to-revision={number of revision from 'history'}
```

## delete running container
```
kubectl delete pod hello-minikube-6c47c66d8-td9p2
```

## delete deployment
```
kubectl delete deploy hello-minikube
```
## delete ReplicationController
```
kubectl delete rc helloworld-controller
```
## delete PV/PVC
```
oc delete pvc/pvc-scenario-output-prod
```

## port forwarding from local to pod/deployment/service
next receipts allow to redirect 127.0.0.1:8080 to pod:6379
```
kubectl port-forward redis-master-765d459796-258hz      8080:6379 
kubectl port-forward pods/redis-master-765d459796-258hz 8080:6379
kubectl port-forward deployment/redis-master            8080:6379 
kubectl port-forward rs/redis-master                    8080:6379 
kubectl port-forward svc/redis-master                   8080:6379
```

## NodeSelector for certain host
```
spec:
  nodeSelector: 
    kubernetes.io/hostname: gtxmachine1-ev
```

## persistent volume
```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume3
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data3"
```

to access created volume
```
ls /mnt/data3
```

list of existing volumes
```
kubectl get pv 
kubectl get pvc
```

## Serverless
* OpenFaas
* Kubeless
* Fission
* OpenWhisk


## labels
### show labels for each node
```
kubectl get nodes --show-labels
```

### add label to Node
```
kubectl label nodes {node name} my_label=my_value
```

### remove label from Node
```
kubectl label nodes {node name} my_label-
```

### deploy Pod on Node with label
```
apiVersion: v1
kind: Pod
metadata:
...
spec:
...
  nodeSelector:
    my_label=my_value
```

### create Deployment for specific node
```
apiVersion: some-version
kind: Deployment
metadata:
...
spec:
...
  nodeSelector:
    my_label=my_value
```

### resolving destination node
![when label was not found](https://i.postimg.cc/mDjTpWw3/type-affinity-anti-affinity.png)

* nodeAffinity
* * preferred - deploy in any case, with preferrence my_label=my_value
```
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: my_label
            operator: In
            values:
            - my_value
```
* * required - deploy only when label matched my_label=my_value
```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: my_label
            operator: In
            values:
            - my_value
```
* nodeAntiAffinity
```
spec:
  affinity:
    nodeAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
```
* podAffinity
* * preferred
spec.affinity.podAffinity.preferredDuringSchedulingIgnoredDuringExecution
* * required
spec.affinity.podAffinity.requiredDuringSchedulingIgnoredDuringExecution
* podAntiAffinity
* * preferred
spec.affinity.podAntiAffinity.preferredDuringSchedulingIgnoredDuringExecution
* * required
spec.affinity.podAntiAffinity.requiredDuringSchedulingIgnoredDuringExecution

## delete node from cluster
```bash
kubectl get nodes
kubectl delete {node name}
```

## add node to cluster
```
ssh {master node}
kubeadm token create --print-join-command  --ttl 0
```
expected result from previous command
```
kubeadm join 10.14.26.210:6443 --token 7h0dmx.2v5oe1jwed --discovery-token-ca-cert-hash sha256:1d28ebf950316b8f3fdf680af5619ea2682707f2e966fc0
```
go to node, clean up and apply token
```
ssh {node address}
rm -rf /etc/kubernetes
# apply token from previous step with additional flag: --ignore-preflight-errors=all
kubeadm join 10.14.26.210:6443 --token 7h0dmx.2v5oe1jwed --discovery-token-ca-cert-hash sha256:1d28ebf950316b8f3fdf680af5619ea2682707f2e966fc0 --ignore-preflight-errors=all
```

expected result from previous command
```
...
This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.
 
Run 'kubectl get nodes' on the master to see this node join the cluster. 
```
next block is not mandatory in most cases
```
systemctl restart kubelet
```

## logs
```
kubectl logs <name of pod>
```

## create dashboard 
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
```

## access dashboard
```
kubectl -n kube-system describe secret admin-user
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default
kubectl proxy
```

## common
### execute command on specific pod
```
kubectl exec -it {name of a pod}  -- bash -c "echo hi > /path/to/output/test.txt" 
```

# Extending 
## [custom controller](https://github.com/kubernetes/sample-controller)

# Flannel
![deployment diagram](https://i.postimg.cc/d3ZwjhXd/flannel.png)
restart nodes
```sh
# remove died pods
kubectl delete pods kube-flannel-ds-amd64-zsfz  --grace-period=0 --force
kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl create  -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

read logs
```
kubectl logs --namespace kube-system kube-flannel-ds-amd64-j4frw -c kube-flannel 
```
read logs from all pods
```
for each_node in $(kubectl get pods --namespace kube-system | grep flannel | awk '{print $1}');do echo $each_node;kubectl logs --namespace kube-system $each_node -c kube-flannel;done
```

read settings
```
kubectl --namespace kube-system exec kube-flannel-ds-amd64-wc4zp ls /etc/kube-flannel/
kubectl --namespace kube-system exec kube-flannel-ds-amd64-wc4zp cat /etc/kube-flannel/cni-conf.json
kubectl --namespace kube-system exec kube-flannel-ds-amd64-wc4zp cat /etc/kube-flannel/net-conf.json

kubectl --namespace kube-system exec kube-flannel-ds-amd64-wc4zp ls /run/flannel/
kubectl --namespace kube-system exec kube-flannel-ds-amd64-wc4zp cat /run/flannel/subnet.env

kubectl --namespace kube-system exec kube-flannel-ds-amd64-wc4zp ls /etc/cni/net.d
kubectl --namespace kube-system exec kube-flannel-ds-amd64-wc4zp cat /etc/cni/net.d/10-flannel.conflist
```
read DNS logs
```
kubectl get svc --namespace=kube-system | grep kube-dns
kubectl logs --namespace=kube-system coredns-78fcd94-7tlpw | tail
```

# Flannel
![architecture](https://i.postimg.cc/1X49qNcb/flannel.png)

# Helm
[documentation](https://docs.helm.sh/)
## Architecture
![main components](https://i.postimg.cc/gkBhFQHG/helm-architecture.png)


## installation
```
sudo snap install helm --classic
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
```

## de-installation
```
helm reset
```

## initialization
```
helm init
# sync latest available packages
helm repo update
```

## useful variables
```
* $HELM_HOME: the location of Helm's configuration
* $TILLER_HOST: the host and port that Tiller is listening on
* $HELM_BIN: the path to the helm command on your system
* $HELM_PLUGIN_DIR: the full path to this plugin (not shown above, but we'll see it in a moment).
```

## analyze local package
```
helm inspect { folder }
helm lint { folder }
```

## search remote package
```
helm search 
helm describe {full name of the package}
```

## information about remote package
```
helm info {name of resource}
helm status {name of resource}
```

## create package locally
```
helm create 
```
![folder structure](https://i.postimg.cc/d1kXZrL7/helm-sceleton.png)

### create package with local templates
```
ls -la ~/.helm/starters/
```

## install package
```
helm install { full name of the package }
helm install --name {my name for new package} { full name of the package }
helm install --name {my name for new package} --namespace {namespace} -f values.yml --debug --dry-run { full name of the package }
```

## install aws plugin
```
helm plugin install https://github.com/hypnoglow/helm-s3.git
```

## list of installed packages
```
helm list
helm list --all
helm ls
```

## package upgrade
local package
```
helm upgrade  {deployment/svc/rs/rc name} . --set replicas=2,maria.db.password="new password"
```
package by name
```
helm upgrade {name of package} {folder with helm scripts} --set replicas=2
```

check upgrade
```
helm history
helm rollback {name of package} {revision of history}
```

## remove packageHelm
```
helm delete --purge {name of package}
```


## trouble shooting
### issue with 'helm list'
```
E1209 22:25:57.285192    5149 portforward.go:331] an error occurred forwarding 40679 -> 44134: error forwarding port 44134 to pod de4963c7380948763c96bdda35e44ad8299477b41b5c4958f0902eb821565b19, uid : unable to do port forwarding: socat not found.
Error: transport is closing
```
solution
```
sudo apt install socat
```

### incompatible version of client and server
```
Error: incompatible versions client[v2.12.3] server[v2.11.0]
```
solution
```
helm init --upgrade
kubectl get pods --namespace kube-system # waiting for start Tiller
helm version
```

### certificate is expired
```
bootstrap.go:195] Part of the existing bootstrap client certificate is expired: 2019-08-22 11:29:48 +0000 UTC
```
solution
```bash
sudo cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# archive configuration
sudo cp /etc/kubernetes/pki /etc/kubernetes/pki_backup
sudo mkdir /etc/kubernetes/conf_backup
sudo cp /etc/kubernetes/*.conf /etc/kubernetes/conf_backup

# remove certificates
sudo rm /etc/kubernetes/pki/./apiserver-kubelet-client.crt 
sudo rm /etc/kubernetes/pki/./etcd/healthcheck-client.crt 
sudo rm /etc/kubernetes/pki/./etcd/server.crt 
sudo rm /etc/kubernetes/pki/./etcd/peer.crt 
sudo rm /etc/kubernetes/pki/./etcd/ca.crt 
sudo rm /etc/kubernetes/pki/./front-proxy-client.crt 
sudo rm /etc/kubernetes/pki/./apiserver-etcd-client.crt 
sudo rm /etc/kubernetes/pki/./front-proxy-ca.crt 
sudo rm /etc/kubernetes/pki/./apiserver.crt 
sudo rm /etc/kubernetes/pki/./ca.crt 
sudo rm /etc/kubernetes/pki/apiserver.crt 
sudo rm /etc/kubernetes/pki/apiserver-etcd-client.crt 
sudo rm /etc/kubernetes/pki/apiserver-kubelet-client.crt 
sudo rm /etc/kubernetes/pki/ca.crt 
sudo rm /etc/kubernetes/pki/front-proxy-ca.crt 
sudo rm /etc/kubernetes/pki/front-proxy-client.crt 

# remove configurations
sudo rm /etc/kubernetes/apiserver-kubelet-client.*
sudo rm /etc/kubernetes/front-proxy-client.*
sudo rm /etc/kubernetes/etcd/*
sudo rm /etc/kubernetes/apiserver-etcd-client.*
sudo rm /etc/kubernetes/admin.conf
sudo rm /etc/kubernetes/controller-manager.conf
sudo rm /etc/kubernetes/kubelet.conf
sudo rm /etc/kubernetes/scheduler.conf

# re-init certificates
sudo kubeadm init phase certs all --apiserver-advertise-address {master ip address} --ignore-preflight-errors=all

# re-init configurations
sudo kubeadm init phase kubeconfig all --ignore-preflight-errors=all

# re-start
sudo systemctl stop kubectl.service
sudo systemctl restart docker.service
docker system prune -af --volumes
reboot

# /usr/bin/kubelet
sudo systemctl start kubectl.service

# init locate kubectl
sudo cp /etc/kubernetes/admin.conf ~/.kube/config

# check certificate
openssl x509 -in /etc/kubernetes/pki/apiserver.crt  -noout -text  | grep "Not After"
```

## template frameworks
* [go template](https://godoc.org/text/template)
* [sprig template](https://godoc.org/github.com/Masterminds/sprig)
