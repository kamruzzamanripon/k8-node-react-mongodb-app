## 📝 Introduction About APP:

This project aims to provide a real-time chat experience that's both scalable and secure. With a focus on modern technologies, we're building an application that's easy to use and maintain.


## Detailed Workflow Description:


![image](https://github.com/user-attachments/assets/f845a188-8e70-42f7-8577-30af38e83053)


## ✨ Features:


* **Real-time Messaging**: Send and receive messages instantly using Socket.io 
* **User Authentication & Authorization**: Securely manage user access with JWT 
* **Scalable & Secure Architecture**: Built to handle large volumes of traffic and data 
* **Modern UI Design**: A user-friendly interface crafted with React and TailwindCSS 
* **Profile Management**: Users can upload and update their profile pictures 
* **Online Status**: View real-time online/offline status of users 


## 🛠️ Tech Stack:


* **Backend:** Node.js, Express, MongoDB, Socket.io
* **Frontend:** React, TailwindCSS
* **Containerization:** Docker
* **Orchestration:** Kubernetes (planned)
* **Web Server:** Nginx
* **State Management:** Zustand
* **Authentication:** JWT
* **Styling Components:** DaisyUI


## 🔧 Prerequisites:


* **[Node.js](https://nodejs.org/)** (v14 or higher)
* **[Docker](https://www.docker.com/get-started)** (for containerizing the app)
* **[Git](https://git-scm.com/downloads)** (to clone the repository)


## 📝 Setup .env File:


1. Navigate to the `backend` directory:
```bash
cd backend
```
2. Create a `.env` file and add the following content (modify the values as needed):
```env
MONGODB_URI=mongodb://mongoadmin:secret@mongodb:27017/dbname?authSource=admin
JWT_SECRET=your_jwt_secret_key
PORT=5001
```
> **Note:** Replace `your_jwt_secret_key` with a strong secret key of your choice.


## 🏗️ Build and Run the Application"

Follow these steps to build and run the application:

1. Build & Run the Containers:

```bash
cd full-stack_chatApp
```
```bash
docker-compose up -d --build
```

2. Access the application in your browser:

```
http://localhost
```

## 📚 Project Snapshots:

![Settings](frontend/public/settings.png)

![chat](frontend/public/chat.png)

![logout](/frontend/public/logout.png)

![Login](/frontend/public/login.png)

## 🛠️ Getting Started For k8s

#
> [!Important]
> Below table helps you to navigate to the particular tool installation section fast.

| Tech stack    | Installation |
| -------- | ------- |
| Docker  | <a href="#docker">Install and configure Docker</a>     |
| Kind & Kubectl | <a href="#kind">Install and configure Kind & Kubectl</a>     |
| Metallb | <a href="#metallb">Install Metallb</a>     |
| Ingress | <a href="#ingress">Install and configure Ingress</a>     |
| Helm | <a href="#helm">Helm Install and configure</a>     |
| SSL Certificate | <a href="#ssl_certificate">Install and configure Cert Manager</a>     |
| Project Deploy | <a href="#project">Project Deploy and Others</a>
| Monitoring | <a href="#monitor">Namespace Create for Groping Prometheus and grafana and Other</a>
| Prometheus | <a href="#prometheus">Install and configure Prometheus</a>     |
| Grafana | <a href="#grafana">Install and configure Grafana</a>     |
| Clean Up | <a href="#Clean">Clean up</a>     |
#

### Pre-requisites to implement this project:
#

> [!Note]
> vps minimum need 

- <b>RAM - 4GB</b>
- <b>CPU - 2 Core(s)</b>
- <b>Storage - 20 GB</b>

#
- ## <b id="docker">Install and configure Docker</b>
```bash
sudo apt-get update

sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker

```

#
- ## <b id="kind">Install and configure Kind & Kubectl</b>
Install KIND and kubectl using the provided script:
```bash

#!/bin/bash

# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-linux-amd64
chmod +x ./kind
sudo cp ./kind /usr/local/bin/kind

VERSION="v1.31.0"
URL="https://dl.k8s.io/release/${VERSION}/bin/linux/amd64/kubectl"
INSTALL_DIR="/usr/local/bin"

curl -LO "$URL"
chmod +x kubectl
sudo mv kubectl $INSTALL_DIR/
kubectl version --client

rm -f kubectl
rm -rf kind

echo "kind & kubectl installation complete."
```
> [!Note]
> Run this script and it cerate kubectl and kind environment

### 2. Setting Up the KIND Cluster
#### Create a kind-cluster-config.yaml file:

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```
Create the cluster using the configuration file:
```
kind create cluster --config kind-cluster-config.yaml --name my-kind-cluster
```
Verify the cluster:
```
kubectl get nodes
kubectl cluster-info
```
> [!Note]
> Here i add extraPortMappings for running Ingress


#
- ## <b id="metallb">Install Metallb</b>
> [!Note]
> I am using Metallb for use LoadBalance. Suppose you are using Aws/Azure/DigitalOcean ect whose provide kubernates loadBalance facility then doesn't need Metallb. Here I buy VPS from a local company. They give one IP address to access VPS.

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-frr.yaml

```
Check Metallb configuration
```
kubectl get all -n metallb-system
```
![image](/asserts/images/Screenshot_5.jpg)

#### Create a metallb_config.yaml file:
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 160.191.163.33-160.191.163.33
```

apply metallb_config.yaml file
```
kubectl apply -f metallb_config.yaml
```
> [!Note]
> MY VPS IP address is 160.191.163.33. Change this IP as your require



#
- ## <b id="ingress">Install and configure Ingress</b>

```
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```
![image](/asserts/images/Screenshot_6.jpg)

> [!Note]
> Here service/ingress-nginx-controller show  EXTERNAL-IP is your VPS IP. My VPS IP is 160.191.163.33. It ensure that Our Metallb LoadBalance wroking.




#
- ## <b id="helm">Helm Install and configure</b>
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh

./get_helm.sh
```
Check Helm Version
```
helm version
```
![image](/asserts/images/Screenshot_7.jpg)




#
- ## <b id="ssl_certificate">Install and configure Cert Manager || SSL Certificate</b>
```
helm repo add jetstack https://charts.jetstack.io --force-update
helm repo update
```
Installing cert-manager CRDs
```
helm install cert-manager --namespace cert-manager --version v1.16.2 jetstack/cert-manager
```
https://artifacthub.io/packages/helm/cert-manager/cert-manager





#
- ## <b id="project">Project Deploy and Others</b>









## 📜 License


This project is licensed under the MIT License. See the LICENSE file for more details.