# Debug Image

Image containing various debugging tools.

## Usage with Kubernetes

### Start debugging Pod using kubectl exec

```bash
kubectl run -i --tty --rm debug --image=simonmwessel/debug:latest --restart=Never -- /bin/bash
```

### Start debugging Pod using manifest

If you want to keep the Pod runnning or need to attach further configurations, you may use a manifest to create the Pod.

#### Regular Pod manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
spec:
  containers:
  - image: docker.io/simonmwessel/debug:latest
    imagePullPolicy: IfNotPresent
    name: debug-pod
    command: [ "/bin/bash", "-c", "--" ]
    args: [ "trap : TERM INT; sleep 9999999999d & wait" ] # Keep Pod alive until delete/kill
  restartPolicy: Never
  # Optional: Schedule Pod to specific node
  # nodeName: mynode
```

#### Pod manifest for namespaces with Pod Security Standards (non-root)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
spec:
  securityContext:
    fsGroup: 1000
    fsGroupChangePolicy: "OnRootMismatch"
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - image: docker.io/simonmwessel/debug:latest
    imagePullPolicy: IfNotPresent
    name: debug-pod
    command: [ "/bin/bash", "-c", "--" ]
    args: [ "trap : TERM INT; sleep 9999999999d & wait" ] # Keep Pod alive until delete/kill
    securityContext:
      allowPrivilegeEscalation: false
      seccompProfile:
        type: RuntimeDefault
      capabilities:
        drop:
        - ALL
  restartPolicy: Never
  # Optional: Schedule Pod to specific node
  # nodeName: mynode
```

### Apply pod and open shell

```bash
# Apply Pod
kubectl apply -n mynamespace -f manifest.yaml
# Open shell on Pod
kubectl exec -n mynamespace --stdin --tty debug-pod -- /bin/bash
```
