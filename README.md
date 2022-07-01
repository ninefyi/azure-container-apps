# Prepare Azure resource

1. Create a resource group
2. Create an Azure Container App Environment
3. Create an Azure Container Registry
4. Create an Azure Container App
5. Create RBAC and assign a contributor role to a resource group no 1.
6. Create Secret on GitHub > Settings > Secrets > Actions
   1. AZURE_CREDENTIALS
   2. AZURE_RG
   3. ACR_PWD
   4. ACR_USR
   5. ACR_URL (Ex: acrfyi.azurecr.io)
   6. ACA_NAME (Ex: aca-fyi)

## Option 1 Using Azure Portal

## Option 2 Using Azure CLI (Tested on Mac / Linux)

- Create a resource group

```YAML
az group create -l eastus -n rg-container-app
```

- Create an Azure Container App Environment

```YAML
az containerapp env create -n aca-env-fyi -g rg-container-app -l eastus
```

- Create an Azure Container Registry

```YAML
az acr create -n acrfyi -g rg-container-app --sku Basic --admin-enabled -l eastus
az acr build --registry acrfyi.azurecr.io --image nodeapp:$(date +%Y.%m.%d).0 .
```

- Create an Azure Container App

```YAML
az containerapp create -n aca-fyi -g rg-container-app \
    --image acr-fyi.azurecr.io/node-app:v1.0 \
    --environment aca-env-fyi \
    --ingress external --target-port 80 \
    --registry-server acrfyi.azurecr.io --registry-username {acr_user} --registry-password {acr_password} \
    --query properties.configuration.ingress.fqdn
```

- Create RBAC and assign a contributor role to a resource group no 1.

```YAML
az ad sp create-for-rbac --name "sp-container-app" --role contributor \
--scopes /subscriptions/{subscription-id}/resourceGroups/rg-container-app \
--sdk-auth true --years 100
```

- Clean all azure resources

```YAML
az group delete -n rg-container-app
az ad sp delete --id sp-container-app
```
