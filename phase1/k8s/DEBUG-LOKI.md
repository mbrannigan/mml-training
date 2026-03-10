# Debug: Loki Logs Not Working

**For Windows users with PowerShell and Docker Desktop**

Run these commands in order to diagnose the issue.

---

## 1. Check if Promtail pods are running

```powershell
kubectl get pods -n lgtm -l app=promtail
```

**Expected**: 3 pods (one per node), all showing `Running` and `1/1` ready.

---

## 2. Check Promtail logs for errors

```powershell
kubectl logs -n lgtm -l app=promtail --tail=50
```

**Look for**:
- ✅ Good: `msg="tail routine: started" path=/var/log/pods/...`
- ❌ Bad: Connection errors to Loki, permission errors, "no such file or directory"

**Filter logs in PowerShell**:

```powershell
kubectl logs -n lgtm -l app=promtail --tail=50 | Select-String -Pattern "tail|error|watching"
```

---

## 3. Check if Loki is receiving logs

```powershell
kubectl logs -n lgtm -l app=loki --tail=50
```

**Look for**:
- ✅ Good: HTTP POST requests from Promtail, "push request" entries
- ❌ Bad: Connection errors or no POST requests

---

## 4. Test Loki is reachable from inside the cluster

```powershell
kubectl run -n lgtm test-loki --image=curlimages/curl --rm -it --restart=Never -- curl -s http://loki:3100/ready
```

**Should return**: `ready`

---

## 5. Verify log files exist and are readable

```powershell
# Check what directories Promtail can see
kubectl exec -n lgtm -it $(kubectl get pods -n lgtm -l app=promtail -o jsonpath='{.items[0].metadata.name}') -- ls -la /var/log/pods/

# Check actual log files inside a pod directory
kubectl exec -n lgtm -it $(kubectl get pods -n lgtm -l app=promtail -o jsonpath='{.items[0].metadata.name}') -- sh -c "ls -la /var/log/pods/lgtm_grafana-*/grafana/"

# Verify Promtail can read the logs
kubectl exec -n lgtm -it $(kubectl get pods -n lgtm -l app=promtail -o jsonpath='{.items[0].metadata.name}') -- sh -c "head -5 /var/log/pods/lgtm_grafana-*/grafana/0.log"
```

**Expected log format** (CRI format with timestamp):
```
2026-03-08T15:19:56.778570159Z stdout F logger=settings t=2026-03-08T15:19:56.778292448Z level=info msg="Starting Grafana"
```

---

## Common Issue: Docker Desktop Log Path Problem

**Symptom**: Promtail starts but shows no "tail routine: started" messages in logs.

**Root Cause**: Docker Desktop on Windows stores logs at:
```
/var/log/pods/<namespace>_<pod>_<uid>/<container>/0.log
```

Kubernetes service discovery with dynamic path generation often fails to match this pattern correctly.

### ✅ Solution: Use Static Path Pattern (Already Applied)

The current working configuration in `configmaps/promtail-config.yaml` uses:

```yaml
scrape_configs:
  - job_name: kubernetes-pods
    static_configs:
      - targets:
          - localhost
        labels:
          job: kubernetes-pods
          __path__: /var/log/pods/*/*/*.log
    pipeline_stages:
      - cri: {}  # Parse CRI log format
      - regex:
          expression: '^/var/log/pods/(?P<namespace>[^_]+)_(?P<pod>[^_]+)_[^/]+/(?P<container>[^/]+)/'
          source: filename
      - labels:
          namespace:
          pod:
          container:
```

**Key points**:
- `__path__: /var/log/pods/*/*/*.log` - Static wildcard pattern that matches all log files
- `pipeline_stages: cri: {}` - Parses CRI format logs (Docker Desktop default)
- `regex` - Extracts namespace, pod, and container labels from the file path

This configuration is **already applied** in the repository and should work out of the box.

---

## Verification: Check Promtail is Tailing Files

After applying the fix, check Promtail logs:

```powershell
kubectl logs -n lgtm -l app=promtail --tail=50 | Select-String -Pattern "tail routine"
```

**You should see**:
```
level=info msg="tail routine: started" path=/var/log/pods/lgtm_grafana-86cc5bf9c5-lcjdr_.../grafana/0.log
level=info msg="tail routine: started" path=/var/log/pods/lgtm_loki-dcfff89b5-skffc_.../loki/0.log
...
```

If you see these messages, **Promtail is working correctly!**

---

## Verify Logs in Grafana

1. Open Grafana at [http://localhost:3000](http://localhost:3000) (with port-forward running)
2. Go to **Explore** (compass icon in left sidebar)
3. Select **Loki** data source from dropdown
4. Run this query:

```logql
{namespace="lgtm"}
```

**Expected**: You should see logs from all LGTM stack pods.

**Try these queries**:

```logql
# Grafana logs only
{namespace="lgtm", pod=~"grafana.*"}

# Loki logs only  
{namespace="lgtm", pod=~"loki.*"}

# Filter for errors
{namespace="lgtm"} |= "error"

# Filter for specific container
{namespace="lgtm", container="prometheus"}
```

---

## If Logs Still Don't Work

### Check ConfigMap was applied

```powershell
kubectl get configmap promtail-config -n lgtm -o yaml
```

Look for `__path__: /var/log/pods/*/*/*.log` in the output.

### Force restart Promtail

```powershell
# Delete all Promtail pods to force recreation
kubectl delete pods -n lgtm -l app=promtail

# Wait for new pods to start
kubectl get pods -n lgtm -l app=promtail -w
```

Press `Ctrl+C` when all pods show `Running` and `1/1`.

### Check Promtail DaemonSet volume mounts

```powershell
kubectl describe daemonset promtail -n lgtm
```

**Required volume mounts**:
```yaml
Volume Mounts:
  /var/log from varlog (rw)
  /var/lib/docker/containers from varlibdockercontainers (ro)
Volumes:
  varlog:
    Type:          HostPath (bare host directory volume)
    Path:          /var/log
  varlibdockercontainers:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/docker/containers
```

---

## Still Having Issues?

Gather diagnostic information:

```powershell
# Get all pod status
kubectl get pods -n lgtm

# Get Promtail logs
kubectl logs -n lgtm -l app=promtail --tail=100 > promtail-logs.txt

# Get Loki logs
kubectl logs -n lgtm -l app=loki --tail=50 > loki-logs.txt

# Describe Promtail pod
kubectl describe pod -n lgtm $(kubectl get pods -n lgtm -l app=promtail -o jsonpath='{.items[0].metadata.name}') > promtail-describe.txt

# Get ConfigMap
kubectl get configmap promtail-config -n lgtm -o yaml > promtail-config.txt
```

Review these files or share them for further troubleshooting.

---

## Reference

- **Working config**: [phase1/k8s/configmaps/promtail-config.yaml](https://github.com/mbrannigan/mml-training/blob/feature/k8s-manifests/phase1/k8s/configmaps/promtail-config.yaml)
- **Promtail docs**: https://grafana.com/docs/loki/latest/send-data/promtail/
- **CRI log format**: https://github.com/kubernetes/design-proposals-archive/blob/main/node/kubelet-cri-logging.md
