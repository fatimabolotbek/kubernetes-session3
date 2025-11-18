### Running Pod with Arguments – Kubernetes

This small exercise is about learning how to run a Pod in Kubernetes using **command-line arguments** args  and then exporting Pod details to a JSON file. The main idea:Instead of hardcoding the command in the image, we use Kubernetes to pass arguments to the container at runtime.

### Test Requirements

```bash
- Create a Pod manifest saved as: `$(pwd)/pod.yaml`.
- Pod name: **`akumo-pod`**
- Container name: **`akumo-container`**
- Image: **`busybox`**
- The container should run this logic: 
 /bin/sh -c "sleep 5000"
```

- We must use **only `args`**, not the `command` field.
- After the Pod is running, we need to:
    - Get the Pod details in JSON format
    - Save them to: `$(pwd)/out.json`

### How I Ran the Test

create the pod 

```jsx
kubectl apply -f pod.yaml

```

Verify the pod is running 

```jsx
kubectl get pods
```

Export Pod details to JSON:

```jsx
kubectl get pod akumo-pod -o json > out.json
```

Check files in the folder:

```jsx
ls
# pod.yaml  out.json
```

## Key Takeaways

- You can control container behavior using **args** without changing the image.
- args are useful when:
    - You want to reuse the same image with different behaviors.
    - You want to keep image simple and move logic to Kubernetes manifests.
- Exporting Pod info to JSON with:
    
    ```bash
    kubectl get pod akumo-pod -o json > out.json
    
    ```
    
    is a common pattern for debugging and scripting.
    ### Understanding `command` vs `args` in Kubernetes

- In containers (Docker), there are **two parts**:
    - **entrypoint** → in Kubernetes: `command`
    - **cmd / parameters** → in Kubernetes: `args`

## Summary of concepts this test checks

1. **Pod basics** – writing a Pod manifest in YAML.
2. **Using `args` to control container behavior** without changing the image.
3. **Keeping a pod running** using `sleep` for debugging.
4. **Exporting Kubernetes objects to JSON** for automation and inspection.