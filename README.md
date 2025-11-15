# kubernetes-session3
# test wordpress : 
# 🏗️ WordPress + MySQL on Kubernetes (Beginner Friendly)

## 🌱 Goal of this Lab

In this lab, we deploy **WordPress** (frontend) and **MySQL** (backend) into a Kubernetes cluster.

We will:

* Create **two namespaces**:

  * `frontend` → for WordPress
  * `backend` → for MySQL
* Add **resource quotas** for each namespace
* Run a **MySQL Pod + Service** in `backend`
* Run a **WordPress Pod** in `frontend` that connects to the MySQL Service
* Access the WordPress setup page from the browser

This is very similar to a real production pattern: separate app and DB, separate namespaces, controlled resources.

---

## ✅ Prerequisites

* A running Kubernetes cluster (local or cloud)
* `kubectl` configured to talk to that cluster
* The following YAML files in this repo:

  * `namespaces.yaml`
  * `quotas.yaml`
  * `db-pod.yaml`
  * `mysql-service.yaml`
  * `wordpress-pod.yaml`

> Note: YAML files define the **desired state** of the cluster.
> We apply them using `kubectl apply -f <file>`.

---

## 1️⃣ Create Namespaces

**What:**
We create two namespaces: one for the frontend (WordPress), one for the backend (MySQL).

**Why:**
Namespaces help separate different parts of the system:

* Easier to organize
* Easier to apply different policies, quotas, and access control

**Commands:**

```bash
kubectl apply -f namespaces.yaml
kubectl get ns
```

Check that you see at least:

* `frontend`
* `backend`

---

## 2️⃣ Apply Resource Quotas

**What:**
We apply CPU and memory quotas to both namespaces.

**Why:**
Quotas prevent one team or app from using too many resources and affecting others.

**Commands:**

```bash
kubectl apply -f quotas.yaml

# check quotas in each namespace
kubectl get resourcequota -n frontend
kubectl get resourcequota -n backend
```

You should see the quota objects listed for each namespace.

---

## 3️⃣ Deploy MySQL in the Backend Namespace

**What:**
We create a MySQL Pod in the `backend` namespace, and set up the database name, user and password using environment variables.

**Why:**
WordPress needs a database to store posts, users, and settings.
This Pod is our database server.

**Commands:**

```bash
kubectl apply -f db-pod.yaml

# check the Pod
kubectl get pods -n backend

# wait until MySQL is ready
kubectl wait --for=condition=Ready pod/db -n backend --timeout=180s
```

If it’s not Ready, check what’s wrong:

```bash
kubectl describe pod db -n backend
kubectl logs db -n backend --tail=50
```

---

## 4️⃣ Expose MySQL with a Service

**What:**
We create a Service called (for example) `mysql-service` in the `backend` namespace.

**Why:**
The Service gives MySQL a **stable DNS name** inside the cluster.
WordPress will connect to the database using this name instead of an IP address.

**Commands:**

```bash
kubectl apply -f mysql-service.yaml

kubectl get svc -n backend
kubectl get endpoints mysql-service -n backend
```

* The Service should show port **3306**
* The Endpoints should show at least one IP (the MySQL Pod)

---

## 5️⃣ Deploy WordPress in the Frontend Namespace

**What:**
We create a WordPress Pod in `frontend` that uses environment variables to connect to MySQL.

Usually we set variables like:

* `WORDPRESS_DB_HOST` → `mysql-service.backend.svc.cluster.local:3306`
* `WORDPRESS_DB_USER` → same as configured in MySQL
* `WORDPRESS_DB_PASSWORD`
* `WORDPRESS_DB_NAME` → e.g. `wordpress`

**Why:**
The app and DB live in different namespaces, but Kubernetes DNS lets them talk to each other.

**Commands:**

```bash
kubectl apply -f wordpress-pod.yaml

kubectl get pods -n frontend

kubectl wait --for=condition=Ready pod/wordpress -n frontend --timeout=180s
```

If it doesn’t become Ready, check:

```bash
kubectl describe pod wordpress -n frontend
kubectl logs wordpress -n frontend --tail=50
```

---

## 6️⃣ Test Connectivity from WordPress to MySQL

**What:**
We go *inside* the WordPress Pod and check if it can reach MySQL by DNS and by port.

**Why:**
If WordPress shows “Error establishing database connection”, this is the first thing to check:

* DNS name correct?
* Port open?
* Service pointing to the right Pod?

**Commands:**

```bash
kubectl exec -n frontend -it wordpress -- bash -lc "
apt-get update && apt-get install -y inetutils-ping netcat &&
ping -c1 mysql-service.backend.svc.cluster.local &&
nc -zv mysql-service.backend.svc.cluster.local 3306
"
```

If both `ping` and `nc` succeed, network connectivity is good.
If you still see WordPress errors, it’s usually credentials or DB name.

---

## 7️⃣ Access WordPress in the Browser

**What:**
We forward port 80 of the WordPress Pod to a port on our local machine (8080).

**Why:**
This lets us test the app without exposing it publicly.

**Commands:**

```bash
kubectl port-forward -n frontend pod/wordpress 8080:80
```

Now open:

👉 `http://localhost:8080`

You should see:

* WordPress **installation / setup page**
* A form asking for **Site Title**, **Username**, **Password**, **Email**

If you see that screen, everything is working:

* Namespaces are correct
* Quotas aren’t blocking it
* WordPress can talk to MySQL
* The container is serving HTTP

---

## 8️⃣ Cleanup (Optional)

If you want to remove everything:

```bash
kubectl delete -f wordpress-pod.yaml
kubectl delete -f mysql-service.yaml
kubectl delete -f db-pod.yaml
kubectl delete -f quotas.yaml
kubectl delete -f namespaces.yaml
```

This will delete:

* WordPress Pod
* MySQL Pod
* MySQL Service
* Quotas
* Namespaces (`frontend`, `backend`)

---

## 🧠 What You Learned

By completing this lab, you practiced how to:

* Use **namespaces** to separate frontend and backend
* Use **resource quotas** to control total CPU and memory per namespace
* Deploy **MySQL** as a backend database
* Deploy **WordPress** as a frontend application
* Use **Kubernetes DNS** to connect Pods across namespaces
* Verify and troubleshoot connectivity from within the Pod
* Access the application using `kubectl port-forward`

This pattern is very close to real workloads you’ll run as a DevOps / Cloud / Kubernetes engineer.


#############################################################
# test1 

# **README.md — Running a Pod Inside a Kubernetes Namespace**

## **Overview**

This project demonstrates a core Kubernetes skill:
**running an application Pod inside its own namespace.**

Namespaces are essential in real-world clusters because different teams, environments, or applications must stay separated even when sharing the same cluster.

In this test, we:

* Created a dedicated namespace called **application**
* Deployed an Apache web server Pod inside that namespace
* Verified that the Pod is running successfully and isolated from other parts of the cluster

This is a foundation for organizational separation, access control, and resource management in Kubernetes.

---

## **Why Namespaces Matter**

Namespaces allow you to logically divide a Kubernetes cluster.

They provide:

### **1. Isolation**

Each namespace is like its own section of the cluster.
Teams or applications cannot accidentally interfere with each other.

### **2. Organization**

Resources are grouped cleanly.
For example:

* `application` namespace for frontend workloads
* `backend` namespace for databases
* `monitoring` namespace for Grafana / Prometheus

### **3. Access Control**

Role-Based Access Control (RBAC) can be applied per namespace.
A team can be given full access to **their** namespace and limited access elsewhere.

### **4. Naming Flexibility**

You can reuse the same resource names in different namespaces, such as:

* `pod/web`
* `deployment/api`
* `service/nginx`

Namespaces make this possible without name conflicts.

---

## **What Was Done in This Test**

### **1. A Namespace Was Created**

A namespace called **application** was set up to isolate this workload.

This ensures the Pod will not mix with resources in the default namespace or other environments.

### **2. An Apache Pod Was Deployed**

An Apache web server (using the required image and version) was deployed **inside** that namespace.

It represents a simple web application running independently from the rest of the cluster.

### **3. Verification**

We confirmed:

* The namespace exists
* The Pod is running inside the `application` namespace
* The correct container image and port are being used
* The Pod is healthy and responding

This proves the namespace is functioning correctly and workloads inside it are isolated and manageable.

---

## **How to Test the Pod (General Explanation)**

After deploying the Pod, we verified its behavior by:

### **Listing resources in the namespace**

Checking that the Pod appears only under the `application` namespace.

### **Describing the Pod**

Reviewing metadata, labels, container image, and events to ensure it started correctly.

### **Optional Port Forwarding**

Accessing the Apache welcome page through a local forwarded port to confirm the container is serving HTTP traffic.

This demonstrates that the Pod is not only created but is actually running as expected.

---

## **Why This Matters in Real Jobs**

Running Pods in namespaces is something DevOps, SRE, and Cloud Engineers do every day.

This test shows the ability to:

* Work with namespaces
* Deploy workloads correctly
* Organize cluster resources
* Understand isolation and environment separation
# * For theis tast using commands : 
# 1) Create the namespace
kubectl apply -f namespace.yaml

# 2) Verify namespace
kubectl get ns
kubectl get ns application

# 3) Create the Pod in that namespace
kubectl apply -f apache-pod.yaml

# 4) Verify Pod is running in the application namespace
kubectl get pods -n application
kubectl describe pod apache -n application

# 5) (Optional) Test HTTP from your laptop
kubectl port-forward -n application pod/apache 8080:80
# then open http://localhost:8080 in browser

# 6) (Optional) Clean up
kubectl delete -f apache-pod.yaml
kubectl delete -f namespace.yaml



---

## **Conclusion**

This exercise validated the ability to deploy and manage Kubernetes resources inside a dedicated namespace.
It reflects real-world practices for organizing clusters and ensuring that applications remain isolated, secure, and easy to operate.


###############################################################
# test2 

# **README.md — Kubernetes Namespaces & Service Accounts (Cloud Engineering Test)**

## **Overview**

The Cloud Engineering team wants to deploy their application into a Kubernetes cluster, but they need:

* Their **own namespace**
* Their **own service account**
* An application Deployment that **runs using this service account** rather than the default one.

This project demonstrates how to:

1. Create a namespace called `cloud-engineering`
2. Create a service account called `cloud-engineers`
3. Deploy a simple application (`internal-app`) in the default namespace
4. Move the application into the new `cloud-engineering` namespace
5. Run the application using the `cloud-engineers` service account

This shows how real companies isolate workloads using namespaces and assign Pod identities using Service Accounts.

---

## **Why Namespace?**

Namespaces give teams **isolated environments** inside the same Kubernetes cluster.

For example:

* Cloud engineering team gets their own namespace → no conflict with DevOps, Security, or other teams
* Resources inside one namespace don't affect another namespace
* Access control (RBAC) can be applied per namespace

---

## **Why Service Accounts?**

A **Service Account** is the **identity** a Pod uses to communicate with the Kubernetes API server.

Why it matters:

* Pods must authenticate when they read Secrets, ConfigMaps, or talk to the K8s API.
* Teams can have **their own identity** with limited permissions (principle of least privilege).
* We do NOT use human credentials inside Pods — we use service accounts.

---

## **Project Structure**

```
namespace.yaml
serviceaccount.yaml
internal-app.yaml
internal-app-cloudeng.yaml
README.md
```

---

# **Step-by-Step Instructions**

## **1. Create the cloud-engineering namespace**

### File: `namespace.yaml`

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

## **2. Create the cloud-engineers Service Account**

### File: `serviceaccount.yaml`

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

Why?
This gives the Cloud Engineering team a **dedicated identity** for running applications.

---

## **3. Create the internal-app Deployment (initially in default namespace)**

### File: `internal-app.yaml`

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
kubectl rollout status deploy/internal-app -n default
```

Why this step?
This simulates the application running **before** the Cloud Engineering team receives their namespace.

---

## **4. Move the internal-app Deployment to the cloud-engineering namespace**

> In Kubernetes, you **cannot** change the namespace of an existing object.
> You must delete it from the old namespace and re-create it in the new one.

Delete old:

```bash
kubectl delete -f internal-app.yaml
```

### File: `internal-app-cloudeng.yaml`

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

Apply the new version:

```bash
kubectl apply -f internal-app-deploy-cloudeng.yaml 
kubectl get deploy,pods -n cloud-engineering
kubectl rollout status deploy/internal-app -n cloud-engineering
```

Why this step?
Now the app:

* Runs in the **Cloud Engineering namespace**
* Uses their **cloud-engineers service account**
* Is fully isolated from the default namespace

---

## **5. Verify Service Account is in effect**

```bash
kubectl describe pod -n cloud-engineering -l app=internal-app | grep -i "Service Account"
```

You should see:

```
Service Account:  cloud-engineers
```

This proves the Pod is running under the Cloud Engineering identity.

---

# **Cleanup**

```bash
kubectl delete -f internal-app-deploy-cloudeng.yaml 
kubectl delete -f serviceaccount.yaml
kubectl delete -f namespace.yaml
```

---

# **Conclusion**

In this test, we demonstrated how Cloud Engineering can get:

* Their **own namespace** (isolation)
* Their **own Service Account** (identity & security)
* An application Deployment running **inside their namespace**
* Pods running using **their service account**

This mirrors real-world cluster operations where multiple teams share the same Kubernetes cluster securely.