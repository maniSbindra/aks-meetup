
# July DevOps Mumbai Meetup

## AKS Cluster with RBAC

### Link Create AKS cluster : https://docs.microsoft.com/en-us/azure/aks/create-cluster

### Create Resource Group
`az group create --name julymeetuprg --location eastus`

### Create AKS cluster with RBAC
`az aks create --name julymeetupaks --resource-group julymeetuprg --node-count 1 --node-vm-size Standard_D4s_v3 --generate-ssh-keys --enable-rbac`

### Get AKS admin creds
`az aks get-credentials --name julymeetupaks --resource-group julymeetuprg --admin`

## Initialize helm with cluster

### Create service account for tiller
`kubectl create -f helm-rbac.yaml`

### Initialize helm
`helm init --service-account tiller`


## Install service catalog

### Link OSBA : https://docs.microsoft.com/en-us/azure/aks/integrate-azure

### Add Repo for service catalog chart
`helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com`

### install service catalog
`helm install svc-cat/catalog --name catalog --namespace catalog --set controllerManager.healthcheck.enabled=false`

### create service principal for OSBA

```sh
az ad sp create-for-rbac

. ./.env.sh
```

## Install OSBA

### Add helm Repo for osba
`helm repo add azure https://kubernetescharts.blob.core.windows.net/azure`

### helm install OSBA
```sh
helm install azure/open-service-broker-azure --name osba --namespace osba \
    --set azure.subscriptionId=$AZURE_SUBSCRIPTION_ID \
    --set azure.tenantId=$AZURE_TENANT_ID \
    --set azure.clientId=$AZURE_CLIENT_ID \
    --set azure.clientSecret=$AZURE_CLIENT_SECRET
```

### Link wordpress with azure mysql using OSBA : https://github.com/Azure/helm-charts/tree/master/wordpress

### Install Service Broker CLI

```sh
curl -LO https://servicecatalogcli.blob.core.windows.net/cli/latest/$(uname -s)/$(uname -m)/svcat
chmod +x ./svcat
mv ./svcat /usr/local/bin/
```

## Install Wordpress

### Install Wordpress with Azure MySQL backend
`helm install azure/wordpress --name wordpress --namespace wordpress --set resources.requests.cpu=0`

### See brokers
`svcat get brokers`

### See Classes
`svcat get classes`

### See Plans
`svcat get plans`

### See secrets

### App in Action




## ACI

```sh
az container create -g julymeetuprg --name nginx --image nginx --ip-address public

watch az container list -o table

container logs -g julymeetuprg -n nginx

az container delete -g julymeetuprg --name nginx
```


## Virtual Kubelet

### Link using Virtual Kubelet : https://docs.microsoft.com/en-us/azure/aks/virtual-kubelet 

### Access default namespace

```sh
kubectl apply -f virtual-kubelet-rbac.yaml

kubens default
```

### install aci connector on cluster

`az aks install-connector --resource-group julymeetuprg --name julymeetupaks --os-type linux --connector-name julymeetupaciconnector --aci-resource-group MC_julymeetuprg_julymeetupaks_eastus`


### az aks remove-connector --resource-group julymeetuprg --name julymeetupaks --connector-name virtual-kubelet