#  Cloud vs. Edge VNF Deployment: Performance Evaluation

A comparative analysis of Kubernetes environments for telecommunications Virtual Network Functions (VNFs), specifically evaluating a Cloud-like architecture (Minikube) against an Edge-like architecture (MicroK8s) using an Nginx ingress controller.

##  Prerequisites
To replicate this environment locally, you will need:
* **Host Machine:** macOS (Apple Silicon/Intel).
* **Virtualization CLI:** [Multipass](https://multipass.run/) installed to provision Ubuntu VMs.
* **Testing Tool:** Apache Bench (`ab`) installed on the host or inside the VMs.

---

##  Environment Setup & Installation


We use Multipass to create isolated, reproducible environments simulating a well-resourced Cloud node and a constrained Edge node.


```bash
# Launch Cloud VM (2 CPUs, 4GB RAM)
multipass launch jammy --name cloud-vm --cpus 2 --memory 4G --disk 20G

# Launch Edge VM (1 CPU, 2GB RAM)
multipass launch jammy --name edge-vm --cpus 1 --memory 2G --disk 10G
```
## Cloud Node Setup (Minikube)
The Cloud node utilizes a nested virtualization architecture via Docker and Minikube.
Access the Cloud VM:
```bash
multipass shell cloud-vm
```
Install Docker, Minikube, and Kubectl:
```bash
sudo apt update && sudo apt install -y curl apt-transport-https
# Install Docker
sudo apt install -y docker.io
sudo usermod -aG docker $USER && newgrp docker
# Install Minikube & Kubectl
curl -LO [https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64](https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64)
sudo install minikube-linux-amd64 /usr/local/bin/minikube
curl -LO "[https://dl.k8s.io/release/$(curl](https://dl.k8s.io/release/$(curl) -L -s [https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl](https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl)"
sudo install kubectl /usr/local/bin/kubectl
```
Start the Minikube cluster using the Docker driver:
```bash
minikube start --driver=docker
```
## Edge Node Setup (MicroK8s)
The Edge node utilizes a lightweight, bare-metal snap installation of Kubernetes, avoiding nested container overhead.
Access the Edge VM:
```bash
multipass shell edge-vm
```
Install MicroK8s:
```bash
sudo apt update
sudo snap install microk8s --classic

# Grant user permissions
sudo usermod -a -G microk8s $USER
newgrp microk8s

microk8s status --wait-ready
microk8s enable dns hostpath-storage metrics-server

sudo snap alias microk8s.kubectl kubectl
```
## Deploying the Virtual Network Function
```bash
kubectl apply -f deployments/nginx-deployment.yaml
kubectl apply -f services/nginx-service.yaml

kubectl get pods
```
### In the Edge VM
```bash
curl http://localhost:30080
```
### In the Cloud VM
```bash
minikube service nginx-service --url
curl http://192.168.49.2:30080/
```
##  Load Testing
Enable Metrics Server in both environments
```bash
watch -n 1 kubectl top pods
```
###Execute the Load Test:
```bash
ab -n 100000 -c 100 http://192.168.49.2:30080/
```
## Network Bandwidth Testing (iperf3)
```bash 
kubectl run iperf3-server --image=networkstatic/iperf3 -- -s
kubectl run iperf3-client --rm -it --image=networkstatic/iperf3 -- -c <IPERF3_SERVER_POD_IP>
ab -t 60 -c 100 http://192.168.49.2:30080/
```
### Cloud Environment (Minikube)
```bash
minikube addons enable metrics-server
kubectl top pods
minikube dashboard --url
```
### Edge Environment (MicroK8s)
```bash
sudo microk8s enable dashboard metrics-server
kubectl create token default
kubectl port-forward -n kubernetes-dashboard svc/kubernetes-dashboard-kong-proxy 10443:443 --address 0.0.0.0
```
