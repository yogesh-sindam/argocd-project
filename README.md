# argocd-project
this repository testing for deploy application on multiple cluster using GitOps tool argoCD 
##
GitOps tools like argocd, flux 
how argocd is used at production levle 



##########################
deploying a application on multiple cluster

pre-requisites:
1. 3 EKS cluster (1 for Hub, 2 for spoke)
  Hub-Spoke deployment:

steps:
1. create a 3 AWS EKS cluster with name
2. Install argocd on Hub cluster
3. UI login to Argocd
4. add Spoke cluster to hub cluster
5. deploy application on both spoke cluster 

##########################
#step1 start:
create a cluster with the following command:
eksctl create cluster --name hub-cluster --region  
eksctl create cluster --name spoke-cluster-1 --region ap-southeast-2
eksctl create cluster --name spoke-cluster-2 --region ap-southeast-2

check how many cluster with the namespce ap-southeast-2
kubectl get cluster 
$Kubectl config get-contexts | grep ap-southeast-2

kubectl config use-contexts | namespace of cluster
kubectl config current-context

all the action is preformed on hub cluster

#step2 start:
install argocd with this command 



