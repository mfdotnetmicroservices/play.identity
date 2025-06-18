# Play Identity

Play Economy Identity microservice


# Create and publish Contracts package
### For Windows (PowerShell): 

```powershell 
$version="1.0.10"
$owner="mfdotnetmicroservices"
$gh_pat="[PAT HERE]"

dotnet pack src\Play.Identity.Contracts\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.identity -o ..\packages




dotnet nuget push ..\packages\Play.Identity.Contracts.$version.nupkg --api-key $gh_pat --source "github"
```


### For macOS
```bash

version="1.0.10"
owner="mfdotnetmicroservices"
gh_pat="[PAT HERE]"


dotnet pack src/Play.Identity.Contracts/ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.identity -o ../packages

dotnet nuget push ../packages/Play.Identity.Contracts.${version}.nupkg --api-key ${gh_pat} --source "github"

```


## Build the docker image
### windows (powershell)
```powershell

$env:GH_OWNER="mfdotnetmicroservices"
$env:GH_PAT="[PAT HERE]"
$acrname="playeconomyacr"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$acrname.azurecr.io/play.identity:$version" .
```

### macOS (bash)
```bash

export GH_OWNER="mfdotnetmicroservices"
export GH_PAT="[PAT HERE]"
acrname="playeconomyacr"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$acrname.azurecr.io/play.identity:$version" .

```




## Run the docker image
### windows (powershell)
```powershell
$adminPass="[PASSWORD HERE]"
$cosmosDbConnString="[CONN STRING HERE]"
$serviceBusConnString="[CONN STRING HERE]"
docker run -it --rm -p 5002:5002 --name identity -e MongoDbSettings__ConnectionString=$cosmosDbConnString -e ServiceBusSettings__ConnectionString=$serviceBusConnString -e ServiceSettings__MessageBroker="SERVICEBUS" -e IdentitySettings__AdminUserPassword=$adminPass play.identity:$version  
```



## Run the docker image
### macOS (bash)
```bash

adminPass="[PASSWORD HERE]"
cosmosDbConnString="[CONN STRING HERE]"
serviceBusConnString="[CONN STRING HERE]"
docker run -it --rm -p 5002:5002 --name identity -e MongoDbSettings__ConnectionString=$cosmosDbConnString -e ServiceBusSettings__ConnectionString=$serviceBusConnString -e ServiceSettings__MessageBroker="SERVICEBUS" -e IdentitySettings__AdminUserPassword=$adminPass play.identity:$version  

```


## Publishing the Docker image
### For PC
```powershell

$acrname="playeconomyacr"
az acr login --name $acrname
docker push "$acrname.azurecr.io/play.identity:$version"
```

### For MacOS

```bash
acrname="playeconomyacr"
az acr login --name "$acrname"
docker push "$acrname.azurecr.io/play.identity:$version"
```

## Create the kubernetes namespace
### For windows
```powershell
$namespace="identity"
kubectl create namespace $namespace 
```

### For Mac
```bash
namespace="identity"
kubectl create namespace "$namespace" 
```

## Assign the Key Vault Secrets User role to the managed identity for the Key Vault (**If not already done!**)
```bash 

az role assignment create --role "Key Vault Secrets User" \
  --assignee "$IDENTITY_PRINCIPAL_ID" \
  --scope "/subscriptions/$(az account show --query id -o tsv)/resourceGroups/$appnameRg/providers/Microsoft.KeyVault/vaults/$appnamekv"
```


## Creating the Azure Managed Identity and granting it access to Key Vault secrets
### Windows
```powershell
$appnameRg="playeconomy"
$namespace="identity"
$appnamekv="playeconomy-key-vault"
az identity create --resource-group $appnameRg --name $namespace
$IDENTITY_CLIENT_ID=az identity show -g $appnameRg -n $namespace --query clientId -otsv
az keyvault set-policy -n $appnamekv --secret-permissions get list --spn $IDENTITY_CLIENT_ID
```
 
 ### Mac
```bash
#!/bin/bash
appnameRg="playeconomy"
namespace="identity"
appnamekv="playeconomy-key-vault"

az identity create --resource-group "$appnameRg" --name "$namespace"
IDENTITY_CLIENT_ID=$(az identity show -g "$appnameRg" -n "$namespace" --query clientId -o tsv)
IDENTITY_PRINCIPAL_ID=$(az identity show -g "$appnameRg" -n "$namespace" --query principalId -o tsv)
az keyvault set-policy -n "$appnamekv" --secret-permissions get list --spn "$IDENTITY_CLIENT_ID"
```

## Establish the federated identity credential
## For mac
```bash
namespace="identity"
appnamecluster="playeconomy_cluster"
appnameRg="playeconomy"


export AKS_OIDC_ISSUER="$(az aks show --name "${appnamecluster}" --resource-group "${appnameRg}" --query "oidcIssuerProfile.issuerUrl" --output tsv)"


az identity federated-credential create --name ${namespace} --identity-name "${namespace}" --resource-group "${appnameRg}" --issuer "${AKS_OIDC_ISSUER}" --subject system:serviceaccount:"${namespace}":"${namespace}-serviceaccount" --audience api://AzureADTokenExchange 
```


## Install the helm chart 
```bash
namespace="identity"
appnameAcr="playeconomyacr"
helmUser="00000000-0000-0000-0000-000000000000"
helmPassword=$(az acr login --name "$appnameAcr" --expose-token --output tsv --query accessToken)

helm registry login "$appnameAcr.azurecr.io" --username "$helmUser" --password "$helmPassword"

chartVersion="0.1.0"
helm upgrade identity-service oci://$appnameAcr.azurecr.io/helm/microservice --version $chartVersion -f ./helm/values.yaml -n "$namespace" --install
``` 

## Install the helm chart (cont.)
```bash
# If its not working you can append a --debug to the command.
# Ex:

helm upgrade identity-service oci://$appnameAcr.azurecr.io/helm/microservice --version $chartVersion -f ./helm/values.yaml -n "$namespace" --install --debug 

# or if it's still not working try:
helm repo update
```

