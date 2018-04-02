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

#### Scaling Nodes

- `az aks scale -n jackaks -g akstrials --node-count 3`

#### Scaling Pods

- `kubectl get pods`

- `kubectl scale --replicas=5 deployment/azure-vote-front`

- `kubectl get pods`

#### Autoscaling ([horizontal pod autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/))

- `kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=3 --max=10`

- `kubectl get hpa`

#### Update Application

- `docker-compose up --build -d`  (update application and rebuild the images)

- `az acr list -g akstrials --query "[].{acrLoginServer:loginServer}" --output table`

- `docker tag azure-vote-front jomitacr.azurecr.io/azure-vote-front:v2`

- `docker push jomitacr.azurecr.io/azure-vote-front:v2`

- `kubectl get pod`

- `kubectl set image deployment azure-vote-front azure-vote-front=jomitacr.azurecr.io/azure-vote-front:v2`

- `kubectl get service azure-vote-front`

#### Monitor and Log Analytics

- Setup [Container Monitoring solution in Log Analytics](https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-containers) using Azure Portal

- Copy the `<WORKSPACE_ID>` and `<WORKSPACE_KEY>`

- `kubectl create secret generic omsagent-secret --from-literal=WSID=WORKSPACE_ID --from-literal=KEY=WORKSPACE_KEY`

- `kubectl create -f oms-daemonset.yaml`

- `kubectl get daemonset`

- Check the OMS Portal after few minutes to see the dashboard

#### Upgrade cluster

- `az aks get-upgrades -n jackaks -g akstrials --output table`

- `az aks upgrade -n jackaks -g akstrials --kubernetes-version 1.8.2`

- `az aks show -n jackaks -g akstrials --output table`
