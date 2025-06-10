# Play Identity

Play Economy Identity microservice


# Create and publish Contracts package
### For Windows (PowerShell): 

```powershell 
$version="1.0.4"
$owner="mfdotnetmicroservices"
$gh_pat="[PAT HERE]"

dotnet pack src\Play.Identity.Contracts\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.identity -o ..\packages




dotnet nuget push ..\packages\Play.Identity.Contracts.$version.nupkg --api-key $gh_pat --source "github"
```


### For macOS
```bash

version="1.0.4"
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

## Create the Kubernetes secrets
### For windows
```powershell
$cosmosDbConnString="[CONN STRING HERE]"
$serviceBusConnString="[CONN STRING HERE]"
$adminPass="[PASSWORD HERE]"
$namespace="identity"
kubectl create secret generic identity-secrets --from-literal=cosmosdb-connectionstring=$cosmosDbConnString --from-literal=servicebus-connectionstring=$serviceBusConnString --from-literal=admin-password=$adminPass -n $namespace
```

### For mac
```bash
cosmosDbConnString="[CONN STRING HERE]"
serviceBusConnString="[CONN STRING HERE]"
adminPass="[PASSWORD HERE]"
namespace="identity"

kubectl create secret generic identity-secrets --from-literal=cosmosdb-connectionstring="$cosmosDbConnString" --from-literal=servicebus-connectionstring="$serviceBusConnString" --from-literal=admin-password="$adminPass" -n "$namespace"
```