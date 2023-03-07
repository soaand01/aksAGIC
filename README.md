<h1 align="center">
  <br>
  Aks and Application Gateway Ingress Controller
  
  <br>
</h1>

# Step by step

```
# Export your variables
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

```
# Create public ip
az network public-ip create -n $pipName -g $rgName -l $location --allocation-method Static --sku Standard
```

```
# Create vnet
az network vnet create -n $appgwVnetName -g $rgName -l $location --address-prefix 10.0.0.0/16 --subnet-name $appgwSnetName --subnet-prefix 10.0.0.0/24
```

```
# Create WAF policy
az network application-gateway waf-policy create --name $wafPolicyName --resource-group $rgName
```

```
# Create application gateway
az network application-gateway create -n $appgwName -l westeurope -g $rgName --sku WAF_v2 --public-ip-address $pipName --vnet-name $appgwVnetName --subnet $appgwSnetName --priority 100 --waf-policy $wafPolicyName
```

```
# Enable Application Gateway Ingress Controller on AKS
appgwId=$(az network application-gateway show -n $appgwName -g $rgName -o tsv --query "id")
az aks enable-addons -n $aksName -g $rgName -a ingress-appgw --appgw-id $appgwId
```

```
# Create vnet peerings
aksVnetId=$(az network vnet show -n $aksVnetName -g $rgName -o tsv --query "id")
az network vnet peering create -n AppGWtoAKSVnetPeering -g $rgName --vnet-name $appgwVnetName --remote-vnet $aksVnetId --allow-vnet-access

appGWVnetId=$(az network vnet show -n $appgwVnetName -g $rgName -o tsv --query "id")
az network vnet peering create -n AKStoAppGWVnetPeering -g $rgName --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access
```

```
# Let's deploy a simple webserver application
git clone https://github.com/soaand01/aksAGIC.git
cd deployments
kubectl create namespace nginx
kubectl apply -f nginx-deployment.yaml -n nginx
kubectl apply -f nginx-service.yaml -n nginx
kubectl apply -f nginx-ingress.yaml -n nginx
```

```
# Getting your Ingress IP
kubectl get ingress -n nginx
```


## Author

Anderson Soares Lopes

[![GitHub](https://skillicons.dev/icons?i=github)](https://github.com/lopes221)
[![Linkedin](https://skillicons.dev/icons?i=linkedin)](https://www.linkedin.com/in/andersonsoaresl/)
