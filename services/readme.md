Project overview
This repository contains a small Kubernetes demo with two Deployments and two Services:

backend (http-echo)
Deployment: backend-deploy (3 replicas)
Service: backend-svc (ClusterIP)
frontend (nginx)
Deployment: frontend-deploy (3 replicas)
Service: frontend-svc (NodePort, nodePort: 31000)
The manifests are:

backend.yaml
backend-service.yaml (backend-svc)
frontend.yaml
frontend-svc.yaml (frontend-svc)
This repo demonstrates:

Creating Deployments and Services (imperative + declarative)
Service types: ClusterIP and NodePort
Port mapping: service.port, targetPort, and nodePort
Using kubectl create/expose to generate YAML and editing it
Prerequisites
kubectl configured to talk to a Kubernetes cluster (this demo used Minikube)
minikube (if you want the local cluster experience)


How to apply manifests
Apply the backend:
kubectl apply -f backend.yaml
kubectl apply -f backend-service.yaml
Apply the frontend:
kubectl apply -f frontend.yaml
kubectl apply -f frontend-svc.yaml
Verify resources:
kubectl get deploy
kubectl get pods -o wide
kubectl get svc -o wide
Describe to debug:
kubectl describe deploy frontend-deploy
kubectl describe svc frontend-svc
kubectl describe svc backend-svc
Accessing the frontend app (Minikube)
NodePort approach:
Use the Minikube node IP + nodePort. Example used in this demo: http://192.168.49.2:31000
If that URL doesn't work from your host, use:
minikube service frontend-svc --url
or port-forward: kubectl port-forward svc/frontend-svc 8080:80 then open http://localhost:8080
Troubleshooting:
Check endpoints: kubectl describe svc frontend-svc — ensure Endpoints are populated.
Confirm pods have the correct label: kubectl get pods -l app=frontend
If behind WSL2 or Docker networking, prefer minikube service or port-forward.
Notes on common issues (learned while building)
Always ensure Deployment selector.matchLabels exactly matches template.metadata.labels. A small typo (e.g., frontdend vs frontend) will cause Deployment validation to fail.
spec.template.spec.containers must be a YAML list (use a leading - before each container block).
Use kubectl explain to inspect schema (e.g., kubectl explain deployment.spec.template.spec.containers).
Use --dry-run=client -o yaml with kubectl create to generate starter manifests, then edit them instead of writing from scratch.
The Deployment selector becomes immutable after creation; to change it you must delete and recreate the Deployment.
Useful commands (history collected)
The following are the main commands used while building and troubleshooting this demo (imperative + declarative flow):

Generate Deployment YAML (imperative -> declarative)
kubectl create deployment backend-deploy --image=hashicorp/http-echo --replicas=3 --port=5678 --dry-run=client -o yaml > backend.yaml
kubectl create deploy frontend-deploy --image=nginx --replicas=3 --port=80 --dry-run=client -o yaml > frontend.yaml
Apply manifests
kubectl apply -f backend.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f frontend.yaml
kubectl apply -f frontend-svc.yaml
Dry-run & validation
kubectl apply -f frontend.yaml --dry-run=client
kubectl apply -f backend-service.yaml --dry-run=client
Expose a deployment (imperative)
kubectl expose deploy backend-deploy --type=ClusterIP --port=9090 --target-port=5678 --dry-run=client -o yaml > backend-svc.yaml
kubectl expose deploy frontend-deploy --type=NodePort --name=frontend-svc --port=80 --targetPort=80 --dry-run=client -o yaml > frontend-svc.yaml
Inspect resources
kubectl get deploy
kubectl get pods -o wide
kubectl get svc -o wide
kubectl describe svc frontend-svc
kubectl describe deploy frontend-deploy
Port-forward & testing
kubectl port-forward svc/frontend-svc 8080:80
minikube service frontend-svc --url
kubectl run --rm -it curl-test --image=radial/busyboxplus:curl --restart=Never -- /bin/sh
curl http://frontend-svc:80 (from inside cluster)
Cleanup
kubectl delete deploy frontend-deploy
kubectl delete deploy backend-deploy
kubectl delete svc frontend-svc
kubectl delete svc backend-svc
Helpful schema checks
kubectl explain deployment.spec.template.spec.containers
kubectl explain Service.spec.ports

Version control your YAML files and include a clear README (this file).
Next steps / enhancements
Add Health/readiness probes to Deployments.
Add resource requests/limits for pods.
Create an Ingress with an Ingress Controller (Minikube addon) to serve apps on standard ports.
Add CI to validate YAML (kubeval or kubectl apply --server-dry-run).

