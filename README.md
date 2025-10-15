<div align="center">

# 🚀 AKS with Application Gateway Ingress Controller (AGIC)

[![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)](https://azure.microsoft.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Terraform](https://img.shields.io/badge/Terraform-623CE4?style=for-the-badge&logo=terraform&logoColor=white)](https://terraform.io/)

*A comprehensive guide to setting up Azure Kubernetes Service (AKS) with Application Gateway Ingress Controller for scalable and secure web applications.*

</div>

---

## 📋 Table of Contents

- [🔍 Overview](#-overview)
- [🏗️ Architecture](#️-architecture)
- [📋 Prerequisites](#-prerequisites)
- [⚙️ Setup Instructions](#️-setup-instructions)
- [🚀 Deployment](#-deployment)
- [🧪 Testing](#-testing)
- [📊 Monitoring](#-monitoring)
- [🧹 Cleanup](#-cleanup)
- [👤 Author](#-author)

---

## 🔍 Overview

This project demonstrates how to integrate **Azure Kubernetes Service (AKS)** with **Application Gateway Ingress Controller (AGIC)** to provide:

- ✅ **Layer 7 load balancing** with SSL termination
- ✅ **Web Application Firewall (WAF)** protection
- ✅ **Auto-scaling** capabilities
- ✅ **High availability** and fault tolerance
- ✅ **Secure networking** with VNet peering

---

## 🏗️ Architecture

```
Internet → Application Gateway (WAF) → AKS Cluster → NGINX Pods
              ↓
         Public IP + SSL
```

**Key Components:**
- **Application Gateway**: Entry point with WAF protection
- **AKS Cluster**: Container orchestration platform
- **AGIC**: Kubernetes ingress controller
- **VNet Peering**: Secure communication between networks

---

## 📋 Prerequisites

Before you begin, ensure you have:

- [ ] **Azure CLI** installed and configured
- [ ] **kubectl** installed
- [ ] **Azure subscription** with appropriate permissions
- [ ] **Resource Group** created
- [ ] **AKS cluster** already provisioned

---

## ⚙️ Setup Instructions

### 1️⃣ Environment Variables

First, configure your environment variables:

```bash
# 🔧 Export your variables
export rgName=dev
export aksName=dev-aks
export pipName=dev-pip
export appgwVnetName=dev-appgw-vnet
export appgwSnetName=dev-appgw-snet-01
export location=westeurope
export appgwName=dev-appgw
export wafPolicyName=dev-waf-policy
export aksVnetName=dev-vnet
```

### 2️⃣ Infrastructure Setup

#### Create Public IP
```bash
# 🌐 Create public IP for Application Gateway
az network public-ip create \
  -n $pipName \
  -g $rgName \
  -l $location \
  --allocation-method Static \
  --sku Standard
```

#### Create Virtual Network
```bash
# 🏗️ Create VNet for Application Gateway
az network vnet create \
  -n $appgwVnetName \
  -g $rgName \
  -l $location \
  --address-prefix 10.0.0.0/16 \
  --subnet-name $appgwSnetName \
  --subnet-prefix 10.0.0.0/24
```

#### Create WAF Policy
```bash
# 🛡️ Create Web Application Firewall policy
az network application-gateway waf-policy create \
  --name $wafPolicyName \
  --resource-group $rgName
```

#### Create Application Gateway
```bash
# 🚪 Create Application Gateway with WAF v2
az network application-gateway create \
  -n $appgwName \
  -l $location \
  -g $rgName \
  --sku WAF_v2 \
  --public-ip-address $pipName \
  --vnet-name $appgwVnetName \
  --subnet $appgwSnetName \
  --priority 100 \
  --waf-policy $wafPolicyName
```

### 3️⃣ Enable AGIC

```bash
# 🔗 Enable Application Gateway Ingress Controller on AKS
appgwId=$(az network application-gateway show -n $appgwName -g $rgName -o tsv --query "id")
az aks enable-addons \
  -n $aksName \
  -g $rgName \
  -a ingress-appgw \
  --appgw-id $appgwId
```

### 4️⃣ Network Peering

```bash
# 🔄 Create VNet peerings for secure communication
aksVnetId=$(az network vnet show -n $aksVnetName -g $rgName -o tsv --query "id")
az network vnet peering create \
  -n AppGWtoAKSVnetPeering \
  -g $rgName \
  --vnet-name $appgwVnetName \
  --remote-vnet $aksVnetId \
  --allow-vnet-access

appGWVnetId=$(az network vnet show -n $appgwVnetName -g $rgName -o tsv --query "id")
az network vnet peering create \
  -n AKStoAppGWVnetPeering \
  -g $rgName \
  --vnet-name $aksVnetName \
  --remote-vnet $appGWVnetId \
  --allow-vnet-access
```

---

## 🚀 Deployment

Deploy the sample NGINX application:

```bash
# 📦 Clone the repository and deploy
git clone https://github.com/soaand01/aksAGIC.git
cd aksAGIC/deployment

# 🐳 Deploy NGINX application
kubectl create namespace nginx
kubectl apply -f nginx-deployment.yaml -n nginx
kubectl apply -f nginx-service.yaml -n nginx
kubectl apply -f nginx-ingress.yaml -n nginx
```

### 📁 Project Structure
```
aksAGIC/
├── 📄 README.md
└── 📁 deployment/
    ├── 🐳 nginx-deployment.yaml    # NGINX deployment with 5 replicas
    ├── 🌐 nginx-service.yaml       # ClusterIP service on port 8080
    └── 🚪 nginx-ingress.yaml       # Ingress configuration for AGIC
```

---

## 🧪 Testing

### Get Ingress IP Address
```bash
# 🔍 Check your ingress IP
kubectl get ingress -n nginx

# 📊 Monitor pod status
kubectl get pods -n nginx -o wide

# 📈 Check service endpoints
kubectl get endpoints -n nginx
```

### Verify Application Gateway
```bash
# 🌐 Get Application Gateway public IP
az network public-ip show \
  --resource-group $rgName \
  --name $pipName \
  --query ipAddress \
  --output tsv
```

---

## 📊 Monitoring

Monitor your deployment with these commands:

```bash
# 📈 Watch pods in real-time
kubectl get pods -n nginx -w

# 🔍 View ingress controller logs
kubectl logs -n kube-system -l app=ingress-appgw

# 📊 Check Application Gateway backend health
az network application-gateway show-backend-health \
  --name $appgwName \
  --resource-group $rgName
```

---

## 🧹 Cleanup

To clean up resources:

```bash
# 🗑️ Delete Kubernetes resources
kubectl delete namespace nginx

# 🗑️ Delete Azure resources
az network application-gateway delete --name $appgwName --resource-group $rgName
az network public-ip delete --name $pipName --resource-group $rgName
az network vnet delete --name $appgwVnetName --resource-group $rgName
```

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Anderson Soares Lopes**

[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/lopes221)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/andersonsoaresl/)

---

<div align="center">

**⭐ If you found this project helpful, please give it a star! ⭐**

*Made with ❤️ by Anderson Soares*

</div>
