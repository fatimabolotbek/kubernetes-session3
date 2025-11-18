Kubernetes Probes – Test 

This repository contains my solution for configuring readiness and liveness probes in a Kubernetes Pod.The goal of this test was to create a Pod that exposes two custom endpoints and use those endpoints for health checking.

Test Requirements: The application team requires two endpoints inside the container:

/started– Readiness Endpoint

- Returns 200 OK → The app has started and is ready to receive traffic
- Returns 500 → The app is still initializing (not ready yet)

---

### /health– Liveness Endpoint

- Returns 200 OK → The app is healthy and working normally
- Returns 500 → The app is unhealthy, Kubernetes must restart it

What I Needed to Build

- Create a Pod named application-probe
- Use the nginx image
- Configure:

            -readinessProbe → checks /started

             -livenessProbe → checks /health

- Both probes must use port 5000

---

---

## 📁 Final Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: application-probe
  labels:
    app: application-probe
    env: test
spec:
  containers:
    - name: nginx
      image: nginx

      ports:
        - containerPort: 5000

      # Checks if the application is ready for traffic
      readinessProbe:
        httpGet:
          path: /started
          port: 5000
        initialDelaySeconds: 5
        periodSeconds: 5

      # Checks if the application is still healthy
      livenessProbe:
        httpGet:
          path: /health
          port: 5000
        initialDelaySeconds: 10
        periodSeconds: 5

```

---

## 🔍 Important Note About This Test

The default nginx image does NOT run an application on port 5000, and it does NOT have /started  or  /health endpoints.

Because of that:

### Kubernetes probes will fail

### kubectl exec —curl    to port 5000 will fail

### The Pod may go into CrashLoopBackOff depending on probe timing

This is expected behavior, because the test focuses on *your probe configuration*, not on running a real app.

---

## How to Check Probe Status

### 1. Check pod status

```bash
kubectl get pods

```

### 2. Describe the pod to see probe results

```bash
kubectl describe pod application-probe

```

Look under the Events section — it will show:

- readiness probe failed (because  /started doesn’t exist)
- liveness probe failed (because /health doesn’t exist)

This is normal for this test.

---

## Why curl to port 5000 fails

Example:

bash
kubectl exec -it application-probe -- curl -I http://localhost:5000/started


Output:

curl: (7) Failed to connect to localhost port 5000

This is correct because:

### ❌ nginx does NOT listen on port 5000

### ❌ nginx does NOT have /started or /health endpoints

If you had a real app listening on port 5000, you would see:

- `/started` → 200 OK when ready
- `/health` → 200 OK when healthy

---

## What This Test Demonstrates

- You know how to configure readiness and liveness probes
- You understand how probes work with HTTP endpoints
- You correctly used port 5000
- You created a clean and labeled Pod definition
- You understand why probes fail when the app does not expose the endpoints

This is exactly what the task required.
