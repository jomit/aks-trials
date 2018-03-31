# Tutorials

### Create & Test Docker Images

- `git clone https://github.com/Azure-Samples/azure-voting-app-redis.git`

- `cd azure-voting-app-redis`

- `docker-compose up -d`

- `docker images`

- `docker ps`

- Browse to [http://localhost:8080](http://localhost:8080)

- `docker-compose stop`

- `docker-compose down`

### Push Docker Images to Azure Container Registry (ACR)

- Install latest [Azure-CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)

- Need to use Powershell in admin mode to run below commands on Windows due to this [issue](https://github.com/Azure/azure-cli/issues/4843)

- `az group create --name akstrials --location eastus`

- `az acr create --resource-group akstrials --name jomitacr --sku Basic`

- `az acr login --name jomitacr -g akstrials`

- `az acr list --resource-group akstrials --query "[].{acrLoginServer:loginServer}" --output table`

- `docker images`

- `docker tag azure-vote-front jomitacr.azurecr.io/azure-vote-front:v1`

- `docker push jomitacr.azurecr.io/azure-vote-front:v1`

- `az acr repository list --name jomitacr -g akstrials --output table`

- `az acr repository show-tags --name jomitacr -g akstrials --repository azure-vote-front --output table`


### Create Azure Kubernetes Service Cluster

- `az provider register -n Microsoft.ContainerService`

- `az aks create -g akstrials -n jackaks --node-count 1 --generate-ssh-keys`

- Download `kubectl.exe` and set it in your path

    - https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-via-curl  or 

    - `az aks install-cli`

- `az aks get-credentials -g akstrials --n jackaks`

- `kubectl get nodes`

#### Configure ACR permissions for AKS identity
`
- `$CLIENT_ID=$(az aks show -g akstrials --n jackaks --query "servicePrincipalProfile.clientId" --output tsv)`

- `$ACR_ID=$(az acr show -n jomitacr -g akstrials --query "id" --output tsv)`

- `az role assignment create --assignee $CLIENT_ID --role Reader --scope $ACR_ID`


#### Deploy Application on AKS

- Update `azure-vote-all-in-one-redis.yaml` file to use the image from ACR `line 47`
    - `image: jomitacr.azurecr.io/azure-vote-front:v1`

- `kubectl create -f azure-vote-all-in-one-redis.yaml`

- `kubectl get service azure-vote-front --watch`   (wait for EXTERNAL-IP)

- Browse `http://EXTERNAL-IP`

- `kubectl proxy`  (to see the dashboard)