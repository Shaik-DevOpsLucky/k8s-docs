If **HPA (Horizontal Pod Autoscaler)** is not working in Kubernetes, check these things in order — start from the most common issues:

---

## ✅ 1. Check HPA Status

```bash
kubectl get hpa
kubectl describe hpa <hpa-name>
```

Look for:

* Current CPU / memory metrics
* Target values
* Events section (very important)
* Any errors like:

  * `failed to get cpu utilization`
  * `unable to get metrics`
  * `missing request for cpu`

---

## ✅ 2. Verify Metrics Server (Most Common Issue)

HPA needs **metrics-server**.

Check if it's running:

```bash
kubectl get pods -n kube-system | grep metrics
```

Or:

```bash
kubectl top nodes
kubectl top pods
```

If this fails → metrics-server is not working.

Fix:

* Install metrics-server
* Or check logs:

```bash
kubectl logs -n kube-system deployment/metrics-server
```

---

## ✅ 3. Check Resource Requests (VERY IMPORTANT)

HPA **requires CPU requests** to calculate scaling.

Check your deployment:

```bash
kubectl get deployment <app-name> -o yaml
```

Look for:

```yaml
resources:
  requests:
    cpu: "100m"
```

If `requests.cpu` is missing → HPA will NOT scale.

---

## ✅ 4. Verify Target Configuration

Check your HPA spec:

```yaml
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 50
```

Common mistakes:

* Wrong metric type
* Wrong target type
* Using memory without proper config

---

## ✅ 5. Check Deployment Reference

Ensure HPA is pointing to the correct resource:

```yaml
scaleTargetRef:
  apiVersion: apps/v1
  kind: Deployment
  name: my-app
```

Make sure:

* Name is correct
* Deployment exists
* Correct namespace

---

## ✅ 6. Check Events

Events give the real reason:

```bash
kubectl describe hpa <hpa-name>
```

Look at bottom:

```
Events:
  Warning  FailedGetResourceMetric
```

---

## ✅ 7. Check API Version Compatibility

Check Kubernetes version:

```bash
kubectl version
```

Make sure you are using supported HPA API:

* `autoscaling/v2` (recommended)
* Avoid deprecated versions

---

## ✅ 8. Test Load

Sometimes HPA is working — but no load.

Generate load:

```bash
kubectl run -it --rm busybox --image=busybox -- sh
```

Inside:

```sh
while true; do wget -q -O- http://my-service; done
```

Then watch:

```bash
kubectl get hpa -w
```

---

# 🔎 Quick Troubleshooting Flow

1. ❓ Does `kubectl top pods` work?
   → No → Fix metrics-server

2. ❓ Does deployment have CPU requests?
   → No → Add requests

3. ❓ Any errors in `kubectl describe hpa`?
   → Fix based on event message

4. ❓ Is there enough load?
   → Generate traffic

---
