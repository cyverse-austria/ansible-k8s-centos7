# Setup  HA control plane/  multi master node with Haproxy

**RKE2 relies on Containerd**

## Steps:
1. Setup HAproxy node `k8s-reverse-proxy.cyverse.at`
The playbooks bellow will take care of this.

2. we could run the following playbooks to setup the nodes.
```bash
ansible -i inventory/ -m ping all --user root
ansible-playbook -i inventory/ firewalld-config.yml --user root
ansible-playbook -i inventory/ provision-nodes.yml --user root

## 01 error
# yum update repomd.xml signature could not be verified for kubernetes
## solution
# repo_gpgcheck=0 in /etc/yum.repos.d/kubernetes.repo

## 02 stuck on versionlock docker-ce-cli 
## solution
# do it manully and comment out 
# ansible/kubernetes-multi-master/roles/docker-installer/tasks/main.yaml
```

3. initialize first master node.

**make sure haproxy has Master node in the list**

```bash
# ssh root@k8s-c2.cyverse.at

# check first with --dry-run
kubeadm init --control-plane-endpoint="64.227.71.29:6443" --upload-certs \
--pod-network-cidr=10.0.10.0/24 --dry-run

# run actualy
kubeadm init --control-plane-endpoint="64.227.71.29:6443" --upload-certs \
--pod-network-cidr=10.0.10.0/24
# this command will join and int the master node to the cluster
# --control-plane-endpoint (IP of the loadbalancer haproxy)
# --upload-certs (to copy the certs)
```

**output**

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

(MASTER)

  kubeadm join 64.227.71.29:6443 --token 6ixj5w.dj7eai8zw1kx8y34 \
	--discovery-token-ca-cert-hash sha256:341fdd8f7b14bdbbb5a440fd3d97336290c2467b055ed32b7551e0ca78a3b19a \
	--control-plane --certificate-key 9ec2409309a253ec9de57899049b9a333bcfb9952e0720c5542c239453d5df29

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

(WORKER)

kubeadm join 64.227.71.29:6443 --token 6ixj5w.dj7eai8zw1kx8y34 \
	--discovery-token-ca-cert-hash sha256:341fdd8f7b14bdbbb5a440fd3d97336290c2467b055ed32b7551e0ca78a3b19a
```

4. run this command to add the config file
```bash
# ssh root@k8s-c2.cyverse.at
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

5. add worker node

```bash
# this command was from output of 3.
kubeadm join 64.227.71.29:6443 --token 6ixj5w.dj7eai8zw1kx8y34 \
	--discovery-token-ca-cert-hash sha256:341fdd8f7b14bdbbb5a440fd3d97336290c2467b055ed32b7551e0ca78a3b19a
```

6. install pod network
```bash
 kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')" >> pod_setup.txt
```

# Adding new worker node
* update and add vm in **k8s-worker** `ansible/kubernetes-multi-master/inventory/cyverse`
* run ansible commands:
```bash
ansible -i inventory/ -m ping all --user root
ansible-playbook -i inventory/ firewalld-config.yml --user root
ansible-playbook -i inventory/ provision-nodes.yml --user root
```

* Join worker node to the cluster
```bash
# ssh root@k8s-w2.cyverse.at
kubeadm join 64.227.71.29:6443 --token 6ixj5w.dj7eai8zw1kx8y34 \
	--discovery-token-ca-cert-hash sha256:341fdd8f7b14bdbbb5a440fd3d97336290c2467b055ed32b7551e0ca78a3b19a
```

# Adding new master node

* update and add vm in **k8s-worker** `ansible/kubernetes-multi-master/inventory/cyverse`
* run ansible commands:
```bash
ansible -i inventory/ -m ping all --user root
ansible-playbook -i inventory/ firewalld-config.yml --user root
ansible-playbook -i inventory/ provision-nodes.yml --user root
```

* Join master node to the cluster
**make sure haproxy has this Master node in the list**
```bash
# ssh root@k8s-c1.cyverse.at
  kubeadm join 64.227.71.29:6443 --token 6ixj5w.dj7eai8zw1kx8y34 \
	--discovery-token-ca-cert-hash sha256:341fdd8f7b14bdbbb5a440fd3d97336290c2467b055ed32b7551e0ca78a3b19a \
	--control-plane --certificate-key 9ec2409309a253ec9de57899049b9a333bcfb9952e0720c5542c239453d5df29

# if the key is expired run this command from k8s-c1.cyverse.at to get new key
# kubeadm init phase upload-certs --upload-certs
```

* run this command to add the config file
```bash
# ssh root@k8s-c2.cyverse.at
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


# COPY kubeconfig to your local machine
```bash
# this will allow you to access your cluster from your local machine.
scp root@k8s-c1.cyverse.at:/etc/kubernetes/admin.conf ~/.kube/config
```



# install weave CNI 
```bash
# https://www.weave.works/docs/net/latest/kubernetes/kube-addon/
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

# Tainting and Labeling VICE Worker Nodes
Once you have your nodes joined the cluster:

The CyVerse Discovery Environment uses taints and labels to ensure that some nodes are used exclusively for VICE
analyses. To mark a node as a VICE worker node, run this command on any node that has access to the Kubernetes API:

**Run this command to label node**
```bash
kubectl label nodes vice-w1.cyverse.at vice=true
```

**To prevent non-VICE pods from running on a node, run this command:**
```bash
kubectl taint nodes vice-w1.cyverse.at vice=only:NoSchedule
```

**check if labeld**
```bash
kubectl get nodes -l vice=true
```

# deploy vice haproxy `vice-haproxy.cyverse.at`
we need to install haproxy-2.2.20 - see `ansible/kubernetes-multi-master/roles/vice-haproxy-installer/templates/haproxy_working.cfg`
```bash
# navigate
# cd ansible/kubernetes-multi-master/
ansible-playbook -i inventory/ vice-haproxy-install.yaml --user root

```


# TODO
* change hardcoded loadbalancer IP from `ansible/kubernetes-multi-master/roles/k8s-controler-haproxy/tasks/main.yml`
* create joining the worker node ansible 
* create docs - on how to add new master node to the cluster.
* find a solution for `01 error`
* find a solution for `02`


# check this link below if you need to create new token and certs to join the cluster
https://www.youtube.com/watch?v=c_AWJttifTc&t=333s


# enable Metrics API for k8s
```bash
# BEFOR THAT
# Enabling signed kubelet serving certificates
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#kubelet-serving-certs

linux@linux:~/Downloads$ # kubectl top nodes
# error: Metrics API not available
linux@linux:~/Downloads$ # kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability.yaml
```

# kubernetes certificates
https://www.youtube.com/watch?v=gXz4cq3PKdg
* Clsuter CA (cluster certificate authority) - all the certificates that will b used in the cluster is signed by this CA.


# original inventory

```conf
[k8s:children]
k8s-control-plane
k8s-worker

[kube-apiserver-haproxy]
k8s-reverse-proxy.cyverse.at

[k8s-control-plane]
k8s-c1.cyverse.at
k8s-c2.cyverse.at

[k8s-storage:children]
k8s-worker

[k8s-worker]
k8s-w1.cyverse.at
k8s-w2.cyverse.at
k8s-w3.cyverse.at
k8s-w4.cyverse.at
k8s-w5.cyverse.at
vice-w1.cyverse.at

[vice-workers]
vice-w1.cyverse.at

[outward-facing-proxy]
vice-haproxy.cyverse.at

[haproxy]
vice-haproxy.cyverse.at
```