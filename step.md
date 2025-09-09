**Managing Microservices using Istio component**

Prerequisites
Pre-Install Ubuntu 24.04 LTS Sudo User with admin privileges 32 GB RAM or more 4 CPU / vCPU or more 40 GB free 
hard disk space or more Docker / Virtual Machine Manager – KVM & VirtualBox

Install Docker
sudo apt update
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER
newgrp docker

curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
minikube start --cpus=8 --memory=14000

Install Kubectl

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl

Install Argo Rollouts controller on Minikube
Create the namespace for installation of the Argo Rollouts controller

kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
Now, you will see the controller and other components have been deployed. Wait for the pods to be in the Running state.

kubectl get all -n argo-rollouts
Install Argo Rollouts Kubectl plugin with curl for easy interaction with Rollout controller and resources with below command.

curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
kubectl argo rollouts version
Argo Rollouts comes with its own GUI as well that you can access with the below command.

kubectl argo rollouts dashboard
Now, you can access Argo Rollout console, by accessing http://Public IPv4 address:3100 on your browser. You would get the Public IPv4 address from instance created in your prerequites 

Install ArgoCD:
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

To expose it:
kubectl port-forward --address 0.0.0.0 svc/argocd-server 8080:443 -n argocd

Get the Admin Password

The initial username is admin. Get the auto-generated password:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

Login to ArgoCD CLI

Install argocd CLI (from ArgoCD releases ), then:

argocd login <ARGOCD_SERVER>

argocd login Public IPv4 address:8080

Injecting istio

Download:
curl -L https://istio.io/downloadIstio | sh -

Move to the Istio package directory. For example, if the package is istio-1.27.1:

$ cd istio-1.27.1

The installation directory contains:

Sample applications in samples/
The istioctl client binary in the bin/ directory.
Add the istioctl client to your path (Linux or macOS):

$ export PATH=$PWD/bin:$PATH

Install Istio using the demo profile, without any gateways:

$ istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
✔ Istio core installed
✔ Istiod installed
✔ Installation complete
Made this installation the default for injection and validation.

Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:

injecting the istio in the namespace
$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
 Confirm the istio is injected:
istio-1.27.1$ kubectl get ns default --show-labels

Deploy the microservice on Argocd by connecting to the repo

Deploy all the istio management 
kubectl apply -f samples/addons

kubectl get svc -n istio-system

kubectl port-forward svc/kiali -n istio-system --address 0.0.0.0 20001:20001

