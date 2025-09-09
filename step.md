Got it 👍 — let’s rewrite your **“Managing Microservices using Istio Component”** guide into a cleaner, step-by-step process. I’ll organize it by phases, highlight the tools, and make it easier for someone new to follow.

---

# 🚀 Managing Microservices with Istio, ArgoCD & Argo Rollouts

## **1. Prerequisites**

* **OS**: Ubuntu 24.04 LTS (with sudo privileges)
* **Resources**: 32 GB RAM, 4+ vCPU, 40 GB disk
* **Tools**: Docker / KVM / VirtualBox
* **Cluster**: Minikube (local) or EC2 (cloud)

---

## **2. Install Docker**

```bash
sudo apt update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
newgrp docker
```

---

## **3. Install Minikube**

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
minikube start --cpus=8 --memory=14000
```

---

## **4. Install Kubectl**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

---

## **5. Install Argo Rollouts**

```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
kubectl get all -n argo-rollouts
```

Install the plugin:

```bash
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
kubectl argo rollouts version
```

Access the dashboard:

```bash
kubectl argo rollouts dashboard
```

👉 Open **http\://\<EC2\_Public\_IP>:3100**

---

## **6. Install ArgoCD**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Expose it:

```bash
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443
```

Get password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Login:

```bash
argocd login <EC2_Public_IP>:8080
```

---

## **7. Install Istio**

Download & setup:

```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.27.1
export PATH=$PWD/bin:$PATH
```

Install demo profile:

```bash
istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
```

Enable sidecar injection:

```bash
kubectl label namespace default istio-injection=enabled
kubectl get ns default --show-labels
```

---

## **8. Deploy Microservices**

* Connect ArgoCD to your GitHub repo
* Deploy all microservices through **applications.yaml**
* Verify pods & services

---

## **9. Deploy Istio Addons**

```bash
kubectl apply -f samples/addons
kubectl get svc -n istio-system
```

Access Kiali dashboard:

```bash
kubectl port-forward svc/kiali -n istio-system --address 0.0.0.0 20001:20001
```

👉 Open **http\://\<EC2\_Public\_IP>:20001**

---

## **10. Canary & Rollbacks with Argo Rollouts**

* Create rollout manifests for services (e.g., `emailservice`)
* Use **Istio VirtualService** + **Argo Rollouts** for traffic shifting
* Rollback quickly with:

```bash
kubectl argo rollouts undo <ROLL_OUT_NAME>
```

---

✅ With this setup, you now have:

* Declarative GitOps deployments via **ArgoCD**
* Canary/Blue-Green rollouts via **Argo Rollouts**
* Observability & traffic control via **Istio + Kiali**
