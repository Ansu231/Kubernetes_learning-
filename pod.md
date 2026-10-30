# Kubernetes Pods — Beginner Notes

## 1. What is a Pod?

**Hinglish:** Pod Kubernetes ki smallest deployable unit hai. Iske andar ek ya multiple containers run kar sakte hain. Container application run karta hai, aur Pod us container ko Kubernetes mein run karne ke liye environment provide karta hai.

**Interview Answer:**

A Pod is the smallest deployable unit in Kubernetes. It encapsulates one or more containers that share a network namespace and can share storage volumes.

## 2. Pod Example

```text
Kubernetes Cluster
└── Worker Node
    └── Pod: nginx-pod
        └── Container: nginx-container
            └── Application: Nginx
```

- Node: Machine where Pods run.
- Pod: Kubernetes unit containing containers.
- Container: Runs the application.
- Image: Package used to create the container.

## 3. pod.yml Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

## 4. Explanation of YAML Fields

| Field | Meaning |
|---|---|
| `apiVersion: v1` | API version used for the Pod resource. |
| `kind: Pod` | Specifies that we want to create a Pod. |
| `metadata` | Contains identifying information. |
| `name: nginx-pod` | Name of the Pod. |
| `labels` | Key-value pairs used to organize and select resources. |
| `spec` | Defines the desired Pod configuration. |
| `containers` | Lists containers to run inside the Pod. |
| `name: nginx-container` | Name of the container. |
| `image: nginx` | Container image to run. |
| `containerPort: 80` | Documents the intended listening port. It does not expose the application externally by itself. |

## 5. How Does a Pod Work?

1. Create the `pod.yml` file.
2. Run `kubectl apply -f pod.yml`.
3. The API Server validates and processes the request.
4. The Scheduler selects a suitable Worker Node.
5. The Kubelet on that Node works with the container runtime to start the container.
6. Check the Pod status using `kubectl get pods`.

## 6. Important Commands

```bash
# Create or update a Pod from YAML
kubectl apply -f pod.yml

# Create a Pod directly
kubectl run nginx --image=nginx

# List Pods
kubectl get pods

# Show additional details
kubectl get pods -o wide

# Describe a Pod
kubectl describe pod nginx-pod

# View logs
kubectl logs nginx-pod

# Delete a Pod from its YAML definition
kubectl delete -f pod.yml
```

## 7. Important Concepts

- A Pod can contain one or more containers.
- Containers in the same Pod share a network namespace and can communicate using `localhost` on different ports.
- Every Pod has its own IP address.
- `containerPort: 80` does not automatically expose a Pod to the internet.
- If a container crashes, Kubernetes may restart it according to its restart policy.
- A Deployment helps maintain the desired number of application Pods.

## 8. Interview Summary

**What is a Pod?**

A Pod is the smallest deployable unit in Kubernetes. It contains one or more containers that share networking and can share storage.

**How does a Pod work?**

We define the desired configuration in a YAML file and submit it using `kubectl apply`. The API Server processes the request, the Scheduler assigns a suitable Node, and the Kubelet works with the container runtime to start the container.
***
**Remember:**

Node → Pod → Container → Application
<img width="917" height="590" alt="{A7B02790-27F2-47A5-9F71-BBBE8CE03B6B}" src="https://github.com/user-attachments/assets/2b82b481-07dc-40a6-ba80-5a2b7b889cff" />

