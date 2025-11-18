**WordPress + MySQL on Kubernetes (Beginner Friendly)**

 **Goal of this Lab:** In this lab, we deploy **WordPress** (frontend) and **MySQL** (backend) into a Kubernetes cluster.

We will:

- Create **two namespaces**:
    - `frontend` → for WordPress
    - `backend` → for MySQL
- Add **resource quotas** for each namespace
- Run a **MySQL Pod + Service** in `backend`
- Run a **WordPress Pod** in `frontend` that connects to the MySQL Service
- Access the WordPress setup page from the browser

This is very similar to a real production pattern: separate app and DB, separate namespaces, controlled resources.

---

**Prerequisites**

- A running Kubernetes cluster (local or cloud)
- `kubectl` configured to talk to that cluster
- The following YAML files in this repo:
    - `namespaces.yaml`
    - `quotas.yaml`
    - `db-pod.yaml`
    - `mysql-service.yaml`
    - `wordpress-pod.yaml`

Note: YAML files define the desired state of the cluster. We apply them using kubectl apply -f <file>.

---

 **Create Namespaces**

**What:** We create two namespaces: one for the frontend (WordPress), one for the backend (MySQL).

**Why:** Namespaces help separate different parts of the system:

- Easier to organize
- Easier to apply different policies, quotas, and access control

**Commands:**

```
kubectl apply -f namespaces.yaml
kubectl get ns
```

Check that you see at least:

- `frontend`
- `backend`

---

**Apply Resource Quotas**

**What:** We apply CPU and memory quotas to both namespaces.

**Why:** Quotas prevent one team or app from using too many resources and affecting others.

**Commands:**

```
kubectl apply -f quotas.yaml

# check quotas in each namespace
kubectl get resourcequota -n frontend
kubectl get resourcequota -n backend
```

You should see the quota objects listed for each namespace.

---

**Deploy MySQL in the Backend Namespace**

**What:** We create a MySQL Pod in the `backend` namespace, and set up the database name, user and password using environment variables.

**Why:** WordPress needs a database to store posts, users, and settings. This Pod is our database server.

**Commands:**

```
kubectl apply -f db-pod.yaml

# check the Pod
kubectl get pods -n backend

# wait until MySQL is ready
kubectl wait --for=condition=Ready pod/db -n backend --timeout=180s
```

If it’s not Ready, check what’s wrong:

```
kubectl describe pod db -n backend
kubectl logs db -n backend --tail=50
```

---

 **Expose MySQL with a Service**

**What:** We create a Service called (for example) `mysql-service` in the `backend` namespace.

**Why:** The Service gives MySQL a **stable DNS name** inside the cluster. WordPress will connect to the database using this name instead of an IP address.

**Commands:**

```
kubectl apply -f mysql-service.yaml

kubectl get svc -n backend
kubectl get endpoints mysql-service -n backend
```

- The Service should show port **3306**
- The Endpoints should show at least one IP (the MySQL Pod)

---

**Deploy WordPress in the Frontend Namespace**

**What:** We create a WordPress Pod in `frontend` that uses environment variables to connect to MySQL.

Usually we set variables like:

- `WORDPRESS_DB_HOST` → `mysql-service.backend.svc.cluster.local:3306`
- `WORDPRESS_DB_USER` → same as configured in MySQL
- `WORDPRESS_DB_PASSWORD`
- `WORDPRESS_DB_NAME` → e.g. `wordpress`

**Why:** The app and DB live in different namespaces, but Kubernetes DNS lets them talk to each other.

**Commands:**

```
kubectl apply -f wordpress-pod.yaml

kubectl get pods -n frontend

kubectl wait --for=condition=Ready pod/wordpress -n frontend --timeout=180s
```

If it doesn’t become Ready, check:

```
kubectl describe pod wordpress -n frontend
kubectl logs wordpress -n frontend --tail=50
```

---

**Test Connectivity from WordPress to MySQL**

**What:** We go *inside* the WordPress Pod and check if it can reach MySQL by DNS and by port.

**Why:** If WordPress shows “Error establishing database connection”, this is the first thing to check:

- DNS name correct?
- Port open?
- Service pointing to the right Pod?

**Commands:**

```
kubectl exec -n frontend -it wordpress -- bash -lc "apt-get update && apt-get install -y inetutils-ping netcat &&
ping -c1 mysql-service.backend.svc.cluster.local &&
nc -zv mysql-service.backend.svc.cluster.local 3306
"
```

If both `ping` and `nc` succeed, network connectivity is good. If you still see WordPress errors, it’s usually credentials or DB name.

---

**Access WordPress in the Browser**

**What:** We forward port 80 of the WordPress Pod to a port on our local machine (8080).

**Why:** This lets us test the app without exposing it publicly.

**Commands:**

```
kubectl port-forward -n frontend pod/wordpress 8080:80
```

Now open:

 `http://localhost:8080`

You should see:

- WordPress **installation / setup page**
- A form asking for **Site Title**, **Username**, **Password**, **Email**

If you see that screen, everything is working:

- Namespaces are correct
- Quotas aren’t blocking it
- WordPress can talk to MySQL
- The container is serving HTTP

---

 **Cleanup (Optional)**

If you want to remove everything:

```
kubectl delete -f wordpress-pod.yaml
kubectl delete -f mysql-service.yaml
kubectl delete -f db-pod.yaml
kubectl delete -f quotas.yaml
kubectl delete -f namespaces.yaml
```

This will delete:

- WordPress Pod
- MySQL Pod
- MySQL Service
- Quotas
- Namespaces (`frontend`, `backend`)

---

**What You Learned**

By completing this lab, you practiced how to:

- Use **namespaces** to separate frontend and backend
- Use **resource quotas** to control total CPU and memory per namespace
- Deploy **MySQL** as a backend database
- Deploy **WordPress** as a frontend application
- Use **Kubernetes DNS** to connect Pods across namespaces
- Verify and troubleshoot connectivity from within the Pod
- Access the application using `kubectl port-forward`

This pattern is very close to real workloads you’ll run as a DevOps / Cloud / Kubernetes engineer.