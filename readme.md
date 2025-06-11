## **Online Shopping Webapp Deployment on Kubernetes**

This repository provides a comprehensive guide to deploying a pre-built online shopping web application frontend on a Kubernetes cluster. It covers two deployment approaches: the traditional Deployment/Service method and an alternative method using a `kubernetes-mcp` server along with Amazon Q. Additionally, it includes instructions for setting up observability using the `kube-prometheus-stack` Helm chart.

**Prerequisite:** You should have a pre-built application container image hosted on a container registry like Docker Hub.

---

### **1. Project Structure**

```
.
├── helm/
│   └── install-prometheus.sh
├── kubernetes/
│   ├── traditional/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── mcp/
│       └── mcp-deployment.yaml
└── README.md
```

---

### **2. Kubernetes Deployment**

We provide two methods for deploying your application to a Kubernetes cluster. In the following YAML files, make sure to replace `your-dockerhub-image:latest` with the actual path to your image on Docker Hub (e.g., `myusername/my-shopping-app:1.2.0`).

#### **2.1. Traditional Deployment (Deployment/Service)**

This is the standard and most common way to deploy applications on Kubernetes.

**`kubernetes/traditional/deployment.yaml`**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shopping-app-deployment
  labels:
    app: shopping-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: shopping-app
  template:
    metadata:
      labels:
        app: shopping-app
    spec:
      containers:
      - name: shopping-app
        image: trainwithshubham/easyshop-app:latest 
        ports:
        - containerPort: 80
```

**`kubernetes/traditional/service.yaml`**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: shopping-app-service
spec:
  selector:
    app: shopping-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

To deploy, apply these manifests:
```bash
kubectl apply -f kubernetes/traditional/deployment.yaml
kubectl apply -f kubernetes/traditional/service.yaml
```

#### **2.2. Alternative Deployment (kubernetes-mcp)**

This method uses a  `kubernetes-mcp` (Multi-Cluster/Cloud Platform) server along with Amazon Q CLI, which simplifies deployment across multiple environments.

I was using the WSL interface and here are the commands to download Amazon Q CLI and the Kubernetes MCP Server

Download installation zip file:



    curl --proto '=https' --tlsv1.2 -sSf "https://desktop-release.q.us-east-1.amazonaws.com/latest/q-x86_64-linux.zip" -o "q.zip"
    Unzip the installer:unzip q.zip
    Run the install program:./q/install.shBy default, the files are installed to ~/.local/bin.
    
Install Amazon Q debian file

    sudo apt install -y ./amazon-q.deb
    q login


You can login with the free builder ID. Once done you can access the CLI with the command 'q'

Next step is to download the MCP Server which I have done using the uvx
    
    sudo snap install astral-uv --classic
    
Now create a file called mcp.json in ~/.aws/amazonq
and add this

    
    {
    "mcpServers": {
      "awslabs.cdk-mcp-server": {
        "command": "uvx",
        "args": ["awslabs.cdk-mcp-server@latest"],
        "env": {
          "FASTMCP_LOG_LEVEL": "ERROR"
        }
      },
      "kubernetes": {
        "command": "npx",
        "args": [
          "-y",
          "kubernetes-mcp-server@latest"
        ]
      }
    }  
    }



Now run q again
This gives you the capability to deploy kubernetes resources using prompts
---

### **3. Observability with Kube-Prometheus-Stack**

The `helm` directory contains a script to install the `kube-prometheus-stack`, which provides a full-featured monitoring and observability solution.

**`helm/install-prometheus.sh`**:
```bash
#!/bin/bash

# Add the prometheus-community Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update your local Helm chart repository cache
helm repo update

# Install the kube-prometheus-stack chart
helm install kube-prometheus-stack \

  --create-namespace \

  --namespace kube-prometheus-stack \

  prometheus-community/kube-prometheus-stack

```

Run this script to set up your monitoring stack:
```bash
bash ./helm/install-prometheus.sh
```
