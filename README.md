# Play Identity

Play Economy Identity microservice


# Create and publish Contracts package
### For Windows (PowerShell): 

```powershell 
$version="1.0.2"
$owner="mfdotnetmicroservices"
$gh_pat="[PAT HERE]"

dotnet pack src\Play.Identity.Contracts\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.identity -o ..\packages




dotnet nuget push ..\packages\Play.Identity.Contracts.$version.nupkg --api-key $gh_pat --source "github"
```


### For macOS
```bash

version="1.0.2"
owner="mfdotnetmicroservices"
gh_pat="[PAT HERE]"


dotnet pack src/Play.Identity.Contracts/ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.identity -o ../packages

dotnet nuget push ../packages/Play.Identity.Contracts.${version}.nupkg --api-key ${gh_pat} --source "github"

```
