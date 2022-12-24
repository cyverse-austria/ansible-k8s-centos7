# Adding new worker node

## er-01: 
### command
```bash
ansible-playbook -i inventory/ provision-nodes.yml --user root
```

### error
```bash
STUCK
versionlock docker-ce-cli]

# SOLUTION
comment out 
# ansible/kubernetes-multi-master/roles/docker-installer/tasks/main.yaml
```

## er-02: 
### command
```bash
ansible-playbook -i inventory/ provision-nodes.yml --user root
```

### error
```bash
yum update repomd.xml signature could not be verified for kubernetes

# SOLUTION
# repo_gpgcheck=0 in /etc/yum.repos.d/kubernetes.repo
```

## er-03

```bash
# error
# WEAVE - network issue

# solution
kubectl -n kube-system delete $(kubectl get pods -n kube-system -l name=weave-net -o name)
```
