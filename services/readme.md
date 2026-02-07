Project overview
This repository demonstrates a small Kubernetes application with two Deployments and two Services, using both imperative and declarative workflows. It’s built for local testing (Minikube) and illustrates service types, port mapping, and common troubleshooting steps.

Backend
Deployment: backend-deploy (hashicorp/http-echo)
Service: backend-svc (ClusterIP)
Frontend
Deployment: frontend-deploy (nginx)
Service: frontend-svc (NodePort, nodePort: 31000)
Manifests included:

backend.yaml
backend-service.yaml
frontend.yaml
frontend-svc.yaml
Prerequisites
kubectl configured to target your cluster
Minikube (recommended for this demo) or another local Kubernetes cluster
Docker (for local images, if needed)
Basic familiarity with kubectl and YAML

Apply manifests (declarative)
Create backend resources:
kubectl apply -f backend.yaml
kubectl apply -f backend-service.yaml
Create frontend resources:
kubectl apply -f frontend.yaml
kubectl apply -f frontend-svc.yaml
Generate manifests (imperative → declarative)
You can generate starter YAML with kubectl create and then edit the files:

Backend deployment (generate YAML):
kubectl create deployment backend-deploy --image=hashicorp/http-echo --replicas=3 --port=5678 --dry-run=client -o yaml > backend.yaml
Frontend deployment (generate YAML):
kubectl create deployment frontend-deploy --image=nginx --replicas=3 --port=80 --dry-run=client -o yaml > frontend.yaml
Create service YAML from a deployment:
kubectl expose deploy backend-deploy --type=ClusterIP --port=9090 --target-port=5678 --dry-run=client -o yaml > backend-service.yaml
kubectl expose deploy frontend-deploy --type=NodePort --name=frontend-svc --port=80 --target-port=80 --dry-run=client -o yaml > frontend-svc.yaml
Notes:

Use --dry-run=client -o yaml to preview YAML before applying.
Edit generated YAML to add arguments, labels, or other configurations.
How to access the frontend app (Minikube)
Option A — NodePort (example values)

Node IP (example from this demo): 192.168.49.2
NodePort: 31000
Browser URL: http://192.168.49.2:31000
Option B — Let Minikube open the service:

minikube service frontend-svc
Or get a working URL:
minikube service frontend-svc --url
Option C — Port-forward (if node IP isn’t reachable):

kubectl port-forward svc/frontend-svc 8080:80
Browser URL: http://localhost:8080
Troubleshooting:

kubectl describe svc frontend-svc — check Endpoints (should list pod IPs).
kubectl get pods -l app=frontend -o wide — ensure pods are running and have the correct labels.
Ensure host firewall or WSL2 networking does not block the nodePort.
Verify & debug commands
kubectl get deploy
kubectl get pods -o wide
kubectl get svc -o wide
kubectl describe deploy frontend-deploy
kubectl describe svc frontend-svc
kubectl describe svc backend-svc
kubectl logs
kubectl run --rm -it curl-test --image=radial/busyboxplus:curl --restart=Never -- /bin/sh
Inside pod: curl http://frontend-svc:80
Cleanup
kubectl delete -f frontend-svc.yaml
kubectl delete -f frontend.yaml
kubectl delete -f backend-service.yaml
kubectl delete -f backend.yaml
