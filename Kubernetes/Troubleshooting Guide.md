# Kubernetes Troubleshooting Guide

## Table of Contents

1. Pod Not Starting
2. Pod CrashLoopBackOff
3. ImagePullBackOff
4. ErrImagePull
5. Pending Pods
6. Container Creating State
7. OOMKilled Error
8. Node Not Ready
9. Service Not Accessible
10. DNS Resolution Issues
11. Ingress Not Working
12. ConfigMap Issues
13. Secret Issues
14. PVC Pending
15. Volume Mount Failure
16. Deployment Not Updating
17. Pods Stuck in Terminating
18. High CPU Usage
19. High Memory Usage
20. API Server Unreachable
21. Worker Node Not Joining Cluster
22. CNI Network Issues
23. Certificate Expired
24. etcd Issues
25. Helm Deployment Failure
26. kubectl Connection Refused
27. Useful Troubleshooting Commands
28. Scenario-Based Interview Questions

---

# 1. Pod Not Starting

## Check Pod Status

```bash
kubectl get pods
```

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

Check Events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 2. Pod CrashLoopBackOff

## Error

```text
CrashLoopBackOff
```

## Troubleshooting

View logs:

```bash
kubectl logs <pod-name>
```

Previous container logs:

```bash
kubectl logs <pod-name> --previous
```

Describe pod:

```bash
kubectl describe pod <pod-name>
```

---

# 3. ImagePullBackOff

## Cause

* Wrong image name
* Wrong tag
* Registry authentication issue

## Verify

```bash
kubectl describe pod <pod-name>
```

Fix image:

```bash
kubectl set image deployment/nginx nginx=nginx:latest
```

---

# 4. ErrImagePull

## Cause

Container image does not exist.

Verify:

```bash
docker pull nginx:latest
```

Update deployment image.

---

# 5. Pending Pods

## Check

```bash
kubectl describe pod <pod-name>
```

Common Causes:

* Insufficient CPU
* Insufficient Memory
* PVC unavailable
* Node selector mismatch

Check Nodes:

```bash
kubectl get nodes
```

---

# 6. Container Creating State

Check Events:

```bash
kubectl describe pod <pod-name>
```

Possible Reasons:

* Image pulling
* Volume attachment
* Network setup

---

# 7. OOMKilled Error

## Verify

```bash
kubectl describe pod <pod-name>
```

Increase memory limits:

```yaml
resources:
  limits:
    memory: "512Mi"
```

---

# 8. Node Not Ready

Check:

```bash
kubectl get nodes
```

Describe node:

```bash
kubectl describe node <node-name>
```

Verify kubelet:

```bash
systemctl status kubelet
```

---

# 9. Service Not Accessible

Verify Service:

```bash
kubectl get svc
```

Check Endpoints:

```bash
kubectl get endpoints
```

Test:

```bash
curl <service-ip>
```

---

# 10. DNS Resolution Issues

Launch test pod:

```bash
kubectl run test --image=busybox -it --rm -- sh
```

Test DNS:

```bash
nslookup kubernetes.default
```

Verify CoreDNS:

```bash
kubectl get pods -n kube-system
```

---

# 11. Ingress Not Working

Check:

```bash
kubectl get ingress
```

Verify ingress controller:

```bash
kubectl get pods -n ingress-nginx
```

Describe ingress:

```bash
kubectl describe ingress
```

---

# 12. ConfigMap Issues

Check:

```bash
kubectl get configmap
```

Describe:

```bash
kubectl describe configmap app-config
```

---

# 13. Secret Issues

List:

```bash
kubectl get secrets
```

Decode:

```bash
echo <base64-value> | base64 -d
```

---

# 14. PVC Pending

Check PVC:

```bash
kubectl get pvc
```

Describe:

```bash
kubectl describe pvc
```

Verify StorageClass:

```bash
kubectl get storageclass
```

---

# 15. Volume Mount Failure

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

Check:

* PVC
* PV
* StorageClass

---

# 16. Deployment Not Updating

Verify rollout:

```bash
kubectl rollout status deployment/nginx
```

Check history:

```bash
kubectl rollout history deployment/nginx
```

Restart:

```bash
kubectl rollout restart deployment/nginx
```

---

# 17. Pods Stuck In Terminating

Force delete:

```bash
kubectl delete pod pod-name --force --grace-period=0
```

---

# 18. High CPU Usage

Check:

```bash
kubectl top pods
```

Node usage:

```bash
kubectl top nodes
```

---

# 19. High Memory Usage

Check:

```bash
kubectl top pods
```

Increase limits if required.

---

# 20. API Server Unreachable

Verify:

```bash
kubectl cluster-info
```

Check control plane:

```bash
systemctl status kube-apiserver
```

---

# 21. Worker Node Not Joining Cluster

Verify kubelet:

```bash
systemctl status kubelet
```

Rejoin:

```bash
kubeadm join ...
```

---

# 22. CNI Network Issues

Check:

```bash
kubectl get pods -n kube-system
```

Examples:

* Calico
* Flannel
* Cilium

Restart CNI if required.

---

# 23. Certificate Expired

Check:

```bash
kubeadm certs check-expiration
```

Renew:

```bash
kubeadm certs renew all
```

---

# 24. etcd Issues

Check status:

```bash
ETCDCTL_API=3 etcdctl endpoint health
```

Verify logs.

---

# 25. Helm Deployment Failure

Check releases:

```bash
helm list
```

Verify values:

```bash
helm get values release-name
```

Rollback:

```bash
helm rollback release-name 1
```

---

# 26. kubectl Connection Refused

## Error

```text
The connection to the server localhost:8080 was refused
```

Verify kubeconfig:

```bash
kubectl config view
```

Update EKS config:

```bash
aws eks update-kubeconfig --region us-east-1 --name cluster-name
```

---

# 27. Useful Troubleshooting Commands

## Pods

```bash
kubectl get pods -A
```

## Logs

```bash
kubectl logs pod-name
```

## Describe

```bash
kubectl describe pod pod-name
```

## Events

```bash
kubectl get events
```

## Nodes

```bash
kubectl get nodes
```

## Resource Usage

```bash
kubectl top pods
kubectl top nodes
```

## Cluster Info

```bash
kubectl cluster-info
```

---

# 28. Scenario-Based Interview Questions

### Scenario 1

A Pod is in CrashLoopBackOff. How will you troubleshoot?

### Scenario 2

A Pod is in Pending state for 30 minutes.

### Scenario 3

ImagePullBackOff occurs after deployment.

### Scenario 4

Service is created but application is not accessible.

### Scenario 5

Ingress is configured but domain is not opening.

### Scenario 6

Worker node suddenly becomes NotReady.

### Scenario 7

PVC remains in Pending state.

### Scenario 8

Pod gets OOMKilled repeatedly.

### Scenario 9

kubectl shows connection refused.

### Scenario 10

CoreDNS pods are failing.

### Scenario 11

Application cannot resolve service names.

### Scenario 12

Deployment rollout is stuck.

### Scenario 13

Pods are consuming excessive CPU.

### Scenario 14

Helm deployment fails after upgrade.

### Scenario 15

Certificate expiration causes cluster outage.

### Scenario 16

Worker node fails to join cluster.

### Scenario 17

etcd becomes unhealthy.

### Scenario 18

Container starts locally but fails in Kubernetes.

### Scenario 19

LoadBalancer service does not get external IP.

### Scenario 20

Pods cannot communicate across nodes.

---

# Best Practices

* Always check Events first.
* Use kubectl describe before deleting resources.
* Configure Resource Requests and Limits.
* Monitor cluster health regularly.
* Use Liveness and Readiness Probes.
* Backup etcd frequently.
* Keep Kubernetes version updated.
* Use RBAC and least privilege access.
* Enable centralized logging and monitoring.
* Follow GitOps practices for deployments.
