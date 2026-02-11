# End-to-End Cluster using Kind

This document serves as a comprehensive reference for creating, configuring, troubleshooting, and scaling a production-like Kubernetes cluster locally using **Kind** (Kubernetes using Docker).

## 1. Architecture Overview
We built a cluster designed to simulate a real-world environment on a local machine.
* **Cluster Type:** Multi-node (1 Control Plane, 2 Workers).
* **Ingress Controller:** NGINX (Maps `localhost:80` → Cluster).
* **Load Balancer:** MetalLB (Assigns unique IPs to services).
* **Networking:** Port mappings configured to allow direct traffic from the host machine.

## 2. Cluster Provisioning
### Prerequisites
* Docker Desktop / Docker Engine
* Kind 
* Kubectl

### Installation
The cluster configuration file `kind-config.yaml` maps local ports to the container and labels the control plane for Ingress traffic.

```bash
kind create cluster --config yaml/kind-config.yaml --name my-cluster

Output
Creating cluster "my-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.35.0) 🖼
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-my-cluster"

## 3. Networking Setup (The "Plumbing")
### A. Ingress Controller (NGINX)

We installed NGINX and patched it to run on the Control Plane. This resolved the "Connection Reset" error by ensuring the Ingress pod runs on the exact node mapped to Docker's port 80.

Install:
kubectl apply -f [https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml)

Patch (Critical Fix): Without this fix, the Ingress controller may schedule on worker nodes that aren't listening on the host's port 80.

kubectl patch deployment ingress-nginx-controller -n ingress-nginx --type='json' -p='[{"op": "add", "path": "/spec/template/spec/nodeSelector", "value": {"ingress-ready": "true"}}]'

### B. Load Balancer (MetalLB)

We installed MetalLB to provide "Real IP" functionality within the Docker network.

Install:
kubectl apply -f [https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml](https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml)

IP Configuration: The IP range was derived from the Docker bridge network: docker network inspect -f '{{.IPAM.Config}}' kind

# Apply your specific IPAddressPool and L2Advertisement
kubectl apply -f yaml/metallb-config.yaml

## 4. Application Deployment & Scaling

We transitioned from a simple echo server to a dynamic application (agnhost) to test load balancing across multiple replicas.

Manifest Highlights (scaling-test.yaml):

   1.  Deployment: 2 Replicas for high availability.
   2. Service: Type LoadBalancer to receive a dedicated MetalLB IP.

kubectl apply -f yaml/scaling-test.yaml

    Query successful

Gemini said

This is a fantastic README.md! It’s clear, technical, and follows a logical flow from architecture to verification.

To make this "GitHub-ready," I’ve applied standard Markdown styling, fixed the syntax highlighting for code blocks, and organized the troubleshooting section into a clean Markdown table.

You can copy the code block below directly into your repository.
Markdown

# End-to-End Cluster using Kind

This document serves as a comprehensive reference for creating, configuring, troubleshooting, and scaling a production-like Kubernetes cluster locally using **Kind** (Kubernetes in Docker).

## 1. Architecture Overview
We built a cluster designed to simulate a real-world environment on a local machine.
* **Cluster Type:** Multi-node (1 Control Plane, 2 Workers).
* **Ingress Controller:** NGINX (Maps `localhost:80` → Cluster).
* **Load Balancer:** MetalLB (Assigns unique IPs to services).
* **Networking:** Port mappings configured to allow direct traffic from the host machine.

## 2. Cluster Provisioning
### Prerequisites
* Docker Desktop / Docker Engine
* Kind (`brew install kind`)
* Kubectl

### Installation
The cluster configuration file `kind-config.yaml` maps local ports to the container and labels the control plane for Ingress traffic.

```bash
kind create cluster --config yaml/kind-config.yaml --name my-cluster

Expected Output:
Plaintext

Creating cluster "my-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.35.0) 🖼
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-my-cluster"

3. Networking Setup (The "Plumbing")
A. Ingress Controller (NGINX)

We installed NGINX and patched it to run on the Control Plane. This resolved the "Connection Reset" error by ensuring the Ingress pod runs on the exact node mapped to Docker's port 80.

Install:
Bash

kubectl apply -f [https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml)

Patch (Critical Fix): Without this fix, the Ingress controller may schedule on worker nodes that aren't listening on the host's port 80.
Bash

kubectl patch deployment ingress-nginx-controller -n ingress-nginx \
  --type='json' -p='[{"op": "add", "path": "/spec/template/spec/nodeSelector", "value": {"ingress-ready": "true"}}]'

B. Load Balancer (MetalLB)

We installed MetalLB to provide "Real IP" functionality within the Docker network.

Install:
Bash

kubectl apply -f [https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml](https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml)

IP Configuration: The IP range was derived from the Docker bridge network: docker network inspect -f '{{.IPAM.Config}}' kind
Bash

# Apply your specific IPAddressPool and L2Advertisement
kubectl apply -f yaml/metallb-config.yaml

## 4. Application Deployment & Scaling

We transitioned from a simple echo server to a dynamic application (agnhost) to test load balancing across multiple replicas.

Manifest Highlights (scaling-test.yaml):

    Deployment: 2 Replicas for high availability.

    Service: Type LoadBalancer to receive a dedicated MetalLB IP.

Bash

kubectl apply -f yaml/scaling-test.yaml

## 5. Troubleshooting Log

During setup, we encountered a Connection Reset error when accessing http://localhost/foo. Below is the diagnostic path used to resolve it.
Diagnostic Step	Action Taken	Result	Conclusion
1. App Check	kubectl port-forward pod/foo-app	Success	The Application Pod is healthy.
2. Service Check	kubectl get endpoints	Success	The Service found the Pod IPs correctly.
3. Node Location	kubectl get pods -o wide	Failed	The Ingress Controller was running on a Worker Node.

Root Cause: Docker only mapped Port 80 to the Control Plane. Traffic hitting the control plane found no listener because NGINX was on a different node. Resolution: Patched NGINX with a nodeSelector to force it onto the Control Plane.

## 6. Verification & Usage
Method 1: Via Ingress (Domain Style)

    URL: http://localhost/foo

    Mechanism: Docker Port 80 → Control Plane → NGINX Ingress → Service → Pod.

Method 2: Via LoadBalancer (IP Style)

    Get the IP: kubectl get svc foo-service-lb (e.g., 172.18.0.150).

    Command: curl 172.18.0.150:8080

    Mechanism: MetalLB assigns a Docker Network IP directly to the Service.

Test Scaling (Round Robin)

Run a loop to hit the LoadBalancer IP repeatedly to see traffic distribute across replicas:
for i in {1..10}; do curl 172.18.0.150:8080; echo; done

## 7. Cleanup

To remove the entire environment and free up Docker resources:
kind delete cluster --name my-cluster
