# Troubleshooting Kubernetes Deployment

## Quick Diagnostics

Run these commands to check what's happening:

### 1. Check if you're on the right branch and have the files
```bash
git branch
ls phase1/k8s/
```

### 2. Check if namespace exists
```bash
kubectl get namespace lgtm
```

### 3. Check if pods are running
```bash
kubectl get pods -n lgtm
```

### 4. Check services and their ports
```bash
kubectl get svc -n lgtm
```

### 5. Check if Kubernetes is actually running
```bash
kubectl cluster-info
kubectl get nodes
```

## Common Issues

### Issue: "URLs don't work"

**Cause 1: Manifests not deployed yet**
- You need to actually apply the manifests
```bash
cd C:\mml-lab\mml-training
git checkout feature/k8s-manifests
kubectl apply -f phase1/k8s/namespace.yaml
kubectl apply -f phase1/k8s/configmaps/
kubectl apply -f phase1/k8s/
```

**Cause 2: Pods not ready yet**
- Check pod status:
```bash
kubectl get pods -n lgtm -w
```
- Wait until all pods show `Running` and `1/1` ready

**Cause 3: Docker Desktop Kubernetes not enabled**
- Open Docker Desktop → Settings → Kubernetes
- Ensure "Enable Kubernetes" is checked
- Wait for "Kubernetes is running" status

**Cause 4: Port forwarding needed (alternative to NodePort)**

If NodePort doesn't work, use port forwarding instead:
```bash
# Grafana on localhost:3000
kubectl port-forward -n lgtm svc/grafana 3000:3000

# Prometheus on localhost:9090 (in another terminal)
kubectl port-forward -n lgtm svc/prometheus 9090:9090
```

### Issue: Pods in CrashLoopBackOff or Error state

Check logs for the failing pod:
```bash
kubectl logs -n lgtm <pod-name>
kubectl describe pod -n lgtm <pod-name>
```

Common fixes:
- **Config errors**: Check ConfigMap is applied correctly
```bash
kubectl get configmap -n lgtm
```
- **Resource limits**: Pods may need more memory/CPU
- **Image pull errors**: Check internet connection

### Issue: "Connection refused" when accessing URLs

**For Docker Desktop on Windows:**

NodePort services should work on `localhost`, but if they don't:

1. **Check the actual NodePort assigned:**
```bash
kubectl get svc -n lgtm
```
Look for the PORT(S) column, e.g., `3000:30300/TCP`

2. **Try accessing via the node IP:**
```bash
kubectl get nodes -o wide
```
Then use: `http://<NODE-IP>:30300`

3. **Use port-forward as alternative** (recommended for local dev):
```bash
kubectl port-forward -n lgtm svc/grafana 3000:3000
```
Then access at: http://localhost:3000

### Issue: Still running Docker Compose version

If you have both running:
```bash
# Stop Docker Compose version
cd C:\mml-lab\mml-training\phase1
docker-compose down

# Then deploy Kubernetes version
kubectl apply -f k8s/
```

## Verification Steps

### 1. All pods should be running:
```bash
kubectl get pods -n lgtm
```
Expected output:
```
NAME                          READY   STATUS    RESTARTS   AGE
grafana-xxxxxxxxxx-xxxxx      1/1     Running   0          2m
prometheus-xxxxxxxxxx-xxxxx   1/1     Running   0          2m
mimir-xxxxxxxxxx-xxxxx        1/1     Running   0          2m
loki-xxxxxxxxxx-xxxxx         1/1     Running   0          2m
promtail-xxxxx                1/1     Running   0          2m
promtail-xxxxx                1/1     Running   0          2m (one per node)
```

### 2. Services should show NodePort assignments:
```bash
kubectl get svc -n lgtm
```
Expected output:
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
grafana      NodePort    10.96.x.x       <none>        3000:30300/TCP   2m
prometheus   NodePort    10.96.x.x       <none>        9090:30090/TCP   2m
loki         ClusterIP   10.96.x.x       <none>        3100/TCP         2m
mimir        ClusterIP   10.96.x.x       <none>        9009/TCP         2m
```

### 3. Test connectivity:
```bash
# From inside the cluster
kubectl run -n lgtm test-pod --image=curlimages/curl --rm -it --restart=Never -- curl http://grafana:3000

# Should return HTML
```

## Alternative: Use Port Forwarding Instead of NodePort

If NodePort continues to have issues on Docker Desktop, modify the services to use ClusterIP and access via port-forward:

```bash
# Edit services to ClusterIP
kubectl edit svc grafana -n lgtm
# Change type: NodePort to type: ClusterIP

# Then use port forwarding
kubectl port-forward -n lgtm svc/grafana 3000:3000
kubectl port-forward -n lgtm svc/prometheus 9090:9090
```

Access at:
- Grafana: http://localhost:3000
- Prometheus: http://localhost:9090

## What to Share for Help

If you need help, run these and share the output:

```bash
kubectl version --short
kubectl get nodes
kubectl get pods -n lgtm
kubectl get svc -n lgtm
kubectl logs -n lgtm <failing-pod-name>
```
