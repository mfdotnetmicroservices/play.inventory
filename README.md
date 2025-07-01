# Play Inventory

Play Economy Inventory microservice




# Create and publish Contracts package

### For Windows (PowerShell): 


```powershell
$version="1.0.4"
$owner="mfdotnetmicroservices"
$gh_pat="[PAT HERE]"


Play.Inventory.Contracts.

dotnet pack src\Play.Inventory.Contracts\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.inventory -o ..\packages


dotnet nuget push ..\packages\Play.Inventory.Contracts.$version.nupkg --api-key $gh_pat --source "github"
```




### For macOS


```bash

version="1.0.4"
owner="mfdotnetmicroservices"
gh_pat="[PAT HERE]"


dotnet pack src/Play.Inventory.Contracts/ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.inventory -o ../packages

dotnet nuget push ../packages/Play.Inventory.Contracts.$version.nupkg --api-key $gh_pat --source "github"



```




<!-- ## Build the docker image

### windows (powershell)
```powershell

$env:GH_OWNER="mfdotnetmicroservices"
$env:GH_PAT="[PAT HERE]"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t play.inventory:$version .
```

### macOS (bash)
```bash

export GH_OWNER="mfdotnetmicroservices"
export GH_PAT="[PAT HERE]"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t play.inventory:$version .

``` -->



## Build the docker image


### windows (powershell)
```powershell
-beta
$env:GH_OWNER="mfdotnetmicroservices"
$env:GH_PAT="[PAT HERE]"
$acrname="playeconomyacr"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$acrnam.azurecr.io/play.inventory:$version" .
```

### macOS (bash)
```bash
acrname="playeconomyacr"
export GH_OWNER="mfdotnetmicroservices"
export GH_PAT="[PAT HERE]"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$acrname.azurecr.io/play.inventory:$version" .

```






## Run the docker image

### windows (powershell)
```powershell
$version="1.0.4"
docker run -it --rm -p 5004:5004 --name inventory -e MongoDbSettings__Host=mongo -e RabbitMQSettings__Host=rabbitmq --network playinfra_default play.inventory:$version  
```



## Run the docker image
### macOS (bash)
```bash
version="1.0.4"
cosmosDbConnString="[CONN STRING HERE]"
serviceBusConnString="[CONN STRING HERE]"
docker run -it --rm -p 5004:5004 --name inventory -e MongoDbSettings__ConnectionString=$cosmosDbConnString -e ServiceBusSettings__ConnectionString=$serviceBusConnString -e ServiceSettings__MessageBroker="SERVICEBUS" play.inventory:$version

```






## Publishing the Docker image
### For PC
```powershell

$acrname="playeconomyacr"
az acr login --name $acrname
docker push "$acrname.azurecr.io/play.inventory:$version"
```

### For MacOS

```bash
az acr login --name "$acrname"
docker push "$acrname.azurecr.io/play.inventory:$version"
```


## Creating the Azure Managed Identity and granting it access to Key Vault secrets

 ### Mac
```bash
appnameRg="playeconomy"
namespace="inventory"
appnamekv="playeconomy-key-vault"

az identity create --resource-group "$appnameRg" --name "$namespace"
IDENTITY_CLIENT_ID=$(az identity show -g "$appnameRg" -n "$namespace" --query clientId -o tsv)
IDENTITY_PRINCIPAL_ID=$(az identity show -g "$appnameRg" -n "$namespace" --query principalId -o tsv)
az keyvault set-policy -n "$appnamekv" --secret-permissions get list --spn "$IDENTITY_CLIENT_ID"
```



## Create the kubernetes namespace
## For windows
```powershell

$namespace="inventory"
kubectl create namespace $namespace
```

## For Mac
```bash

namespace="inventory"
kubectl create namespace "$namespace"
```


## Establish the federated identity credential
## For Mac

```bash

namespace="inventory"
appnamecluster="playeconomy_cluster"
appnameRg="playeconomy"

export AKS_OIDC_ISSUER="$(az aks show --name "${appnamecluster}" --resource-group "${appnameRg}" --query "oidcIssuerProfile.issuerUrl" --output tsv)"

az identity federated-credential create --name ${namespace} --identity-name "${namespace}" --resource-group "${appnameRg}" --issuer "${AKS_OIDC_ISSUER}" --subject system:serviceaccount:"${namespace}":"${namespace}-serviceaccount" --audience api://AzureADTokenExchange 

```

## Install the helm chart
```bash

namespace="inventory"

helm install inventory-service ./helm -f ./helm/values.yaml -n "$namespace"

```

## Required repository secrets for GitHub workflow 
GH_PAT: Created in GitHub user profile --> Settings --> Developer settings --> Personal access token