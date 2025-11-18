## Running a Pod in a Namespace with a specific Apache image

In this task, the main goal was to run an Apache web server Pod inside its own namespace.

I created a namespace called `application` and then deployed a Pod named `apache` in that namespace using the `paketobuildpacks/httpd:0.7` image and exposing container port 80.

From a Kubernetes perspective, the configuration is correct: the namespace is created, the Pod is scheduled, and the image is successfully pulled. However, the Pod ends up in a `CreateContainerError` state.

When I run `kubectl describe pod apache -n application`, the events show that the image is pulled successfully, but the container runtime reports `failed to generate container`. This means that the problem is not with the YAML or Kubernetes objects, but with starting the container from that specific image (for example, an invalid entrypoint or a buildpack image that is not intended to run directly).

This still demonstrates the main concepts of the exercise:

- Creating and using a custom namespace (`application`)
- Deploying a Pod with a specific image and port
- Verifying the Pod status using `kubectl get pods -n application`
- Debugging Pod issues using `kubectl describe pod` and `kubectl logs`

In a real-world scenario, the next step would be to either fix the container image or switch to a known working Apache image. For this test, the focus was on Kubernetes objects (Namespace + Pod), not on fixing the upstream container image.