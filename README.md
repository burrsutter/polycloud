# AKS, EKS, GKE, OpenShift + ACM Hub + ArgoCD

4 clusters

OpenShift with ACM Hub and OpenShift Gitops (aka ArgoCD)

EKS - 

AKS -

GKE - Sydney

Delivery of Skupper via ArgoCD

Delivery of a Skupper ready demo app via ArgoCD




## Azure AKS Tokyo

```
az --version
azure-cli                         2.37.0 *

core                              2.37.0 *
telemetry                          1.0.6
```

```
export KUBECONFIG=/Users/burr/xKS/.kubeconfig/aks-tokyo-config

az login

az group create --name myAKSTokyoResourceGroup --location japaneast

az aks create --resource-group myAKSTokyoResourceGroup --name tokyo -s Standard_DS3_v2 --node-count 2
```

```
az aks get-credentials --resource-group myAKSTokyoResourceGroup --name tokyo --file $KUBECONFIG
```

```
kubectl cluster-info
Kubernetes control plane is running at https://tokyo-myakstokyoresour-75cfbc-8a30ee5c.hcp.japaneast.azmk8s.io:443
CoreDNS is running at https://tokyo-myakstokyoresour-75cfbc-8a30ee5c.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://tokyo-myakstokyoresour-75cfbc-8a30ee5c.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

```
kubectl get nodes
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-21111621-vmss000000   Ready    agent   2m4s   v1.22.11
aks-nodepool1-21111621-vmss000001   Ready    agent   2m4s   v1.22.11
```


## Amazon EKS Dublin

```
aws --version
aws-cli/2.7.7 Python/3.9.13 Darwin/21.5.0 source/arm64 prompt/off

eksctl version
0.92.0
```

```
export KUBECONFIG=/Users/burr/xKS/.kubeconfig/eks-dublin-config

eksctl create cluster \
--name dublin \
--region eu-west-1 \
--nodegroup-name myEKSworkers \
--instance-types=m6i.xlarge \
--nodes 2 \
--managed
```

```
eksctl utils write-kubeconfig --cluster=dublin --region eu-west-1
aws eks update-kubeconfig --name dublin --region eu-west-1
```

```
kubectl cluster-info
Kubernetes control plane is running at https://C66A1A362694283F399CD20AB42FB42C.gr7.eu-west-1.eks.amazonaws.com
CoreDNS is running at https://C66A1A362694283F399CD20AB42FB42C.gr7.eu-west-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

```
kubectl get nodes
NAME                                           STATUS   ROLES    AGE    VERSION
ip-192-168-13-254.eu-west-1.compute.internal   Ready    <none>   3m3s   v1.21.12-eks-5308cf7
ip-192-168-80-7.eu-west-1.compute.internal     Ready    <none>   3m3s   v1.21.12-eks-5308cf7
```

## Google GKE Sydney

```
export KUBECONFIG=/Users/burr/xKS/.kubeconfig/gke-sydney-config

gcloud container clusters create sydney --region australia-southeast1 --machine-type=e2-standard-4 --num-nodes 1
# num-nodes is per zone in GKE land, this becomes a 3 node cluster
```

```
# To address the following warning
# WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
# To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke

export PATH=/System/Volumes/Data/opt/homebrew/Caskroom/google-cloud-sdk/latest/google-cloud-sdk/bin/:$PATH
export USE_GKE_GCLOUD_AUTH_PLUGIN=True
gcloud container clusters get-credentials sydney --region australia-southeast1
```

```
kubectl cluster-info
Kubernetes control plane is running at https://34.116.80.226
GLBCDefaultBackend is running at https://34.116.80.226/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
KubeDNS is running at https://34.116.80.226/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://34.116.80.226/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

```
kubectl get nodes
NAME                                    STATUS   ROLES    AGE     VERSION
gke-sydney-default-pool-23828bfb-2hfp   Ready    <none>   7m22s   v1.22.8-gke.202
gke-sydney-default-pool-6783d5e9-vlbx   Ready    <none>   7m23s   v1.22.8-gke.202
gke-sydney-default-pool-9e2432ca-w9kb   Ready    <none>   7m23s   v1.22.8-gke.202
```

## Import in ACM Hub

Assumes you have an OCP + ACM Operator + GitOps Operator

https://github.com/burrsutter/acm-argocd-acs

Download pull secret from https://console.redhat.com/openshift/install/pull-secret

### Hub

```
oc project open-cluster-management
```

Assumes you downloaded the pull-secret.txt into $HOME/Downloads - default for most browsers

```
kubectl create secret generic mypullsecret -n open-cluster-management --from-file=.dockerconfigjson=$HOME/Downloads/pull-secret.txt --type=kubernetes.io/dockerconfigjson
```

Update the MCH

```
cat <<EOF | kubectl apply -f -
apiVersion: operator.open-cluster-management.io/v1
kind: MultiClusterHub
metadata:
  name: multiclusterhub
  namespace: open-cluster-management
spec:
  imagePullSecret: mypullsecret
EOF
```

#### Generate Tokyo files

```
./import-hub.sh tokyo
```

This will generate the following files to be applied to the Hub (management cluster):

managedcluster-tokyo.yaml

kac-tokyo.yaml

And two files to be applied to the Spoke (imported/managed cluster):

$HOME/tokyo-klusterlet-crd.yaml

$HOME/tokyo-import.yaml

#### Generate Dublin files

```
./import-hub.sh dublin
```

This will generate the following files to be applied to the Hub (management cluster):

managedcluster-dublin.yaml

kac-dublin.yaml

And two files to be applied to the Spoke (imported/managed cluster):

$HOME/dublin-klusterlet-crd.yaml

$HOME/dublin-import.yaml


#### Generate Sydney files

```
./import-hub.sh sydney
```

This will generate the following files to be applied to the Hub (management cluster):

managedcluster-sydney.yaml

kac-sydney.yaml

And two files to be applied to the Spoke (imported/managed cluster):

$HOME/sydney-klusterlet-crd.yaml

$HOME/sydney-import.yaml



### Tokyo

```
echo $KUBECONFIG
/Users/burr/xKS/.kubeconfig/aks-tokyo-config
```

```
kubectl get nodes
NAME                                STATUS   ROLES   AGE   VERSION
aks-nodepool1-21111621-vmss000000   Ready    agent   86m   v1.22.11
aks-nodepool1-21111621-vmss000001   Ready    agent   86m   v1.22.11
```

```
./import-spoke.sh tokyo
```

### Dublin

```
echo $KUBECONFIG
/Users/burr/xKS/.kubeconfig/eks-dublin-config
```

```
kubectl get nodes
NAME                                STATUS   ROLES   AGE   VERSION
aks-nodepool1-21111621-vmss000000   Ready    agent   86m   v1.22.11
aks-nodepool1-21111621-vmss000001   Ready    agent   86m   v1.22.11
```

```
./import-spoke.sh dublin
```


### Sydney

```
echo $KUBECONFIG
/Users/burr/xKS/.kubeconfig/gke-sydney-config
```

```
kubectl get nodes
NAME                                    STATUS   ROLES    AGE   VERSION
gke-sydney-default-pool-23828bfb-2hfp   Ready    <none>   40m   v1.22.8-gke.202
gke-sydney-default-pool-6783d5e9-vlbx   Ready    <none>   40m   v1.22.8-gke.202
gke-sydney-default-pool-9e2432ca-w9kb   Ready    <none>   40m   v1.22.8-gke.202
```

```
./import-spoke.sh sydney
```

![ACM Imports](images/imported-clusters.png)

## ArgoCD Managed Clusters

This uses OpenShift Gitops NOT upstream ArgoCD.  Primary difference seems to be the namespace "openshift-gitops" vs "argocd".

### Hub
```
export ARGOCD_URL=$(kubectl get route openshift-gitops-server -n openshift-gitops -o jsonpath="{.status.ingress[0].host}")
echo $ARGOCD_URL
```

```
export ARGOCD_PASS=$(kubectl get secret/openshift-gitops-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d)
echo $ARGOCD_PASS
```

```
argocd login --insecure --grpc-web $ARGOCD_URL --username admin --password $ARGOCD_PASS
```

```
argocd cluster list
SERVER                          NAME        VERSION  STATUS   MESSAGE                                              PROJECT
https://kubernetes.default.svc  in-cluster           Unknown  Cluster has no application and not being monitored.
```

```
open http://$ARGOCD_URL
```

Use `admin` and the password found via `echo $ARGOCD_PASS`

![ArgoCD Clusters Before](images/argocd-clusters-before.png)

### Tokyo ArgoCD Import

```
kubectl config get-contexts -o name
tokyo
```

```
argocd cluster add --kubeconfig $KUBECONFIG tokyo --name tokyo
```

output:

```
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `tokyo` with full cluster level admin privileges. Do you want to continue [y/N]? y
INFO[0011] ServiceAccount "argocd-manager" created in namespace "kube-system"
INFO[0012] ClusterRole "argocd-manager-role" created
INFO[0012] ClusterRoleBinding "argocd-manager-role-binding" created
Cluster 'https://tokyo-myakstokyoresour-75cfbc-8a30ee5c.hcp.japaneast.azmk8s.io:443' added
```

### Dublin ArgoCD Import

```
kubectl config get-contexts -o name
arn:aws:eks:eu-west-1:659615255973:cluster/dublin
```

```
argocd cluster add --kubeconfig $KUBECONFIG arn:aws:eks:eu-west-1:659615255973:cluster/dublin --name dublin
```

### Sydney ArgoCD Import

```
kubectl config get-contexts -o name
gke_ocp42project_australia-southeast1_sydney
```

```
argocd cluster add --kubeconfig $KUBECONFIG gke_ocp42project_australia-southeast1_sydney --name sydney
```

## Managed Clusters in ArgoCD

```
argocd cluster list
SERVER                                                                      NAME        VERSION  STATUS   MESSAGE                                              PROJECT
https://34.116.80.226                                                       sydney               Unknown  Cluster has no application and not being monitored.
https://C66A1A362694283F399CD20AB42FB42C.gr7.eu-west-1.eks.amazonaws.com    dublin               Unknown  Cluster has no application and not being monitored.
https://tokyo-myakstokyoresour-75cfbc-8a30ee5c.hcp.japaneast.azmk8s.io:443  tokyo                Unknown  Cluster has no application and not being monitored.
https://kubernetes.default.svc                                              in-cluster           Unknown  Cluster has no application and not being monitored.
```

![ArgoCD Clusters After](images/argocd-clusters-after.png)


### Hub for Skupper ApplicationSet



## Clean Up
```
az aks delete --resource-group myAKSTokyoResourceGroup --name tokyo
```

```
eksctl delete cluster --region=eu-west-1 --name=dublin
```

```
gcloud container clusters delete sydney --region australia-southeast1
```