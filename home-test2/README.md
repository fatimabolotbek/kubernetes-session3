Namespaces

Service Accounts

Moving Deployment

 Using serviceAccountName

 Real-world explanation

 Natural English

Clear structure

---

# **Kubernetes Namespace & Service Account Test**

## **Overview**

This test demonstrates two fundamental Kubernetes concepts:

1. **Namespaces** – used for isolating applications
2. **Service Accounts** – used for assigning Pod identities

The Cloud Engineering team wants to deploy their application into your Kubernetes cluster, but they want:

- Their **own namespace**
- Their **own service account**
- Their application to **run using that service account**

This README explains what was done, why it matters, how the YAML files work, and how everything fits together.

---

# **What The Test Required**

According to the assignment:

1️⃣ Create a namespace called **cloud-engineering**

2️⃣ Create a service account called **cloud-engineers** inside that namespace

3️⃣ Create a simple Deployment called **internal-app** in the **default** namespace

4️⃣ Update the Deployment so it runs in the **cloud-engineering** namespace and uses the **cloud-engineers** service account

Finally, document everything in a README and push it to GitHub.

---

# **Why Namespaces Matter**

Namespaces allow different teams or environments to operate safely inside the same cluster.

Benefits:

### ✔ Isolation

Each team gets its own space → no conflicts, no accidental deletion of each other’s resources.

### ✔ Organization

Resources are grouped logically:

- `cloud-engineering` for Cloud Engineering
- `default` for generic workloads
- `dev`, `prod`, etc.

### ✔ Access Control (RBAC)

You can give Cloud Engineering access **only** to their namespace.

### ✔ Name Reuse

`deployment/internal-app` can exist in multiple namespaces without conflict.

---

# **Why Service Accounts Matter**

A Service Account is the **identity** that Pods use when talking to the Kubernetes API server.

A Pod might need to:

- Read a ConfigMap
- Read a Secret
- Check its own state
- Communicate with other services

Each Pod **needs an identity**.

Cloud Engineering requires their own identity → **cloud-engineers** service account.

This is best practice because:

- We never put user credentials inside Pods
- We assign least privilege to each team
- Pods authenticate safely inside the cluster

---

# **Project File Structure**

```
namespace.yaml
serviceaccount.yaml
internal-app.yaml
internal-app-cloudeng.yaml
README.md

```

---

# **Step-by-Step Implementation**

## **1️⃣ Create the cloud-engineering namespace**

**namespace.yaml**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cloud-engineering

```

Apply:

```bash
kubectl apply -f namespace.yaml
kubectl get ns cloud-engineering

```

---

## **2️⃣ Create the cloud-engineers Service Account**

**serviceaccount.yaml**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cloud-engineers
  namespace: cloud-engineering

```

Apply:

```bash
kubectl apply -f serviceaccount.yaml
kubectl get sa -n cloud-engineering

```

---

## **3️⃣ Deploy internal-app in the default namespace**

**internal-app.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: internal-app
  namespace: default
  labels:
    app: internal-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: internal-app
  template:
    metadata:
      labels:
        app: internal-app
    spec:
      containers:
      - name: app
        image: nginx:stable
        ports:
        - containerPort: 80

```

Apply:

```bash
kubectl apply -f internal-app.yaml
kubectl get deploy,pods -n default

```

---

# **Important Kubernetes Rule**

You **cannot move** a Deployment to a new namespace.

You **must delete it** and **recreate it** in the new namespace.

---

## **4️⃣ Re-create internal-app in the cloud-engineering namespace**

This time using the **cloud-engineers** service account.

**internal-app-cloudeng.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: internal-app
  namespace: cloud-engineering
  labels:
    app: internal-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: internal-app
  template:
    metadata:
      labels:
        app: internal-app
    spec:
      serviceAccountName: cloud-engineers
      containers:
      - name: app
        image: nginx:stable
        ports:
        - containerPort: 80

```

Apply:

```bash
kubectl delete -f internal-app.yaml
kubectl apply -f internal-app-cloudeng.yaml
kubectl get deploy,pods -n cloud-engineering

```

---

# ✔️ **Verify the Pod is using the correct Service Account**

```bash
kubectl describe pod -n cloud-engineering -l app=internal-app | grep "Service Account"

```

Expected:

```
Service Account:  cloud-engineers

```

---

# **Cleanup (optional)**

```bash
kubectl delete -f internal-app-cloudeng.yaml
kubectl delete -f serviceaccount.yaml
kubectl delete -f namespace.yaml

```

---

# **Main Concepts Demonstrated**

### ✔ Kubernetes Namespace

Provides isolation + organization + RBAC boundaries.

### ✔ Kubernetes Service Account

Assigns Pod identity → for secure API communication.

### ✔ Deployment recreation

Namespaces cannot be changed → must delete and re-create.

### ✔ Running Pod with custom identity

Using:

```yaml
serviceAccountName: cloud-engineers

```

### ✔ Real-world relevance

Every company that uses Kubernetes requires:

- Namespaces per team
- Service accounts per workload
- RBAC permissions based on identity
- Deployment movement across environments

This test demonstrates all of those skills.

---

# **Conclusion**

This test shows the ability to manage Kubernetes identities and isolation by:

- Creating a dedicated namespace
- Creating a dedicated service account
- Deploying an application
- Moving it securely into the correct namespace
- Running it under the correct identity

This is exactly how real Cloud Engineering teams structure their Kubernetes workloads.