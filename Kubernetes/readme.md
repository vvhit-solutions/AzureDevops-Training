
# 📘 Kubernetes `kubectl` Quick Reference Guide

This guide provides a comprehensive overview of commonly used `kubectl` commands, organized by category, along with practical examples to assist in managing Kubernetes clusters efficiently.

# 📋 Frequently Used `kubectl` Commands

| Command                                                        | Description                                         |
|:---------------------------------------------------------------|:----------------------------------------------------|
| kubectl get pods                                               | List all pods in the current namespace              |
| kubectl get services                                           | List all services in the current namespace          |
| kubectl get nodes                                              | List all nodes in the cluster                       |
| kubectl describe pod <pod-name>                                | Show details of a specific pod                      |
| kubectl logs <pod-name>                                        | Show logs from a specific pod                       |
| kubectl exec -it <pod-name> -- /bin/bash                       | Execute a shell in a running pod                    |
| kubectl apply -f <file.yaml>                                   | Apply a configuration from a YAML file              |
| kubectl delete -f <file.yaml>                                  | Delete resources defined in a YAML file             |
| kubectl create -f <file.yaml>                                  | Create resources from a YAML file                   |
| kubectl config get-contexts                                    | List available contexts                             |
| kubectl config use-context <context>                           | Switch to a specific context                        |
| kubectl scale deployment <name> --replicas=<num>               | Scale a deployment to a specific number of replicas |
| kubectl expose deployment <name> --type=LoadBalancer --port=80 | Expose a deployment as a service                    |
| kubectl rollout status deployment <name>                       | Check rollout status of a deployment                |
| kubectl get all                                                | List all resources in the current namespace         |
| kubectl port-forward <pod-name> 8080:80                        | Forward local port to a port on a pod               |
| kubectl top pod                                                | Show resource usage of pods                         |
| kubectl edit deployment <name>                                 | Edit a deployment in place                          |
| kubectl delete pod <pod-name>                                  | Delete a specific pod                               |
| kubectl get events                                             | List all recent events in the namespace             |

---

## 🔧 Setup and Configuration

### Enable Autocompletion

**Bash:**
```bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

**Zsh:**
```bash
source <(kubectl completion zsh)
echo "source <(kubectl completion zsh)" >> ~/.zshrc
```

### Create an Alias for `kubectl`
```bash
alias k=kubectl
complete -o default -F __start_kubectl k
```

### View Cluster Information
```bash
kubectl cluster-info
kubectl config view
kubectl config get-contexts
kubectl config use-context <context-name>
```

---

## 📦 Working with Pods

### List All Pods
```bash
kubectl get pods
kubectl get pods -n <namespace>
```

### Describe a Pod
```bash
kubectl describe pod <pod-name>
```

### Delete a Pod
```bash
kubectl delete pod <pod-name>
```

### Execute a Command in a Pod
```bash
kubectl exec -it <pod-name> -- /bin/bash
```

### View Pod Logs
```bash
kubectl logs <pod-name>
kubectl logs -f <pod-name>
```

---

## 🚀 Deployments and ReplicaSets

### Create a Deployment
```bash
kubectl create deployment <deployment-name> --image=<image-name>
```

### Scale a Deployment
```bash
kubectl scale deployment <deployment-name> --replicas=<number>
```

### Update a Deployment Image
```bash
kubectl set image deployment/<deployment-name> <container-name>=<new-image>
```

### Rollback a Deployment
```bash
kubectl rollout undo deployment/<deployment-name>
```

---

## 🌐 Services and Networking

### Expose a Deployment as a Service
```bash
kubectl expose deployment <deployment-name> --type=LoadBalancer --port=80 --target-port=8080
```

### List Services
```bash
kubectl get services
```

### Describe a Service
```bash
kubectl describe service <service-name>
```

---

## 🛠️ ConfigMaps and Secrets

### Create a ConfigMap from a File
```bash
kubectl create configmap <configmap-name> --from-file=<file-path>
```

### Create a Secret from Literal Values
```bash
kubectl create secret generic <secret-name> --from-literal=<key>=<value>
```

### View ConfigMaps and Secrets
```bash
kubectl get configmaps
kubectl get secrets
```

---

## 📂 Namespaces

### List All Namespaces
```bash
kubectl get namespaces
```

### Create a Namespace
```bash
kubectl create namespace <namespace-name>
```

### Delete a Namespace
```bash
kubectl delete namespace <namespace-name>
```

---

## 🧪 Debugging and Troubleshooting

### Describe a Resource
```bash
kubectl describe <resource-type> <resource-name>
```

### View Events
```bash
kubectl get events
```

### Port Forwarding
```bash
kubectl port-forward <pod-name> <local-port>:<pod-port>
```

---

## 📄 Additional Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Command Reference](https://kubernetes.io/docs/reference/kubectl/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

*This guide is based on Kubernetes version 1.32. For the most up-to-date information, please refer to the official Kubernetes documentation.*
