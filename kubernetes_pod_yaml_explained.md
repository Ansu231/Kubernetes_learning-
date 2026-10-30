# Kubernetes Pod and `pod.yaml` — Complete Explanation in Hinglish

This guide focuses on the `pod.yaml` file: what it contains, why we write it, how Kubernetes reads it, and what each field means.

## 1. Relationship between `pod.yaml` and a running Pod

Think of `pod.yaml` as an instruction file. You describe the Pod you want Kubernetes to run. You then submit that configuration to Kubernetes.

```text
pod.yaml
   |
   v
kubectl apply -f pod.yaml
   |
   v
Kubernetes API Server
   |
   v
Desired state is stored in the cluster
   |
   v
Scheduler selects a suitable worker node
   |
   v
Kubelet + container runtime start the container
   |
   v
Pod runs the application (for example, Nginx)
```

**Important:** The YAML file does not run the application by itself. It describes the configuration. Kubernetes acts on that configuration when you submit it.

## 2. What does a `pod.yaml` file contain?

A Kubernetes Pod manifest commonly contains four main sections:

| Field | Meaning | Purpose |
|---|---|---|
| `apiVersion` | API version | Identifies the API version used for this resource |
| `kind` | Resource type | Says that you want to create a Pod |
| `metadata` | Pod information | Defines its name, namespace, labels, and other metadata |
| `spec` | Desired configuration | Defines containers, images, ports, resources, and other settings |

These are the main building blocks. Under `spec`, you can configure additional options depending on the application's requirements.

## 3. Complete `pod.yaml` example

This example defines a Pod running an Nginx web server.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: dev

spec:
  containers:
    - name: nginx-container
      image: nginx:1.28

      ports:
        - containerPort: 80

      resources:
        requests:
          cpu: "100m"
          memory: "64Mi"
        limits:
          cpu: "500m"
          memory: "128Mi"

      env:
        - name: APP_ENV
          value: "development"

      imagePullPolicy: IfNotPresent
```

This is an illustrative manifest. For production, select an image version supported by your environment and review its security and resource requirements.

## 4. Line-by-line explanation

### A. `apiVersion: v1`

```yaml
apiVersion: v1
```

Kubernetes APIs have different versions. `v1` identifies the API version used for the Pod resource.

- `apiVersion` is the field name.
- `v1` is its value.
- Pods use the core Kubernetes API, whose version is `v1`.

This is not the version of Nginx or the Kubernetes cluster.

### B. `kind: Pod`

```yaml
kind: Pod
```

This tells Kubernetes which type of resource you want to create.

| `kind` | What it creates |
|---|---|
| `Pod` | A Pod |
| `Deployment` | A Deployment that manages application replicas |
| `Service` | A stable network endpoint for accessing selected Pods |
| `ConfigMap` | A resource for non-sensitive configuration data |
| `Secret` | A resource intended for sensitive configuration data |

In this example, `kind: Pod` means you are defining a Pod directly rather than a Deployment.

### C. `metadata`

```yaml
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: dev
```

`metadata` contains information that identifies and classifies the Pod.

#### `name: nginx-pod`

This is the Pod's name within its namespace. You can use it to inspect the Pod:

```bash
kubectl get pod nginx-pod
kubectl describe pod nginx-pod
kubectl logs nginx-pod
```

#### `labels`

Labels are key-value pairs attached to Kubernetes resources. They help organize resources and select related Pods.

```yaml
labels:
  app: nginx
  environment: dev
```

- `app: nginx` indicates which application the Pod belongs to.
- `environment: dev` indicates the intended environment.

You can use these labels to find Pods:

```bash
kubectl get pods --show-labels
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev
```

A Service can use a label selector to direct traffic to Pods labeled `app: nginx`.

**Remember:** `name` identifies a particular Pod; `labels` help group and select resources.

### D. `spec`

```yaml
spec:
```

`spec` stands for specification. It defines the **desired state** of the Pod.

Simple Hinglish mein: aap Kubernetes ko bata rahe ho ki Pod ke andar kya run hona chahiye aur uski configuration kya honi chahiye.

You can define:

- Which containers should run.
- Which container images to use.
- Which ports containers expose.
- CPU and memory requests and limits.
- Environment variables.
- Volumes, security settings, probes, and other options.

Kubernetes uses this desired configuration to create and manage the Pod.

### E. `containers`

```yaml
spec:
  containers:
    - name: nginx-container
      image: nginx:1.28
```

A Pod can contain one or more containers. These containers share the same Pod network namespace and can communicate through `localhost`. They can also share storage when configured with a common volume.

#### Why is there a hyphen (`-`)?

```yaml
containers:
  - name: nginx-container
```

The hyphen indicates an item in a YAML list. Kubernetes allows a list of containers under `containers`.

#### `name: nginx-container`

This is the container's name inside the Pod. It is not the Pod's name and not the image name.

#### `image: nginx:1.28`

This identifies the container image Kubernetes should run.

- `nginx` is the image name.
- `1.28` is the image tag.

A container image packages the application and the files it needs to run. Kubernetes uses the image to start the container on the worker node.

If the image is unavailable locally, Kubernetes may need to pull it from a container registry. If the image cannot be obtained, the Pod may encounter `ErrImagePull` or `ImagePullBackOff`.

### F. `ports` and `containerPort`

```yaml
ports:
  - containerPort: 80
```

Nginx commonly listens for HTTP requests on port `80` inside the container.

`containerPort: 80` documents the port the container is expected to listen on.

This field **does not** automatically expose the Pod to the internet, create a Kubernetes Service, or publish the port on the worker node.

To make an application reachable through a stable Kubernetes endpoint, you would commonly create a Service. External access may require additional configuration, such as a LoadBalancer Service or an Ingress.

### G. `resources`

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "64Mi"
  limits:
    cpu: "500m"
    memory: "128Mi"
```

This section defines the container's CPU and memory requirements and constraints.

#### Requests — scheduling requirements

Kubernetes uses requested CPU and memory when determining whether a node has enough allocatable resources to schedule the Pod.

- `cpu: "100m"` means 0.1 CPU core.
- `memory: "64Mi"` means 64 mebibytes of memory.

#### Limits — resource constraints

These define the maximum resource usage permitted for the container.

- `cpu: "500m"` means half a CPU core.
- `memory: "128Mi"` means 128 mebibytes of memory.

CPU usage is throttled when the container hits its CPU limit. If it exceeds its memory limit, it may be terminated with an `OOMKilled` reason.

**Practical example:** Suppose a worker node has enough available memory to satisfy a request of 64 MiB. That does not mean Kubernetes reserves the full 128 MiB limit when scheduling the Pod. The scheduler primarily evaluates resource requests against the node's available allocatable resources. The node must also have sufficient resources for its other Pods and system workloads.

### H. `env`

```yaml
env:
  - name: APP_ENV
    value: "development"
```

Environment variables pass configuration into the container.

- `name: APP_ENV` is the variable's name.
- `value: "development"` is its value.

The application can read this variable while it runs. An application might use it to decide which configuration to load.

You can define multiple environment variables by adding more list entries under `env`.

For sensitive values such as passwords or API tokens, avoid hardcoding them in a manifest. Use an appropriately configured Kubernetes Secret or an external secret-management system. Secrets also require suitable access controls and encryption practices.

### I. `imagePullPolicy`

```yaml
imagePullPolicy: IfNotPresent
```

This controls when Kubernetes attempts to pull the container image.

| Value | Meaning |
|---|---|
| `IfNotPresent` | Pull the image if it is not already present locally |
| `Always` | Ask the container runtime to resolve and pull the image when starting the container |
| `Never` | Do not pull; the image must already be available locally |

`IfNotPresent` is suitable for illustrating a version-tagged image. With mutable tags, a cached image can differ from the image currently stored in the registry. Production environments commonly prefer immutable image digests when reproducibility is important.

## 5. YAML syntax and indentation

YAML uses indentation to show which fields belong inside other fields.

```yaml
metadata:
  name: nginx-pod
  labels:
    app: nginx
```

- `metadata` is the parent field.
- `name` and `labels` are nested under `metadata`.
- `app: nginx` is nested under `labels`.

Important YAML rules:

- Use consistent spaces for indentation; do not use tabs for indentation.
- A colon (`:`) separates a key from its value.
- A hyphen (`-`) indicates an item in a list.
- Child fields must be indented under their parent.
- Field names and values are case-sensitive where Kubernetes expects specific values.

For example, `kind: Pod` is correct. `kind: pod` is not the conventional valid resource kind.

## 6. What happens when you apply `pod.yaml`?

Save the file as `pod.yaml` and run:

```bash
kubectl apply -f pod.yaml
```

Conceptually, the following happens:

1. **`kubectl` reads the file and sends the configuration** to the Kubernetes API Server.
2. **The API Server validates the request**, including authorization and applicable validation rules.
3. **Kubernetes records the desired state** in the cluster's backing data store, etcd, after admission and validation steps succeed.
4. **The scheduler selects a worker node** for a newly created, unscheduled Pod. It evaluates node suitability, including resource requests and scheduling constraints.
5. **The kubelet and container runtime start the container.** The kubelet on the selected node works with the runtime to obtain the image if needed and start the container.

Creating the Pod object and getting the application to a running state are different things. The `apply` command may succeed even if the container subsequently fails to start.

## 7. Commands to work with your Pod

| Command | What it does |
|---|---|
| `kubectl apply -f pod.yaml` | Creates or updates the resource described in the file |
| `kubectl get pods` | Lists Pods in the current namespace |
| `kubectl get pod nginx-pod -o wide` | Shows more details, including the assigned node and Pod IP |
| `kubectl describe pod nginx-pod` | Shows configuration, conditions, and recent events |
| `kubectl logs nginx-pod` | Displays the container's logs |
| `kubectl exec -it nginx-pod -- /bin/sh` | Opens a shell inside the container, if a shell is available |
| `kubectl delete -f pod.yaml` | Deletes the resource described in the file |

To validate the YAML against the cluster without applying it:

```bash
kubectl apply --dry-run=server -f pod.yaml
```

This asks the API Server to validate the request without persisting the resource. It requires a reachable cluster and appropriate permissions.

To check the Pod's status:

```bash
kubectl get pod nginx-pod
```

Possible statuses include `Pending`, `Running`, `Succeeded`, `Failed`, and `Unknown`. A `Running` Pod does not necessarily prove that the application is healthy or serving traffic correctly.

## 8. Other important fields you may see in `pod.yaml`

You do not need every field in every Pod. Use fields according to the application's requirements.

| Field | Purpose |
|---|---|
| `namespace` | Under `metadata`; identifies the namespace in which the Pod belongs |
| `restartPolicy` | Controls whether containers restart after exiting: `Always`, `OnFailure`, or `Never` |
| `command` | Overrides the container image's default entrypoint |
| `args` | Supplies arguments to the container process |
| `volumeMounts` | Mounts configured volumes inside a container |
| `volumes` | Defines storage or other volume sources available to the Pod |
| `livenessProbe` | Checks whether a container should be restarted |
| `readinessProbe` | Checks whether the container is ready to receive traffic |
| `startupProbe` | Gives a slow-starting application time to initialize before other probes take effect |
| `securityContext` | Configures security-related settings for the Pod or container |
| `nodeSelector` | Restricts scheduling to nodes with matching labels |
| `affinity` | Defines advanced node or Pod placement preferences and requirements |
| `tolerations` | Allows a Pod to be scheduled onto nodes with matching taints, subject to other scheduling constraints |
| `serviceAccountName` | Specifies the ServiceAccount identity used by the Pod |

### Example 1: Namespace

```yaml
metadata:
  name: nginx-pod
  namespace: dev
```

This places the Pod in the `dev` namespace, assuming that namespace exists.

Namespaces help separate and organize resources within a cluster. A Pod called `nginx-pod` can exist in multiple namespaces because names only need to be unique within their namespace.

### Example 2: Liveness probe

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 10
```

- `httpGet` tells Kubernetes to make an HTTP request.
- `path: /` is the endpoint to check.
- `port: 80` is the container port to check.
- `initialDelaySeconds: 10` delays the initial probe for 10 seconds after the container starts.
- `periodSeconds: 10` sets the interval between probes.

If the liveness probe repeatedly fails, Kubernetes restarts the affected container according to the probe and restart-policy behavior.

A liveness probe differs from a readiness probe. Readiness determines whether the Pod should receive traffic through compatible Service endpoints, while liveness helps detect containers that need restarting.

### Example 3: Temporary storage

```yaml
volumeMounts:
  - name: app-storage
    mountPath: /data

volumes:
  - name: app-storage
    emptyDir: {}
```

This creates a temporary directory volume for the Pod and mounts it at `/data` inside the container.

The application can read and write files there. The volume survives a container restart while the same Pod remains, but its contents are deleted when the Pod is removed.

For data that must persist beyond the lifetime of a Pod, you would typically consider a PersistentVolumeClaim and an appropriate storage backend.

## 9. Pod YAML versus Deployment YAML

In real-world Kubernetes projects, you will often see `deployment.yaml` instead of a standalone `pod.yaml`.

### `pod.yaml`

Defines one Pod directly. If the Pod is deleted, Kubernetes does not automatically recreate that standalone Pod just because the original YAML file still exists on your machine.

### `deployment.yaml`

Defines a Deployment, which manages Pods through ReplicaSets. If a managed Pod disappears unexpectedly, the Deployment controller works to restore the desired number of replicas.

Example Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.28
```

Notice:

- The Deployment uses `apiVersion: apps/v1` because it is a different resource.
- `replicas: 3` expresses the desired number of Pod replicas.
- `selector.matchLabels` identifies the Pods the Deployment manages.
- `template` defines the metadata and specification for the Pods it creates.

## 10. Troubleshooting a Pod created from YAML

Imagine you applied `pod.yaml`, but the Nginx Pod is not working.

### Step 1 — Check the status

```bash
kubectl get pods
```

Check whether the Pod is `Pending`, `Running`, or in another state.

### Step 2 — Inspect the Pod

```bash
kubectl describe pod nginx-pod
```

Look at the Events section for image-pull errors, scheduling failures, or container startup issues.

### Step 3 — Check the logs

```bash
kubectl logs nginx-pod
```

Use this to investigate application errors. If a container has restarted, this command can show logs from its previous instance when available:

```bash
kubectl logs nginx-pod --previous
```

### Step 4 — Review the manifest

Check the image, resource requests, environment variables, volume configuration, and other relevant fields in `pod.yaml`. Correct the cause, then apply the updated file.

## Final recap

Remember this basic structure:

```yaml
apiVersion: v1       # Which API version?
kind: Pod            # Which resource?
metadata:            # What is it called and how is it classified?
  name: nginx-pod
spec:                # What should it run and how?
  containers:
    - name: nginx-container
      image: nginx:1.28
```

- **`apiVersion`** — which API version to use.
- **`kind`** — what resource you want.
- **`metadata`** — the resource's identity and labels.
- **`spec`** — the desired configuration.
- **`containers`** — which application containers should run inside the Pod.

Think of `pod.yaml` as the desired-state description that Kubernetes uses to create and configure a Pod.
