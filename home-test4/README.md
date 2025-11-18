# Context

As part of application debugging, the company requires:

- The ability to view pod logs
- The ability to persist logs to a file, not just view in the terminal

Kubernetes stores container logs inside the node, and `kubectl logs` lets us read them.

In this test, we deploy a Pod that prints 5000 digits of Pi, then exits.

We retrieve the logs and save them as a text file.

---

## What I Needed to Do

### 1. Deploy a Pod named pi

Using this YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pi
spec:
  containers:
  - name: pi
    image: perl:5.34.0
    command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(5000)"]
  restartPolicy: Never

```

### 2. Retrieve the logs from the Pod

```bash
kubectl logs pi

```

### 3. Save the logs into a file called pi-logs.txt

```bash
kubectl logs pi > $(pwd)/pi-logs.txt

```

This creates a real file in your current directory.

---

## Why This Test Matters (Main Concept)

This test focuses on how Kubernetes manages logs, and why logs are important:

### 1. Debugging

Logs show:

- what the app printed
- errors
- output from the container
- how the app behaved before it crashed or finished

### 2. Kubernetes does NOT automatically save logs forever

Logs disappear when:

- the pod is deleted
- the node dies
- container rotates logs

Companies usually persist logs to:

- S3
- Elasticsearch
- Loki
- CloudWatch
- Or simply text files (as in this test)

### 3. This test checks that you know how to extract logs

You must know:

- `kubectl logs`
- pod lifecycle
- how to save logs locally

---

## Understanding the Pod (Pi Calculation)

The container runs this command:

```perl
print bpi(5000)

```

This prints the first 5000 digits of Pi, then the container exits.

Because the pod finishes fast, we set:

```yaml
restartPolicy: Never

```

This means Kubernetes does NOT restart the pod again and again.

---

## How to Validate the Work

### Check the Pod:

```bash
kubectl get pod pi

```

It should be in `Completed` state.

### View logs:

```bash
kubectl logs pi

```

You should see a long list of Pi digits.

### Check the file:

```bash
cat pi-logs.txt

```

It should contain the same output.

---

## Is It Safe to Commit pi-logs.txt to Git?

Yes — it contains only digits of Pi, not secrets.

No sensitive data.

It’s okay to upload.

But in real projects, logs should NOT be pushed to Git because logs may contain:

- tokens
- usernames
- sensitive error messages
- IPs
- stack traces
- customer data

For this test, it is safe.

---

## Summary

In this test, I demonstrated:

- Deploying a simple Kubernetes Pod
- Running a one-time job-like container
- Retrieving the Pod logs
- Persisting the logs into a file
- Understanding how Kubernetes handles logs and why they matter

This matches real Kubernetes debugging situations where logs must be saved and reviewed.