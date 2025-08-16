# Node App on Kubernetes

This repository contains Kubernetes manifests to deploy a simple Node.js application.

The manifests define a `Deployment` and a `Service` inside the `devops-gitops` Namespace, exposing the app via a `NodePort` Service.

## Repository layout

```
node-app/
  namespace.yaml     # Namespace: devops-gitops
  deployment.yaml    # Deployment: node-app (2 replicas), image beamerv12/devops-poc-node-app:latest, containerPort 3000
  service.yaml       # Service: node-app-service (NodePort), port 80 -> targetPort 3000
```

## Prerequisites

- kubectl installed and configured to talk to a Kubernetes cluster
- Cluster access with permissions to create Namespaces, Deployments, and Services
- Optional: `minikube` or `kind` for local testing

## Quick start

1) Create the Namespace:

```bash
kubectl apply -f node-app/namespace.yaml
```

2) Apply the Deployment and Service to that Namespace:

```bash
kubectl -n devops-gitops apply -f node-app/deployment.yaml
kubectl -n devops-gitops apply -f node-app/service.yaml
```

3) Verify resources:

```bash
kubectl get ns devops-gitops
kubectl -n devops-gitops get deploy,rs,pods,svc -l app=node-app
```

## Accessing the app

The Service is of type `NodePort`, exposing TCP port 80 which targets the container on port 3000.

- Option A: NodePort (cluster node IP)

```bash
NODE_PORT=$(kubectl -n devops-gitops get svc node-app-service -o jsonpath='{.spec.ports[0].nodePort}')
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "http://$NODE_IP:$NODE_PORT"
# Try the endpoint (adjust path if your app serves a specific route)
curl "http://$NODE_IP:$NODE_PORT/"
```

- Option B: Port-forward to localhost

```bash
kubectl -n devops-gitops port-forward svc/node-app-service 8080:80
```

Then visit `http://localhost:8080`.

- Option C: Minikube convenience URL

```bash
minikube service node-app-service -n devops-gitops --url
```

## Customize

- Image (in `node-app/deployment.yaml`):

```yaml
image: beamerv12/devops-poc-node-app:latest
```

To update the image without editing the file:

```bash
kubectl -n devops-gitops set image deployment/node-app node-app=<your-image>:<tag>
```

- Replicas:

```bash
kubectl -n devops-gitops scale deployment/node-app --replicas=3
```

- Rolling restart and status:

```bash
kubectl -n devops-gitops rollout restart deployment/node-app
kubectl -n devops-gitops rollout status deployment/node-app
```

- Logs:

```bash
kubectl -n devops-gitops logs deployment/node-app -f
```

## Cleanup

Delete the app resources:

```bash
kubectl -n devops-gitops delete -f node-app/service.yaml -f node-app/deployment.yaml
```

Optionally delete the Namespace:

```bash
kubectl delete ns devops-gitops
```

## Notes

- Deployment name: `node-app`, label `app: node-app`
- Service name: `node-app-service`, type `NodePort`, `port: 80 -> targetPort: 3000`
- Namespace: `devops-gitops`